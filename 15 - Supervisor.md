# Supervisor
Elixir inherits its error handling philosophy from Erlang. "Let it crash". Instead of attempting to handle error catching and handling in your application code (and therefor obfuscating that code and making it more complex) OTP instead handles and restarts crashed processes. It does this via Supervisors.

A Supervisor is a process that is responsible for watching other processes and restarting them if they crash.

```elixir
defmodule MyApp.Supervisor
  use Supervisor
  
  def start_link do
    Supervisor.start_link(__MODULE__, [])
  end
  
  def init([]) do
    # each child process passed to the worker needs to be
    # a GenServer that implements a `start_link` function
    children = [
      worker(ProcessA, [args]),
      worker(ProcessB, [args])
    ]
    
    supervise(children, strategy: :one_for_one)
  end
end
```

## Restart strategies

* `:one_for_one` will restart a single process if it dies
* `:simple_one_for_one` works basically the same as `:one_for_one` but works better if you are dynamically adding child processes to a supervisor.
* `:one_for_all` will restart all of a supervisor's child processes if one fails.
* `:rest_for_one` is a little odd. With this strategy, if a process dies, all the processes that were started after it in start order will be restarted.

## Which strategy should you use?
Use for `:one_for_one` until it doesn't work for you

## Supervision trees
- Error isolation
- Elegant error recovery
- Self-healing systems

## How to build OTP applications
Don't ask yourself "What modules/classes do I need?". Instead ask yourself "What processes do I need?".

## Tips
- You can create a supervised mix project with

```elixir
> mix new <app_name> --sup  # lib/my_app.ex will be a supervisor
```

- GenServer processes can be named to make it easy to identify/refer to them later on

```elixir
GenServer.start_link(Game.Cache, [], name: Game.Cache)
GenServer.cast(Game.Cache, {:save, state})
```

- Worker processes can have IDs

```elixir
worker(SupervisedProcess, [], id: "some-id")
```

- Supervisors can themselves be supervised. Just `supervisor` instead of `worker` in your `children` list.

```elixir
supervisor(Game.Supervisor, [])
```

- Try a built-in Erlang database for storage such as [ETS](http://www.erlang.org/doc/man/ets.html) (Erlang's memory-based store) or [DETS](http://www.erlang.org/doc/man/dets.html) (Erlang's disk-based store). It also has a relational database called [Mnesia](http://www.erlang.org/doc/man/mnesia.html).

```flow

```