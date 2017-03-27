# OTP and GenServer

## An introduction to OTP
OTP = Open Telephony Protocol

OTP is a framework for building Erlang distributed systems, regardless of whether or not they have anything to do with a telephony system. OTP is bundled with every Erlang release and is arguably the most important part of the Erlang standard library. Pretty much every Erlang distributed system uses it, not just telephony ones. The telephony problem was just the one that the creators of OTP were working on when it was created.

Since Elixir can call Erlang functions and modules with no runtime penalty, it is easy to use OTP from within Elixir projects. In a few cases Elixir has added a wrapper module around an OTP module to give an interface that works better with the pipeline operator or just feels more idiomatic to Elixir. In many cases though Elixir will expect you to just call the Erlang module explicitly.

> **INFO**
> Come back and complete this sentence as more are covered

OTP has a lot of modules, but the most important are GenServer, ...

## Creating your own version of a "GenServer"
Take the example from the last lesson:

```elixir
defmodule Speaker do
  def speak do
    receive do
      {:say, msg} ->
        IO.puts(msg)
      _other ->
        # throw the message away
    end
    
    speak
  end
end

pid = spawn(Speaker, :speak, [])
send pid, {:say, "Hello world!"}
```

We can take the code responsible for keeping the process alive etc. and put it in a separate module. Then our module only has to deal with outputing messages to STDOUT.

```elixir
defmodule Speaker do
  def handle_message({:say, msg}) do
    IO.outs msg
  end
  
  def handle_message(_other) do
    false
  end
end

defmodule Server do
  def start(callback_module) do
    spawn fn ->
      loop(callback_module)
    end
  end
  
  def loop(callback_module) do
    receive do
      message -> callback_module.handle_message(message)
    end
    
    loop(callback_module)
  end
end

server = Server.start(Speaker)
send server, {:say, "Hello world!"}
```

In this version the server is really basic, it just handles the spawning of the processes and the `receive` loop. But what if we want to let the managed process send messages back to the caller?

```elixir
defmodule Server do
  def start(callback_module) do
    parent = self
    spawn fn ->
      loop(callback_module, parent)
    end
  end
  
  def loop(callback_module, parent) do
    receive do
      message -> callback_module.handle_message(message, parent)
    end
    
    loop(callback_module, parent)
  end
end

defmodule PingPong do
  def handle_message(:ping, from) do
    send from, :pong
  end
  
  def handle_message(:pong, from) do
    send from, :ping
  end
end

ping_pong = Server.start(PingPong)
send ping_pong, :ping
# sends back :pong
```

But what if we need to maintain state? We can pass it in as an argument to the `Server.start/2` function and then pass it into the loop. Each time a message is received, the state is updated with the current balance.

```elixir
defmodule Server do
  def start(callback_module, state) do
    parent = self
    spawn fn ->
      loop(callback_module, parent, state)
    end
  end

  def loop(callback_module, parent, state) do
    receive do
      message ->
        state = callback_module.handle_message(message, parent, state)
        loop(callback_module, parent, state)
    end
  end
end

defmodule BankAccount do
  def handle_message({:deposit, amount}, _from, balance) do
    balance + amount
  end

  def handle_message({:withdraw, amount}, _from, balance) do
    balance - amount
  end

  def handle_message(:balance, from, balance) do
    send from, {:balance, balance}
    balance
  end
end

account = Server.start(BankAccount, 0)
send account, {:deposit, 50}
send account, {:withdraw, 25}
send account, {:balance}      #=> receives a {:balance, 25} message
```

## GenServer
The following should now look a little familiar:

```elixir
# start a process which _isn't_ linked to the current process
# i.e. won't bring current process down if it goes down
{:ok, pid} = GenServer.start(CallBackModule, [arg1, arg2], opts)

# start a process which _is_ linked to the current process
# i.e. will bring current process down if it goes down
{:ok, pid} = GenServer.start_link(CallBackModule, [arg1, arg2], opts)
```

Using our previous example:

```elixir
{:ok, account} = GenServer.start(BankAccount, 0)

GenServer.cast(account, {:deposit, 50})
GenServer.cast(account, {:withdraw, 25})
balance = GenServer.call(account, :balance) #=> 25
```

### `cast`
Sends a message without waiting or expect a response.

### `call`
Sends a message and waits for a response, blocking the current process.

Let's expand on the previous example:

```elixir
defmodule BankAccount do
  use GenServer

  # client functions
  
  def start(balance) do
    {:ok, account} = GenServer.start(__MODULE__, balance)
    account
  end

  @doc """
  Sends the `{:deposit, amount}` message to the `account` mailbox.
  Does not reply.
  """
  def deposit(account, amount) do
    GenServer.cast(account, {:deposit, amount})
  end

  @doc """
  Sends the `{:withdraw, amount}` message to the `account` mailbox.
  Does not reply.
  """
  def withdraw(account, amount) do
    GenServer.cast(account, {:withdraw, amount})
  end

  @doc """
  Sends the `:balance` message to the `account` mailbox.
  Gets the balance back as a reply.
  """
  def balance(account) do
    GenServer.call(account, :balance)
  end

  # server functions
  
  @doc """
  the init function can be used to modify the initial state
  or do something else with it before starting
  """
  def init(balance) do
    {:ok, balance}
  end

  @doc """
  Handles the deposit message. Adds the given amount to the
  current balance.
  """
  def handle_cast({:deposit, amount}, balance) do
    {:noreply, balance + amount}
  end

  @doc """
  Handles the withdraw message. Subtracts the given amount
  from the current balance.
  """
  def handle_cast({:withdraw, amount}, balance) do
    {:noreply, balance - amount}
  end

  @doc """
  Handles the balance message. Replies with the current balance.
  """
  def handle_call(:balance, _from, balance) do
    # second element contains the balance to reply with
    # the third contains the current balance, which remains unchanged
    {:reply, balance, balance}
  end
end
```

## When to use a GenServer
If you are used to programming in an OO language, you might have noticed that GenServers are a little like objects. If so, you'll be tempted to use them _like_ objects, meaning that you'll start using them everywhere, even when they're no necessary.

For example, the BankAccount module above could just as easily (and arguably more simply) written like so:

```elixir
defmodule BankAccount do
  defstruct balance: 0

  def new(balance) do
    %BankAccount{balance: balance}
  end

  def deposit(account, amount) do
    %{account | balance: account.balance + amount}
  end

  def withdraw(account, amount) do
    %{account | balance: account.balance - amount}
  end

  def balance(account) do
    account.balance
  end
end

```

The moral of the story is, just because you _can_ make something a GenServer, doesn't mean that you _should_.

## GenServer features
In order to decided whether something should be a GenServer or not take note of the following features:

- GenServer processes are distributed across cores (whereas the struct based approach above would be done within the same process)
- A GenServer process works on one message at a time (this makes GenServers ideal if you need to synchronise data from multiple sources or have a single source of truth. In example above this means that a bank account accessed from various different interfaces can have those messages played out in the order in which they came in, keeping the bank balance correct. However processing one message at a time can also make a GenServer a bottleneck)
- GenServer processes can be stopped and restarted (the critical functions in your application probably should be GenServers so that they can be properly supervised, started and stopped).
- GenServer process can be updated in place (the code for GenServers can also be updated whilst the app is still running, see docs on code_change callback)

## Final thoughts
Only make something a GenServer if it needs to be one. If you do make something a GenServer then hide the calls to GenServer functions behind your own wrapper functions so that you can change the implementation later if need be.
