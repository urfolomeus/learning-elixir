# Processes
Processes, or more specifically Beam VM processes, are at the heart of the power of Erlang and Elixir.

## Spawning processes
It's very easy to spawn a new process:

```elixir
spawn fn ->
  IO.puts "This will run in a separate process"
end

spawn SomeModule, :some_function, [arg1, arg2]
```

## Messaging
You don't normally spin up new processes without caring what they do. You usually want to hear back from them about what they're doing. For this we use messaging.

Spawning a process returns a *process identifier* or *pid*:

```elixir
pid = Spawn(fn -> ... end)

# get the current process's pid with self()
self
```

### Sending messages

```elixir
pid = Spawn(fn -> ... end)

send pid, :message
```

### Receiving messages
Every process in Elixir has a mailbox and this is where messages sent to them sit until they are handled. You can get a message from this mailbox by using the `receive` macro.

```elixir
receive do
  message -> # do something with the message
end
```

This will pull the first message out of the mailbox or, if there are no messages, block until a message is received.

`receive` supports pattern matching:

```elixir
receive do
  {:say, msg} ->
    IO.puts(msg)
  {:think, msg} ->
    Logger.debug(msg)
  _other ->
    # default case
end
```

> **IMPORTANT**
> If you don't have a match for each type of message in the processes mailbox (or a default case) then thge process's mailbox can fill up and overflow, causing the process to crash.

You can also specify a timeout for waiting for messages with `after`:

```elixir
receive do
  message -> process(message)
after 500 ->
  # do something
end
```

## An example process
Suppose we want a process that prints out any `say` messages that we give it to the console. All other messages will be ignored.

```elixir
defmodule Speaker do
  def speak do
    receive do
      {:say, msg} ->
        IO.puts(msg)
        speak # call the `speak` function again so that this recurses
      _other ->
        speak # throw the message away
    end
  end
end

pid = spawn(Speaker, :speak, [])
```

We call `speak` after every message is received so that this process will run until we kill it. It is fine to run an infinite loop in a process like this due to Elixir's tail call optimisation. You can send messages to this process as follows:

```elixir
send pid, {:say, "hello"}
```

## Process death
By default, if a spwaned process dies, the process that spawned it will not be notified. In many cases this is good, because it stops a failed process from taking down the entire system. Errors in the failed process will remain with it and not bubble up the process "stack".

```elixir
# in this case self() will be unaware of pid's demise
pid = spawn(fn -> ... end)
Process.exit(pid, :kill)
```

You can tie two processes together with `spawn_link`:

```elixir
romeo = self
juliet = spawn_link(fn -> ... end)

# this will cause the current process to exit too, because it is linked
# to the process we are killing
Process.exit(juliet, :kill)
```

If you would prefer to trap and handle the exit more elegantly, you can use `Process.flag` to catch the exit message as follows:

```elixir
Process.flag(:trap_exit, true)
juliet = spawn_link(fn -> ... end)

receive do
  {:EXIT, pid, reason} ->
    # revive juliet?
end
```

Monitor spawned processes with `spawn_monitor`:

```elixir
# `spawn_monitor` returns both pid and reference to the monitor
{juliet, _ref} = spawn_monitor(fn -> ... end)

# spawn_monitor will return a :DOWN message rather than :EXIT
# it also sends a reference to itself so that you can trap messages
# from multiple monitors
receive do
  {:DOWN, _ref, :process, pid} ->
    # revive juliet
end
```