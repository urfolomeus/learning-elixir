## Calculating `List.length`

```elixir
defmodule MyList do
  def length(list) do
    length(list, 0)
  end
  
  defp length([],    count), do: count
  defp length([_|t], count), do: length(t, count + 1)
end

list = [1, 2, 3, 4]
MyList.length(list) #=> 4

# length([1, 2, 3, 4], 0)
# length([2, 3, 4], 1)
# length([3, 4], 2)
# length([4], 3)
# length([], 4) #=> returns 4
```

## Tail call optimisation
When a programming language calls a function, it uses a special place in memory called the function stack. Each function that needs to be run takes up a frame on this stack. In many programming languages the function call above would take up 5 frames on this stack, one for each of the iterations through the recursion process. The programming language holds all of these frames in memory until the last iteration completes and then the final result could be returned. So, if you tried to do this with a very long list, you'd fill up all the available memory and cause a dreaded stack overflow error. Since Elixir relies so heavily on this type of operation it has a way of dealing with it more efficiently called Tail Call Optimisation.

The term Tail Call refers to a function where the last thing it does is call itself. In such cases Elixir will only put one function call on the stack at a time and throw out the last one. Since there's only one call on the stack at a time, this operation won't cause a stack overflow, no matter how long the list is.

However this only works if the last thing a function does is call itself. It is easy to accidentally do something that means you're not actually having the function call itself last.

```elixir
# this is not tail recursive because the string concatenation happens last
def join([h|t] do
  h <> " " <> join(t)
end

# this is tail recursive
def join([h|t], string) do
  string = string <> " " <> h
  join(t, string)
end
```

## `MyList.each/2`

```elixir
defmodule MyList do
  def each([], _fun), do: :ok
  def each([h|t], fun) do
    fun.(h)
    each(t, fun)
  end
end

MyList.each([1, 2, 3], fn(n) -> IO.puts num end)

# each([1, 2, 3], fun) => 1
# each([2, 3], fun)    => 2
# each([3], fun)       => 3
# each([], fun)        => :ok
```

## `MyList.map/2`

```elixir
defmodule MyList do
  def map(list, fun) do
    do_map(list, fun, [])
  end
  
  defp do_map([], _fun, acc) do
    :lists.reverse(acc)
  end
  defp do_map([h|t], fun, acc) do
    result = fun.(h)
    acc = [result | acc]
    do_map(t, fun, acc)
  end
end

list = [
  {"Daniel", 24},
  {"Ash", 32}
]

MyList.map list, fn({name, _age}) ->
  name
end

# do_map([{"Daniel", 24}, {"Ash", 32}], fun) => ["Daniel"]
# do_map([{"Ash", 32}], fun) => ["Ash", "Daniel"]
# do_map([], _fun) => ["Daniel", "Ash"]
```