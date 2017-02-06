## Functions

### Anonymous functions

```elixir
# defining an anonymous function
add = fn(a,b) ->
  a + b
end

# can also be done on a single line
add = fn(a,b) ->  a + b end

# you can bind anonymous functions to other variables
hello = add

# call an anonymous function with .(*params)
add.(1, 2)   #=> 3
hello.(1, 2) #=> 3

# changing the function bound to `add` does not change the function
# bound to `hello` because rebinding `add` just points it to another
# memory location. `hello` will still be pointing to the original
# location
add = fn(a,b,c) -> a + b + c end

add.(1, 2, 3)   #=> 6
hello.(1, 2)    #=> 3
add.(1, 2)      #=> BadArityError (arity is num of params function takes)
hello.(1, 2, 3) #=> BadArityError
```

### Named functions

```elixir
# defining a named function (note: needs to be in a module)
defmodule MyModule do
  def add(a, b) do
    a + b
  end
end
```

## Calling named functions

```elixir
# from outside module: must use Module name
defmodule MyModule do
  def add(a, b) do
    a + b
  end
end

add(1,2)          #=> (CompileError) undefined function add/2
MyModule.add(1,2) #=> 3

defmodule MyImplicitModule
  def add_and_ouput(a, b) do
    result = a + b
    "Implicitly returning #{result}"
  end
end

# functions have implicit returns, last value is returned
MyImplicitModule.add_and_output(1, 2) #=> "Implicitly returning 3"

# from inside module: don't need Module name
defmodule MyModule do
  def add(a, b) do
    a + b
  end
  
  def add42(a, b) do
    add(a + b) + 42
  end
end

MyModule.add42(1,2) #=> 45
```

### Private functions

```elixir
defmodule Number do
  def format(number) do
    format = config[:format]
    ... # convert number using format from config (i.e. "n,nnn")
  end
  
  defp config do
    # get config here
  end
end

Number.format(1234) #=> "1,234"
Number.config       #=> undefined function: Numner.config/0
```

## Modules
Named functions must be placed within modules. Modules are a way to gather related functions together and to namespace them to prevent clashes with functions in other code you might be writing/using.

```elixir
# defining
defmodule MyModule do
  ...
end
```

### Nesting modules

```elixir
# define
defmodule Math do
  defmodule Division do
    def divide(a, b) do
      a / b
    end
  end
end

# define shorthand
defmodule Math.Division do
  def divide(a, b) do
    a / b
  end
end

# call
Math.Division.divide(15, 5) #=> 3
```

### Using other modules
Calling other modules using their full name can be tedious, especially if the name is really long.

```elixir
# calling another module with a really long name
defmodule MyModule do
  def my_function(args) do
    Really.Long.OtherModule.other_function
  end
end


# ALIASING

# using the alias macro instead
defmodule MyModule do
  alias Really.Long.OtherModule
  
  def my_function(args) do
    OtherModule.other_function
  end
end

# or you can aliase using any name you like
defmodule MyModule do
  alias Really.Long.OtherModule, as: O
  
  def my_function(args) do
    O.other_function
  end
end


# IMPORTING

# when you import you don't need to use the other module's name at all
defmodule MyModule do
  import Really.Long.OtherModule
  
  def my_function(args) do
    other_function
  end
end

# you can also scope what gets imported by using `only` and
# specifying the function names and their arities
defmodule MyModule do
  import Really.Long.OtherModule, only: [other_function: 1]
  
  def my_function(args) do
    other_function
  end
end

# or you can scope what doesn't get imported by using `except` and
# specifying the function names and their arities
defmodule MyModule do
  import Really.Long.OtherModule, except: [other_function: 1]
  
  def my_function(args) do
    other_function
  end
end

# if you want a function on another module to appear as if it is
# written on your module, you can use `defdelegate`
defmodule MyModule do
  defdelegate function(arg1, arg2), to: Really.Long.OtherModule
                                    as: :my_function
end

MyModule.my_function(1, 2) #=> calls OtherModule.function(1, 2)
```

## Documentation

- @moduledoc: Documentation for your module
- @doc: Documentation for your function

> **INFO**
> - You can and should use Markdown to write your docs.
> - Private functions can't be documented with `@doc` tags.

```elixir
defmodule Math do
  @moduledoc """
  Defines some basic math operation functions.
  """
  # ...
  
  @doc """
  Adds two integers together.
  
  ## Examples
      add(1, 2)
      3
      
      add(5, 5)
      10
  """
  def add(a, b) do
    a + b
  end
end

# Using documentation in iex
h Math
#=> moduledoc

h Math.add
#=> doc
```

> **Info**
> Also check out ex_doc for making HTML versions of you documentation.

## Structs
You can define Structs in Modules using the `defstruct` macro.

`defstruct attr1: val, attr2: ...`

```elixir
defmodule User do
  defstruct name: nil,
            email: nil
end

%User{}               #=> %User{email: nil, name: nil}
                      #=> %{__struct__: User, name: nil, email: nil}
%User{name: "Daniel"} #=> %User{email: nil, name: "Daniel"}
                      #=> %{__struct__: User, name: "Daniel", email: nil}
```