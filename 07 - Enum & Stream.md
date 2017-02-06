# Enum
A type is consdiered enumerable if it implements the Enum protocol. Most Enum functions have this signature: `Enum.function(Enumerable, fun)`.

The following types are enumerable:

* Lists
* Keyword lists
* Maps (but not Structs)
* Ranges
* Streams

# Enum.at/2
Gets an element of a list at a certain index.

```elixir
Enum.at([1,2,3], 1, :default)           #=> 2
Enum.at(%{name: "Daniel"}, 0, :default) #=> {:name, "Daniel"}
```

> **INFO**
> Notice that enumerating over a Map returns Tuples and not Maps with the single value.

## Enum.filter/2
Filter an enumerable down to only those elements that pass the filter.

```elixir
Enum.filter ["string", 2, {}, %{}], fn(val) ->
  is_number(val)
end
#=> [2]

Enum.filter %{name: "Daniel", dob: 1991}, fn({_key, val}) ->
  is_binary(val)
end
#=> [name: "Daniel"]
```

> **INFO**
> Note that we don't get a List back when we enumerate over a Map. We get back a KeywordList, which is a List of key-value Tuples.

## Enum.reduce/2

Reduces an enumerable down to a single value.

```elixir
Enum.reduce [1,2,3], 0, fn(num, sum) ->
  sum + num
end
#=> 6

Enum.reduce ["episodes", "07-enum-and-stream", "", fn(segement, path) ->
  path <> "/" <> segment
end
#=> "/episodes/07-enum-and-stream"
```

## Enum.into/2
Convert an Enumerable to another type. Target must implement the `Collectable` protocol.

```elixir
%{name: "Daniel", dob: 1991}
|> Enum.filter(fn({_k, v}) -> is_binary(v) end)
|> Enum.into(%{})
#=> %(name: "Daniel"}
```

## Enum.take/2
Take a number of elements from an Enumerable.

```elixir
Enum.take(1..10, 5)
#=> [1,2,3,4,5]
```

# The Capture Operator
Wrap the named function in an anonymous function that takes the number of args required by the named function's arity.

```elixir
# longhand
Enum.filter [1,2,3], fn(val) -> is_number(val) end

# shorthand where values passed to arguments are not specified
Enum.filter [1,2,3], &is_number/1

# shorthand where values passed to arguments are specified
Enum.filter [1,2,3], &is_number(&1)

# another example
Enum.reduce [1,2,3], 0, fn(num, sum) ->
  sum + num
end

# becomes
Enum.reduce [1,2,3], &(&1 + &2)
# because we're not calling another function, we're using the `+` operator

# we can even capture the `+` operator
Enum.reduce([1, 2, 3], &+/2)
# which basically does what the `Enum.sum/1` function does so just use that instead ;)
Enum.sum([1, 2, 3])

# you can also capture functions from other modules
Enum.map(["Daniel", "Joe"], &String.upcase/1)  #=> ["DANIEL", "JOE"]
```

# Stream
The `Stream` module is the lazy version of the `Enum` module.

```elixir
[1, 2, 3, "string"]
  |> Stream.filter(&is_number/1)
  |> Stream.map(&(&1 * &2))
  
#=> %Stream{enum: [1, 2, 3, "string"], funs: [...]}
```

When you call functions in the `Stream` module they are not run straight away. Instead they return a `Stream` struct that contains the enumerable to be operated on and a list of functions to run.

```elixir
%Stream{
  enum: [1, 2, 3, "string"],
  funs: [
    #Function<7.16851754/1 in Stream.filter/2>,
    #Function<20.16851754/1 in Stream.map/2>
  ]
}
```

This means that Streams can be composed programatically.

```elixir
# build up the list of functions that you want to perform
stream = Stream.filter(list, &is_number/1)
stream = Stream.filter(stream, &(&1 * 2 == 4)

# now run it by asking it for its result,
# for example by calling `Enum.into/2`
Enum.into(stream, [])

# or, if you don't care about the result, you can use `Stream.run/1`
[1, 2, 3]
  |> Stream.each(fn(n) -> IO.puts(n) end)
  |> Stream.run
```

So why (or at least when) would you use `Stream` over `Enum`? A good example is when the result of running the `Enum` functions would result in an enumerable being looped over more then once.

```elixir
# iterates over the list twice
list
  |> Enum.filter(&is_number/1)
  |> Enum.filter(&(&1 * 2 == 4))

# iterates over the list once
list
  |> Stream.filter(&is_number/1)
  |> Stream.filter(&(&1 * 2 == 4))
  |> Enum.into([])
```

## `Stream.cycle/1`
Create an infinte stream of repeating elements.

```elixir
Stream.cycle(["Spring", "Summer", "Autumn", "Winter"])
  |> Enum.take(8)
  
#=> ["Spring", "Summer", "Autumn", "Winter", "Spring", "Summer", "Autumn", "Winter"]
```

## `Stream.iterate/2`
Create an infinite stream with a function.

```elixir
Stream.iterate(2, &(&1 * 2))
  |> Enum.take(3)
  
#=> [2, 4, 8]
```

## `Stream.resource/3`
Convert anything into a stream.
  - paginated data
  - lines in a file
  - events on a socket
  
for more info see [the blog post](http://blog.danielberkompas.com/elixir/twilio/2015/03/28/stream-paginated-apis-in-elixir.html)

```elixir
Stream.resource(start_fun, next_fun, after_fun)
```

## `Stream` vs. `Enum`
- `Stream` implements most of `Enum` functions lazily
- `Enum` is better for short enumberables and one operation
- `Stream` is better for long enumerables with multiple operations