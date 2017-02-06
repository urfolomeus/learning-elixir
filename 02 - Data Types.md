## Two groups
Base Types and High Level Types (which are made from Base Types)

## Base types

### Numbers
Integers (no decimal places)
Floats (have decimal places)

### Atoms
Named constant. Most often used for naming things.

```elixir
# how to write atoms
:this_is_an_atom
:"An atom with spaces"

# Module names are atoms
GenServer.start_link
GenServer == :Genserver #=> true

# True and False are atoms
true == :true #=> true
false == :false #=> true

# Nil is an atom
nil == :nil #=> true
```

### Binaries
Represents a series of bytes which can be mapped to a number or a value.

Examples:

```elixir
01101000 01100101 01101100 01101100 01101111

# is represented in Elixir as this Binary:
<<104, 101, 108, 108, 111>>

# which can also be written this way
"hello"
```

### Strings
Are just binaries using a different notation

Examples:

```elixir
# Strings are just binaries
"hello" == <<104, 101, 108, 108, 111>>

"""
This is a multiline string.
They are used in documentation.
"""
```

### Maps
Key-Value stores. `%{key: value, ...}`

Examples:

```elixir
episode = %{
  name: "Data types",
  author: "Daniel Berkompas"
}

episode.name      #=> "Data types"
episode[:author]  #=> "Daniel Berkompas"
```

**CAVEAT**

```elixir
# If you use Strings for keys you can't use the dot notation
# for retrieving values
episode = %{
  "name"   => "Data types",
  "author" => "Daniel Berkompas"
}

episode.name      #=> ERROR: doesn't work
episode["author"]  #=> "Daniel Berkompas"
```

> **CAVEAT [Since 1.2]**
> "Maps can now scale from dozens to millions of keys. Therefore, usage of the modules Dict and HashDict is now discouraged and will be deprecated in future releases, instead use Map. Similarly, Set and HashSet will be deprecated in favor of MapSet"

### Tuples
A collection of items. Usually used to hold together a fixed number of items.

Examples:

```elixir
me = {"Daniel", 24}

# how to access elements in a Tuple
elem(me, 0) #=> "Daniel"
elem(me, 1) #=> 24

# how to update an element in an existing Tuple
put_elem(me, 1, 25) #=> {"Daniel", 25}
```

### Lists
Variable length collection. Like Tuples they can contain any types of values. Similar to arrays in other languages.

Examples:

```elixir
ages = [42, 31, 24]
names = ["Ash", "Leslie", "Dori"]

Enum.at(names, 0) #=> "Ash"
```

> **CAVEAT**
> Implemented as immutable `head/tail` pairs (also known as a singly-linked list). 
> This means that it's very **FAST** to add an element to the front of the list.

```elixir
list1 = [1, 2 , 3]
list2 = [0 | list1] #=> [0, 1, 2, 3]
```

This is because all you are doing is creating a new element with a link to the first element of `list1`. `list1` doesn't change (and never will because it's immutable), so `list2` just contains a link to `list1`, it doesn't contain its own copy.

However it is potentially **SLOW** to add an element to the end of the list.

```elixir
list1 = [1, 2 , 3]
list2 = list1 ++ [4]
```

This is slower because in order to add element `4` to the end of the list you need to modify the `tail` of element `3`. However you can't do this because element `3` is immutable, so you need to make a copy of it in `list2`. This proliferates right to the start of `list2`, thus making it a potentially very expensive operation.

Inserting an element into the middle of a list is similarly expensive.

**Summary**

Lists:
- are `head/tail` pairs
- immutability makes them *memory efficient*
- prepending is **FAST**
- appending is **SLOW**
- inserting elements can also be slow
- reading the whole list can be slow

### Character lists

`[integer, integer, ...]`

Examples:

```elixir
# shorthand notation
`hello`

# under the hood
[104, 101, 108, 108, 111]
```

> **IMPORTANT**
> Unless you are working with an Erlang library, use Binaries instead!

### Functions
`fn(args) -> ... end`

Examples:

```elixir
add = fn(a,b_ ->
        a + b
      end
      
add.(1, 2) #=> 3
```

### Other basic types

- PIDs
- References
- Records
- Port references

## How to check types

`is_type(value)`

Examples:

```elixir
is_atom(:hello)       #=> true
is_list([1, 2, 3])    #=> true
is(map(%{key: value}) #=> true
```

## High Level Types
Not true types. They are constructs build from basic types.

### Keyword lists
`[{:atom, value}, ...]`

Examples:

```elixir
# shorthand notation
attrs = [name: "Daniel Berkompas", email: "test@example.com"]

# under the hood
[{:name, "Daniel Berkompas"}, {:email, "test@example.com"}]

# you can access values in a keyword list by their label
attrs[:name]   #=> "Daniel Berkmompas"
attrs[:email]  #=> "test@example.com"
```

> **CAVEAT**
> Bear in mind that keyword lists are **lists** and have all of the pros and cons associated with them.

### Structs
`%{__struct__: ModuleName, ...}`

Examples:

```elixir
# shorthand notation
%Episode{
  title: "Data Types",
  author: "Daniel Berkompas"
}

# under the hood
%{
  __struct__: Episode,
  title: "Data Types",
  author: "Daniel Berkompas"
}
```

### Range
`%Range{first: number, last: number}`

Examples:

```elixir
# shorthand notation
0..100

# under the hood
%Range{
  first: 0,
  last:  100,
}
```

### Regular expressions
`%Regex{opts: ..., re_pattern: ...}`

Examples:

```elixir
# shorthand notation
-r/hello/

# under the hood
%Regex{
  opts: "",
  re_pattern: {:re_pattern, <<69, 82, 67, 80, 81, 0, ...>>},
  source: "hello"
}
```

### Other high level types

- Tasks
- Agents
- Streams
- HashDicts (don't use)
- HashSets (don't use)