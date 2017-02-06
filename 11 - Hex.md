# Hex
Hex is a package manager for the Erlang ecosystem. Hex packages in Elixir/Erlang are analagous to gems in Ruby. It integrates seamlessly with `mix`.

## Finding a hex package
- @h4cc's [Awesome Elixir list](https://github.com/h4cc/awesome-elixir)
- [Hex web search](https://hex.pm/)
- `mix hex.search <package_name>`

## Installing a hex package

### Specifying packages
Add `{:package_namw, opts}` to your `deps` function in your `mix.exs` file.

```elixir
defmodule MyProject.Mixfile do
  # ...
  
  def deps do
    [{:package_name, "1.0.0"}]
  end
end
```

There are many ways to specify which package version you want in your mix file.

```elixir
# only install version 1.0.0
{:package_name, "1.0.0"}

# install any version greater than 1.0.0
{:package_name, "> 1.0.0"}

# install any version between 1.0.0 and 1.1.0
{:package_name, "~> 1.0.0"}

# install any version between 1.0 and 2.0
{:package_name, "~> 1.0"}
```

You can also specify dependencies on git repos.

```elixir
# from Github
{:package_name, github: "username/repo"}

# from Git
{:package_name, git: "git@domain.tld:username/repo"}

# using tag
{:package_name, github: "username/repo", tag: "v1.0.0"}

# using branch
{:package_name, github: "username/repo", branch: "release"}

# using commit SHA
{:package_name, github: "username/repo", ref: "17bbd278f9c"}
```

Other options include:

```elixir
# only install/use this package in the given environment
# this is especially useful if you publish your package on Hex
# and have libs that users of your package don't need but maintainers do
{:package_name, only: :test}

# override another package's dependency
# prevents a package from forcing you to use a version of another package
# that you don't want to use
{:package_name, "2.0.0", override: true}
```

### Getting specified dependencies
`mix deps.get`
- downloads packages from Hex/Git
- caches Hex packages on your file system (Hex will only install 1 copy of a given version of a given package. This not only saves space on your machine but also makes subsequent downloads faster. It also allows you to install common packages in projects when offline.
- places dependencies in your project's `deps` folder

`mix compile`
Compiles all files in your project, including all dependencies.

## Removing a dependency
Remove it from `deps` in your `Mix,exs` file and then run `mix deps.clean <package_name>`.

## Configuring dependencies
Add settings for your dependencies using `config/2` in the `config/config.exs` file (or equivalent environment files if that makes more sense):

```elixir
config :package_name, setting_a: "value", setting_b: "value
```

Most Hex packages will tell you in their README what needs to go in the config files.

## Uploading to Hex
**IMPORTANT** Project name must be unique in order to publish to Hex.

Add `package/0` settings to your Mixfile.

```elixir
defp package do
  [
    files: ["lib", "mix.exs", "README.md"],
    contributors: ["Your name"],
    licenses: ["MIT"] # or whatever license you want
    links: %{
      "Github" => "https://github.com/username/repo"
    }
end
```

Update the `application/0` settings, specifying each of your runtime dependencies as an application in the :applications list:

```elixir
def application do
  [applications: [:dependency_a, :dependency_b]]
end
```

Update `project/0` settings in the Mixfile:

```elixir
def project do
  [
    ...
    version: "version number to publish",
    ...
    source_url: "https://gihub.com/username/repo",
    description: "Short description",
    package: package,
    deps: deps
  ]
end
```

Once all that has been done, publish to Hex with `mix hex.publish`.

Use `mix help | grep hex` to get the full list of hex tasks:

```elixir
mix hex               # Prints Hex help information
mix hex.build         # Builds a new package version locally
mix hex.config        # Reads or updates Hex config
mix hex.docs          # Publishes docs for package
mix hex.info          # Prints Hex information
mix hex.key           # Hex API key tasks
mix hex.outdated      # Shows outdated Hex deps for the current project
mix hex.owner         # Hex package ownership tasks
mix hex.publish       # Publishes a new package version
mix hex.registry      # Hex registry tasks
mix hex.search        # Searches for package names
mix hex.user          # Hex user tasks
```

**INFO**
`mix hex.outdated` will let you find all outdated hex packages in your project.