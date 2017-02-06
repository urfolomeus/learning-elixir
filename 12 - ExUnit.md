# ExUnit
Elixir has a strong testing culture and ExUnit is the main tool used.

## Starting ExUnit
Because ExUnit is an OTP application that needs to manage multiple processes in order to run your tests, it needs to be started. This is usually done in the `tests/test_helper.exs` file in an Elixir project. If you used `mix new` to generate your project then this will have already been done for you.

```elixir
# tests/test_helper.exs
ExUnit.start
```

## ExUnit Test Anatomy

```elixir
# tests/math_test.exs

defmodule MyApp.MathTest do
  use ExUnit.Case
  
  test ".add sums two numbers" do
    assert Math.add(1, 2) == 3
  end
end
```

### Assert
Check that some expression returns a truthy result.

```elixir
assert Math.add(1, 2) == 3
assert 5 > 3
assert 2 <= 4
assert Math.subtract(2, 1) != nil

# the above will give you a nice error message that accurately explains
# what went wrong. However you can override that message with your own
assert customer != nil, "Customer was not created in the database"

# some other assertion options
assert_raise     # Assert that a code block raises an exception
assert_in_delta  # Assert that two things differ in a specific way
assert_receive   # Assert that a process message was received
```

### Refute
The mirror opposite of `assert`.

```elixir
refute customer == nil, "Customer was not created in the database"
```

### Custom assertions
Because assertions in Elixir are just functions, you can easily make your own assertions, which can be made of smaller assertions.

```elixir
def assert_customer_created(data) do
  assert data != nil
  assert data.customer != nil
end
```

### Shared setup

```elixir
defmodule MyApp.MathTest do
  use ExUnit.Case
  
  setup do
    variable = "some value"
    {:ok, variable: variable}
  end
  
  test "some test", %{variable: variable} do
    # use variable here
  end
end
```

> **INFO**
> Check out `ExUnit.CaseTemplate` in a future episode to see another way to share setup.

## Running tests asynchronously
If your tests in a module don't rely on any shared state, such as a database, then you can run your tests asynchronously.

```elixir
defmodule MyApp.MathTest do
  use ExUnit.Case, async: true
  
  # ...
end
```

## Tagging

```elixir
# this would tag all the tests in the module as slow
@moduletag :slow

# this would only tag all the one test
@tag :slow
test "a really, really, slow test" do
  # ...
end

# you can pend tests with the built-in :skip tag
@tag :skip
@tag skip: "Reason"

# if you create tests that have no implementation they will fail
# automatically rather than be marked as pending as in the likes of RSpec
# note: these just get tagged :not_implemented so you could skip them as
# below if desired
test ".add sums two numbers"
test ".div divides two numbers"
test ".mul multiplies two numbers"

# then in your tests/test_helper.exs file
ExUnit.start
ExUnit.configure exclude: [:slow]
```

## Doctests
Allows you to test your inline documentation. You write them by indenting by 4 spaces and then writing `iex>` followed by the expression you'd like to test. You can run the expression over multiple lines by using `...>` on each subsequent line. Finally, on a line of its own at the end, put the expected result.

```elixir
defmodule MyApp.Math do
  @doc """
  Adds two numbers together and returns the sum.
  
  ## Examples
  
      iex> Math.add(1, 2)
      3
      
      iex> sum = Math.add(5, 5)
      ...> Math.add(sum, 5)
      15
  """
  def add(a, b), do: a + b
end

# then in your tests/math_test.exs file add a `doctest <module_name>` line
defmodule MyApp.MathTest do
  use ExUnit.Case
  
  doctest MyApp.Math
end
```

Doctest have the following benefits:
- your documentation can be relied on
- if your code changes your doctests will remind yot to update your docs

> **OPINION**
> Prefer doctests to normal tests as this will make for better documentation. Only use normal tests if there is a lot of setup required, functionality not available in doctests is required (i.e. `asset_raise`), or the resulting doctest would be too large to comfortably fit within the docs.
> 
> That said, if a test cannot be described easily in a doctest then that might hint that the code under test is too complex and could do with being further broken down.
> 
> 1. Write tests on every public function
> 2. If you don't want to test a function, make it private or set `@doc false`
> 3. Prefer doctests to regular tests

## Running the tests

```elixir
# in a mix project
> mix test
> mix test --exclude slow  # don't include tests/modules with :slow tag
> mix test --include slow  # include tests/modules with :slow tag
> mix test --only slow     # only run tests/modules with :slow tag

# otherwise
> elixir my_test.exs
```