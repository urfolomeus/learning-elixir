# Protocols

## Extension
In all programming languages there comes a time when you need to extend an existing language feature or library.

### Importing
Function-based approach to extension.

```elixir
defmodule MyModule do
  import LibraryModule
  
  # now override the library function with your own definition
  # then use your module's version, not the imported libraries
  def lib_function do
    do_something_else
  end
end
```

### Protocols
Data-based approach to extension.

Supposing we have the following structs:

```elixir
%Episode{
  title: "Introduction",
  number: 1
}

%User{
  first_name: "Daniel",
  last_name: "Berkompas"
}
```

and we want to use a `name` function from anothetr library that we don't control that doesn't yet know how to extract a name from those two structs, we could do so if the library has a `Protocol` for naming things:

```elixir
# in the existing library
defprotocol Nameable do
  def name(struct)
end

Nameable.name(struct_of_unknown_type)
```

we can do this by implementing the protocol in our code:

```elixir
# in our code
defimpl Nameable, for: Episode do
  def name(%{number: number, title: title}) do
    "#{number}. #{title}"
  end
end

defimpl Nameable, for: User do
  def name(%{first_name: first_name, title: title}) do
    "#{first_name} #{last_name}"
  end
end
```

## Comparing dates

```elixir
defmodule DateCompare do
  @doc "Determines if date1 is greater than date2"
  def gt(date1, date2) do
  end
end
```