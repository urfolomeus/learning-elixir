# Mix
Mix is the build tool for Elixir. It handles:
- Project organisation
- Dependency management
- Build tasks
- Custom tasks

## Creating a new mix project

```elixir
> mix new <project_name>
> cd <project_name>
```

Or, if you want to create an umbrella project

```elixir
> mix new <project_name> --umbrella
```

Then you can cd into the `apps` folder of your umbrella project and create as many apps as you want using `mix new`.

Example:

```elixir
> mix new big_project --umbrella
> cd big_project/apps
> mix new component_a
> mix new component_b
```

If you have an umbrella project you can run mix tasks from the root of the umbrella project and it will run that mix task in all of its apps.

Example:

```elixir
> mix test
==> component_a
.

Finished in 0.06 seconds (0.06s on load, 0.00s on tests)
1 test, 0 failures

Randomized with seed 102541
==> component_b
.

Finished in 0.00 seconds
1 test, 0 failures
```

## Starting iex with your mix project compiled

```elixir
> iex -S mix
```

## Config
Add your configuration to the `config/config.exs` file.

Examples:

```elixir
# config/config.exs
use Mix.Config
config :crypto, prefix: "Encrypted: "

# lib/crypto/encryptor.ex
defmodule Crypto.Encryptor do
  @prefix Application.get_env(:crypto, :prefix)

  def encrypt(text) do
    @prefix <> String.reverse(text)
  end
end
```

```elixir
# config/config.exs
use Mix.Config
config :crypto, Crypto.Encryptor: "Encrypted: "

# lib/crypto/encryptor.ex
defmodule Crypto.Encryptor do
  @prefix Application.get_env(:crypto, __MODULE__)[:prefix]

  def encrypt(text) do
    @prefix <> String.reverse(text)
  end
end
```

To use environment specific variables:

```elixir
# config/config.exs
import_config "#{Mix.env}.exs"

# config/dev.exs
config :crypto, Crypto.Encryptor: "Dev: "

# config/test.exs
config :crypto, Crypto.Encryptor: "Test: "

# config/prod.exs
config :crypto, Crypto.Encryptor: "Prod: "
```

## Tasks

### help
`mix help` shows you the available mix tasks.

### compile
`mix compile` builds the project for the given environment (i.e. `MIX_ENV=prod mix compile` will build the project for the production environment).

The compiled files will be stored in the `_build` folder under a folder for the environment.

### test
`mix test` runs the test suite.

## Writing your own mix tasks
Create a new folder inside your `lib` folder called `mix/tasks`. Mix tasks require the following boilerplate:

```elixir
defmodule Mix.Tasks.<task_name> do
  @shortdoc "Description to appear in the output of `mix help`"

  use Mix.Task

  def run(args) do
    # do what needs to be done
  end
end
```

Example:

```elixir
defmodule Mix.Tasks.Encrypt do
  @shortdoc "Encrypts some arbitrary text"
  
  @moduledoc """
  Takes a `-t` option which specifies which text to encrypt.

      mix encrypt -t hello
  """

  use Mix.Task

  def run(args) do
    {opts, _, _} = OptionParser.parse(args, aliases: [t: :text])
    IO.puts Crypto.Encryptor.encrypt(opts[:text])
  end
end
```

**INFO** Remember to `mix compile` after making changes to the task.

Running `mix help` will include the shortdoc in the list.

```elixir
> mix compile
> mix help

mix                   # Runs the default task (current: "mix run")
mix app.start         # Starts all registered apps
...
mix encrypt           # Encrypts some arbitrary text
...
```

Running `mix help <task_name>` will display the moduledoc.

```elixir
> mix compile
> mix help encrypt

                                  mix encrypt

Takes a -t option which specifies which text to encrypt.

┃ mix encrypt -t hello

Location: _build/dev/lib/crypto/ebin
```