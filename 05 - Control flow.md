## Basic control flow

### `cond`
Run the first block where the expression is truthy.

```
cond do
  expr -> code
  true -> default
end
```

> **INFO**
> Use `cond` when you need to make a decision based on the value of multiple variables.

Example:

```elixir
cond do
  1 + 1 == 1 ->
    "This will never happen!"
  2 * 2 != 4 ->
    "Nor this"
  true ->
    "This will"
end
```

## `case`
Run a block of code based on the value of a given expression.

```
case expr do
  output -> code
  _other -> default
end
```

> **INFO**
> - Use `case` when you need to make a decision based on the value of a single variable.
> Powerful because you can use pattern matching

Example:

```elixir
case Stripe.Customer.create(attrs) do
  {:ok, customer} ->
    "A customer was created with ID #{customer.id}"
  {:error, reason} ->
    "Customer could not be created because #{reason}"
  other ->
    "An unknown error occurred: #{other}"
end
```

### `if`
An if statement

```
if expr do
  // code if expr is true
else
  // code if expr is false
end
```

### `unless`
An unless statement

```
unless expr do
  // code if expr is false
end
```

## Pattern matching - Part II

```elixir
# take the following function
def blank?(value) do
  # determine whether a value is nil, false, or ""
end

# first implementation
def blank?(value) do
  case value do
    nil    -> true
    false  -> true
    ""     -> true
    _other -> false
  end
end

# second implementation
def blank?(nil),    do: true
def blank?(false),  do: true
def blank?(""),     do: true
def blank?(_other), do: false
```

## Guards
Guards let you use pattern matching to determine which function to run.

```
def my_function(arg) when expr do
  # ...
end
```

```elixir
# third implementation
def blank?(value) when value in [nil, false, ""], do: true
def blank?(_other), do: false
```

Case statements also support guards.

```elixir
# fourth implementation
def blank?(value) do
  case value do
    value when value in [nil, false, ""] -> true
    _ -> false
  end
end
```

We can use guard clauses in case statements to further refine pattern matches.

Example:

```elixir
case response do
  {:ok, body} ->
    # great success
  {:error, status_code, body} when status_code in 400..499 ->
    # handle 400 status codes
  {:error, status_code, body} when status_code in 500..599 ->
    # handle 500 status codes
  _other ->
    # default case
end
```

> **CAVEAT**
> *Always* include a catch all error clause when using guards so that anything which doesn't match at all can be caught and handled. 

### "Static typing"
Combined, pattern matching and guards can act like static typing:

```elixir
# in this example only numbers are accepted because neither function clause
# allows non-numeric values to be given as params
def zero?(0), do: true
def zero?(n), when is_number(n) do: false

# in this example name can be called with either a User struct or an
# Episode struct that has a name that is of type Binary. Everything else
# results in an error.
def name(%User{} = user) do
  user.first_name <> " " <> user.last_name
end
def name(%Episode{name: name}) when is_binary(name), do: name
def name(unsupported) do
  raise "name does not support #{inspect unsupported}"
end
```

See [here](http://elixir-lang.org/getting-started/case-cond-and-if.html#expressions-in-guard-clauses) for the expressions that are allowed to be used in Guards.

The code we've been using so far as been useful for showing the different options, and for introducing Guards. However, the best solution would probably be as follows:

```elixir
# fifth (and final) implementation
def blank?(value), do: value in [nil, false, ""]
```

Whenever we need to return a boolean based on an expression, it is best to return that expression as it in return (pun unavoided) will provide `true` or `false`.

## Final thoughts

> **IMPORTANT**
> - prefer pattern matching over `if`
> - prefer function pipelines to nested `if`, `cond` or `case` statements.

[**Related blog post**](http://blog.danielberkompas.com/2015/09/03/better-pipelines-with-monadex.html)