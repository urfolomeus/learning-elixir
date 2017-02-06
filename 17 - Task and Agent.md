# Task and Agent

## Task
If you just want to run a block of code asynchronously you can use `Task` rather than a `GenServer`. The most common way to do this is to use the `Task.async` function.

```elixir
# first form - takes an anonymous function
task = Task.async fn ->
  IO.puts "Hello world!"
end

# second form - takes a module name, a function name
# and a list of arguments
task = Task.async(IO, :puts, ["Hello world!"])

# `Task.async` runs in a separate process, to get the
# result, call `Task.await` passing in the task
result = Task.await(task)
```

To take our `pmap` example from episode 13 on processes and use `Task`:

```elixir
defmodule Parallel do
  def pmap(list, fun) do
    list
    |> Enum.map(&Task.async(fn -> fun.(&1) end ))
    |> Enum.map(&Task.await/1)
  end
end
```

### Supervised Tasks
Tasks can also be supervised in the same way that GenServers are:

```elixir
children = [
  supervisor(Task.Supervisor, [[name: MyApp.TaskSupervisor]])
]

# if you don't need to capture the result use `start_child`
Task.Supervisor.start_child(MyApp.TaskSupervisor, fn ->
  IO.puts "Hello world!"
end)

# if you do need to capture the result use `async`
Task.Supervisor.async(MyApp.TaskSupervisor, IO, :puts, ["Hello world!"])
|> Task.await
```

### Task Facts
- Tasks are _linked_ by default
- You can mount tasks directly in your Supervision tree
- See `Task.yield/1` if you want to await multiple times

## Agent
Agent is a simple abstraction around state. It basically meets the need for a GenServer that just stores and updates a value. You can start an Agent like a GenServer except you pass in a function that returns the initial state of the Agent. You can then call `update` and `get` on the Agent to change and retrieve its state.

```elixir
{:ok, agent} = Agent.start_link(fn -> 0 end)
Agent.update(agent, fn(state) -> state + 1 end)
Agent.get(state, fn(state) -> state end)
```

Updating our BankAccount from ch14 to use an Agent rather than a GenServer.

```elixir
defmodule BankAccount do
  def start_link(balance) do
    Agent.start_link(fn -> balance end)
  end

  def deposit(account, amount) do
    Agent.update(account, fn(balance) -> balance + amount end)
  end

  def withdraw(account, amount) do
    Agent.update(account, fn(balance) -> balance - amount end)
  end

  def balance(account) do
    Agent.get(account, fn(balance) -> balance end)
  end
end
```

Or to use the capture operator to make things even more succinct:

```elixir
defmodule BankAccount do
  def start_link(balance) do
    Agent.start_link(fn -> balance end)
  end

  def deposit(account, amount) do
    Agent.update(account, &(&1 + amount))
  end

  def withdraw(account, amount) do
    Agent.update(account, &(&1 - amount))
  end

  def balance(account) do
    Agent.get(account, &(&1))
  end
end
```