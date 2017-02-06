# Nodes
A node is an instance of the BEAM virtual machine. You start one up every time you run `iex` or run a `mix` task.

You use the `Node` module to interact with nodes.
- `Node.self`: returns the current node's name
- `Node.list`: returns a list of connected node names

```elixir
$ iex --sname <node_name>

iex> Node.self
:node_name@computer
```

## Connecting to another node

```elixir
$ iex --sname <node_name>

iex> Node.connect(:other_node@computer)
true

iex> Node.list
[:other_node@computer]
```

## Connecting across the network
You can connect nodes across separate machines, as long as they share the same cookie value.

> **INFO**
> Keep in mind thaty the cookie is not a secure measure. It is actually sent as plain text across the network. BEAM doesn't secure the connection over the network. Instead it's your job to secure the network itself.

```elixir
# On computer A
$ iex --sname node1 --cookie secret

# On computer B
$ iex --sname node2 --cookie secret
```

Nodes on the same machine automatically use the same cookie. You can inspect your machine's cookie by inspecting `~/.erlang.cookie`.

## Spawning processes across nodes
Once you have nodes set up and linked together. it's pretty easy to spawn processes on other nodes.

```elixir
pid = Node.spawn :node2@computer, fn ->
  IO.puts "Hello world!"
end
```

## Sending messages to other nodes

```elixir
send pid, :message

# if you don't have a pid and the process on the other node is named
# you can use a tuple with the process name as the first element and
# the node name as the second element
GenServer.cast({BackgroundJob, :node2@other}, {:new_job, ...})
```

## Important facts
- code is _not sent_
- all data in the message will be copied
- network latency still applies

## Benefits
- work can be distributed between machines
- communication is native
- scaling follows a clear trajectory
  1. split the important components of your app into processes
  2. then, as things get more complex, into separate OTP apps
  3. if one or more of those apps starts experiencing too much load you can easily sart running it on a machine with better hardware