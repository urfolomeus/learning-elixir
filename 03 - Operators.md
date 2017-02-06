## What are operators?
Operators are just functions. However they are a special form of function with a special type of calling notation and a limited number of parameters.

i.e. instead of calling `func(left, right)` we call `left <func> right`.

## The match operator
`pattern = data`

**Not** and assignment operator (although it has some similar characteristics). Instead it matches the pattern on the left hand side with the data on the right hand side. If the two match exactly then a match is made, if a variable name is placed on the left hand side then the data on the right hand side is bound to the variable name.

Example:

```elixir
24 = 24 #=> 24 (i.e. match)
24 = 42 #=> MatchError no match of right hand side value: 42

name = "Daniel"
age  = 24

name #=> "Daniel"
age  #=> 24
```

> **A side note on variables**
> Variables in Elixir are dynamically typed. IOW you don't have to assign a type when you create a variable, you just bind any piece of data to it.

### Rebinding variables
Unlike Erlang, variables can be rebound in Elixir. If the variable on the left hand side is already bound then it will be rebound to the data on the right.

Example:

```elixir
name = "Daniel"
name #=> "Daniel"
name = "Ash"
name #=> "Ash"
```

> **Note**
> Rebinding a variable in Elixir doesn't make the values mutable. All it does is point the variable name to a new location in memory containing the new value.

### Multi-part matching
Complex, multi-part patterns can be used as well.

Example:

```elixir
{name, age} = {"Daniel", 24}

name #=> "Daniel"
age  #=> 24
```

> **CAVEAT**
> Variables can only be bound once per pattern, so the following will not work:

```elixir
{age, age} = {23, 24}
```

### Ignoring parts of a pattern match
You can use `_` to tell the pattern matcher to ignore parts of the match. This is so that you can deal with situations where you need to pattern match against say a tuple with two elements where you only care about on of those options. You can safely get the values that you want, whilst discarding those that you don't.

Example:

```elixir
{name, _} = {"Daniel", 24}

name #=> "Daniel"
_    #=> Unbound variable _ (to show that _ is not bound to 24)

# you can also name the underscored variable to make code easier
# to read and understand
{name, _age} = {"Ash", 30}
name #=> "Ash"
_age #=> warning: the underscored variable "_age" is used after being set. A leading underscore indicates that the value of the variable should be ignored. If this is intended please rename the variable to remove the underscore (a nice error that warns you to remove the underscore if you intend to access this variable later)
```

### Conditional matches
You can also use the match operator to make assertions. This enables you to conditionally make matches or bind values.

Example:

```elixir
# allows you to say only bind age if the first element is "Daniel"
{"Daniel", age} = {"Daniel", 24}
age #=> 24

# this will fail with a MatchError and therefore won't bind age
{"Daniel", age} = {"Ash", 30}
#=> (MatchError) no match of right hand side value: {"Ash", 30}
```

This can be used to quickly fail in certain scenarios. For example, it is an Elixir idiom to use tuples when dealing with IO where the first element is an atom that can be pattern matched on.

Consider the following example where a file is being read. It is impossible to know for sure when writing the code that the file specified will exist at run time. As such we can pattern match against `:ok` to deal with what will happen if the file exists, and pattern match against `:error` in scenarios where something has gone wrong.

Example:

```elixir
# if the file on the right hand side exists, then the contents variable
# is bound
{:ok, contents} = File.read("exists.txt")
#=> {:ok, "Contents of text file.\n"}

# if the file on the right hand side doesn't exist then a MatchError
# is thrown
{:ok, contents} = File.read("does_not_exist.txt")
#=> (MatchError) no match of right hand side value: {:error, :enoent}

# we can then gracefully handle errors that result from attempting
# to read the file
{:error, error_type} = File.read("does_not_exist.txt")
error_type #=> :enoent
```

### Matching on assigned variables
As was mentioned previously Elixir variables, unlike Erlang variables, can be rebound.

```elixir
name = "Daniel"
name #=> "Daniel"
name = "Ash"
name #=> "Ash"
```

But what if we want to pattern match on the current binding of a variable rather than rebind it? We can use the `^` (pin) operator for this.

Example:

```elixir
name = "Daniel"
{^name, age} = {"Daniel", 24}
name #=> "Daniel"
age  #=> 24

{^name, age} = {"Ash", 30}
#=> (MatchError) no match of right hand side value: {"Ash", 30}
```

Without the pin operator the following would happen:

```elixir
name = "Daniel"
{name, age} = {"Daniel", 24}
name #=> "Daniel"
age  #=> 24

{name, age} = {"Ash", 30}
name #=> "Ash"
age  #=> 30
```

If you try to pin a variable before it has been bound, you get an error.

```elixir
^name = "Daniel"
#=> (CompileError) iex:1: unbound variable ^name
```

### What Elixir types can be matched?
All Elixir types can be matched, including Maps, Structs and Binaries.

Examples:

```elixir
# matching Maps
my_map = %{name: name, age: age}
my_map = %{name: "Daniel", age: 24}
my_map.name #=> "Daniel"
my_map.age  #=> 24

# matching Structs
defmodule Author do; defstruct name: "", age: 0; end
my_struct = %Author{}
my_struct = %Author{name: "Daniel", age: 24}
my_struct.name #=> "Daniel"
my_struct.age  #=> 24

# matching Binaries
"Hello, " <> word = "Hello, World!"
word #=> "World!"
```

## Math operators
Standard math operators are included.

Examples:

```elixir
1 + 1 #=> 2
5 - 2 #=> 3
3 * 5 #=> 15
8 / 2 #=> 4.0 (always returns a float, even if result is a whole number)
```

Also included is the remainder operator `rem` (same as `modulus`/`%` in other languages).

Examples:

```elixir
rem(15, 5) #=> 0
rem(5, 2)  #=> 1
```

## Comparison operators
Used for comparing values.

### Equality
`==` or `===`

Examples:

```elixir
"Daniel" == "Daniel"  #=> true
1 == 2                #=> false
1 == 1.0              #=> true
1 === 1.0             #=> false
1 == "1"              #=> false
```

### Inequality
`!=` or `!==`

Examples:

```elixir
1 != 2    #=> true
1 != 1.0  #=> false
1 !== 1.0 #=> true
```

### Greater than/less than

Compare values with `>`, `>=`, `<`, `<=`

Examples:

```elixir
2 > 1  #=> true
2 >= 2 #=> true
1 < 2  #=> true
2 <= 2 #=> true
```

## List operators

### Assert an element is present in a list
`in`

Examples:

```elixir
"Daniel" in ["Ash", "Leslie", "Dori"] #=> false
"Daniel" in ["Daniel", "Ash"]         #=> true
```

### Combine two or more lists
`++`

> **CAVEAT**
> Remember the performance issues related with this.

Example:

```elixir
[1, 2, 3] ++ [4] #=> [1, 2 ,3 ,4]
```

### Remove members from a list
`--`

> **CAVEAT**
> Remember the performance issues related with this.

Example:

```elixir
[1, 2, 3] -- [1, 3] #=> [2]
```

### Prepend to a list
`|`

Example:

```elixir
list = [1, 2, 3]
[0  | list] #=> [0, 1, 2, 3]
```

The prepend operator can also be used in complex list patter matches.

Examples:

```elixir
[head | tail] = [1, 2, 3]
head #=> 1
tail #=> [2, 3]

[a, b, c | tail ] = [1, 2, 3, 4]
a    #=> 1
b    #=> 2
c    #=> 3
tail #=> [4]
```

## Binary operators

### Concatenate two or more binaries
`<>`

Example:

```elixir
"Hello" <> " " <> "World!" #=> "Hello World!"
```

### Interpolate values into binaries
`#{}`

> **Note**
> This is not a true operator (so what is it?).

Examples:

```elixir
"The value of one plus two is: #{1 + 2}"
#=> "The value of one plus two is: 3"

name = "Daniel"
"Hello, #{name}!"
#=> "Hello, Daniel!"
```

### Compare binary to a pattern
`=~`

Examples:

```elixir
"Goodbye" =~ ~r/Good/ #=> true
"Goodbye" =~ "Good"   #=> true
"Hello"   =~ "World"  #=> false
```

### Bitwise operators
Work directly on binary data. Check out the `Bitwise` module for more info.

## Logical operators

### Assert two expressions are truthy
`and` or `&&`

> **Note**
> `and` and `&&` are not exact equivalents. `and` requires that the value on the left hand side be a boolean, whereas `&&` will allow any value that is truthy/falsy

Examples:

```elixir
%{name: name, age: age} = %{name: "Daniel", age: 24}

name == "Daniel" and age > 23 #=> true
name == "Daniel" && age > 23  #=> true
name == "Daniel" && age       #=> 24
name == "Daniel" and age      #=> 24
name && age                   #=> 24
name and age                  #=> (ArgumentError) argument error: "Daniel"
```

> **Note**
> With `and` the value on the right hand side of the last comparison can be non-boolean. As such you can use the boolean expression to return the last value providing all the former assertions were true.

For example:

```elixir
%{name: name, age: age, likes: likes} = %{name: "Daniel", age: 24, likes: "Elixir"}

name and age and likes
#=> ** (ArgumentError) argument error: "Daniel"

name == "Daniel" and age and likes
#=> ** (ArgumentError) argument error: 24

name == "Daniel" and age > 23 and likes
#=> "Elixir"

name == "Daniel" and age < 23 and likes
#=> false

```

Of course if you have assertions for each part of the expression then `true` or `false` will be returned.

```elixir
name == "Daniel" and age > 23 and likes == "Elixir"
#=> true

name == "Daniel" and age > 23 and likes == "rotten fruit"
#=> false
```

### Return the first truthy expression
`or` or `||`

> **Note**
> `or` and `||` are not exact equivalents. `or` requires that the value on the left hand side be a boolean, whereas `||` will allow any values that are truthy/falsy

Examples:

```elixir
"Daniel" || nil #=> "Daniel"
nil || "Daniel" #=> "Daniel"
nil || false    #=> false (if all expressions are falsy last is returned)

# use `or` to set a default value
name = user.name || "John Smith"
```

## Pipeline operator
`|>`

Used to chain function together into a pipeline, hence the name.

Examples:

```elixir
defmodule PipelineTest do
  def foo(val) do
    "foo: #{val}"
  end
  
  def bar(val) do
    "bar: #{val}"
  end
end

# longhand
var = "hello"               #=> "hello"
var = PipelineTest.foo(var) #=> "foo: hello"
var = PipelineTest.bar(var) #=> "bar: foo: hello"

# becomes
var = var |> PipelineTest.foo |> PipelineTest.bar #=> "bar: foo: hello"
      
# which translates to
var = PipelineTest.bar(PipelineTest.foo(var)) #=> "bar: foo: hello"
```