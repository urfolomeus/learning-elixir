Sigils are a way to create a shorthand.

```elixir
%Regex{opts: [], re_pattern: {:re_pattern, 0, 0, ...}}

# with a sigil this becomes:
~r/hello/
```

Sigils are written starting with a tilde `~`, followed by a letter, followed by content within delimiters, followed optionally with one or more options.

`~r/content/options`

**Eight** delimiters are supported:

```elixir
~r/hello/i
~r|hello|i
~r"hello"i
~r'hello'i
~r(hello)i
~r[hello]i
~r{hello}i
~r<hello>i
```

## Built-in Sigils
Each built-in sigil in Elixir has two variants: lowercase and uppercase. The lowercase sigils allow interpolation, but uppercase sigils do not.

```elixir
word = "there"

~w(hello #{word})
#=> ["hello", "there"]

~W(hello, #{word})
#=> ["hello", "\#{word}"]
```

### `~w`
Use as a shorthand to create a list of words:

```elixir
~w(hello there)
#=> ["Hello", "there"]
```

Add the "a" option to make a list of atoms:

```elixir
~w(hello there)a
#=> [:hello, :there]
```

### `~s`
Avoid double-escaping quote symbols:

```elixir
"\"hello\""
```

can be represented by:

```elixir
~s("hello")
```

This is great when writing docstrings that need to have quoted content.

### `~c`
Create character list:

```elixir
~c(some text with a 'direct quote')
#=> 'some text with a \'direct quote\''
```

Useful if your character list contains apostrophes.

### `~r`
Used to shorthand the creation of regular expressions.

```elixir
~r/pattern/opts

# i.e.
~r/episodes/im
```

## How sigils work
`~r/hello/` calls your module's `sigil_r/2` function.

Example

```elixir
~r/hello/im

# is transformed by the compiler to:
sigil_r("hello", 'im')

# which returns:
%Regex{opts: ['i', 'm'], re_pattern: {:re_pattern, 0, 0 ...}}
```

## Creating your own sigils

```elixir
defmodule MySigils do
  def sigil_u(content, _opts) do
    content
    |> String.split
    |> Enum.map(&String.upcase/1)
  end
  
  # now you can do `~u(hello there)` and get ["HELLO", "THERE"]
end
```

You can pull your sigils in to any other module just by importing.

```elixir
defmodule MyModule do
  import MySigils
  
  def test do
    ~u(hello there)
  end
end
```

You can also override the default implementation of the Elixir built-in functions should you need to.

**PROCEED WITH CAUTION**

```elixir
defmodule MyModule do
  import Kernel, except: [sigil_r: 2]
  
  def sigil_r(content, opts) do
    "Hello world"
  end
  
  def use_sigil do
    ~r/hello/ # uses our implementation rather than Kernel's
  end
end
```