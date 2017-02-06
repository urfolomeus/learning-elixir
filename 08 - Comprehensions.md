# The `for` macro
There are three main parts:
- Generators
- Filters
- `:into`

`for` combines features of `map`, `filter` and `into`.

```elixir
for element <- Enumerable do
  element
end
#=> returns a list
```

## Generators
A generator is used to specify what we want the `for` macro to iterate over. This takes the format of `element <- Enumerable` where `element` is the variable that you want to assign each value in the `Enumerable` to.

A single generator behaves like a map:

```elixir
for name <- ["Joe", "Suzy"] do
  String.upcase(name)
end
#=> ["JOE", "SUZY"]

# is equivalent to:
Enum.map ["Joe", "Suzy"], fn(name) ->
  String.upcase(name)
end
```

Use multiple generators to create complex lists.

```elixir
deck = for suit <- [:hearts, :diamonds, :clubs, :spades],
    face <- [2, 3, 4, 5, 6, 7, 8, 9, 10, :jack, :queen, :king, :ace] do
  {suit, face}
end
#=> [{:hearts, 2}, {:hearts, 3}, {:hearts, 4}, {:hearts, 5}, ...]
```

Generators can do filtering with a pattern:

```elixir
for {:spades, face} <- deck do
  {:spades, face}
end
```

## Filters
You can do more complex filtering with Filters.

```elixir
for element <- Enumerable, filter do
  element
end
```

You can use variables from the generators in filters.

```elixir
for {:spades, face} <- deck, is_number(face) do
  {:spades, face}
end
#=> [{:spades, 2}, {:spades, 3}, ... , {:spades, 10}]
```

You can have multiple filters:

```elixir
for {suit, face} <- deck,
    suit == :spades,
    is_number(face),
    face > 5 do
  {suit, face}
end
#=> [{:spades, 6}, {:spades, 7}, {:spades, 8}, {:spades, 9}, {:spades, 10}]
```

## `:into`
This is used to make the `for` macro return something other than a list. For example, when filtering maps it can be used to ensure that the returned value is also a map.

```elixir
for {key, val} <- %{name: "Daniel", dob: 1991, email: "..."},
    key in [:name, :email],
    into: %{} do
  {key, val}
end
#=> %{name: "Daniel", email: "..."}
```

You can use `:into` with the `for` macro to return anything that implements the `Collectable` protocol. The following types support `Collectable`:
- Map
- List
- IO.Stream (can use this to continuously print from STDIN to STDOUT)
- Bitstring (Binary)

## Binary Comprehensions
Covered in more depth in a later chapter.

```elixir
pixels = <<213, 45, 132, 64, 76, 32, 76, 0, 0, 234, 32, 15>>

for <<r::8, g::8, b::8 <- pixels>>, do: {r, g, b}
#=> [{213, 45, 132}, {64, 76, 32}, {76, 0, 0}, {234, 32, 15}]
```

# `Enum` vs. `Stream` vs. `for`

```
|            | `Enum`  | `Stream` | `for` |
| `map`      | YES     | YES      | YES   |
| `filter`   | YES     | YES      | YES   |
| Lazy       | NO      | YES      | NO*   |
| Iterations | DEPENDS | ONE      | ONE*  |
| & Operator | YES     | YES      | NO**  |
```
* `for`, whilst not lazy, does only need one iteration to run and so is more efficient than `Enum`. However you cannot compose `for` in the same way as you can with `Stream`. The full `for` "process" must be built in one place and cannot be added to after the fact.

** because `for` takes a `do:` block and not a function, you cannot use the capture operator with it

## Tips
**IMPORTANT**
1. When running a single operation use `Enum` of `for` (prefer `Enum` unless good reason to do othereise though)
2. When running multiple operations use `Stream` or `for` (prefer `for` unless you want to compose across multiple lines)
3. When generating a list use `for` or a `Stream` generator (definitely prefer `for` first)
4. When working with multiple lists use `for`