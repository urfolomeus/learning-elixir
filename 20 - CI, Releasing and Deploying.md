# Standing on the shoulders ...

* The docker and distillery part of these notes were gleaned from a tutorial that Paul sent me <https://github.com/plamb/deploying-elixir/blob/master/docs/distill_with_docker_pt1.md>, the Phoenix parts from the Phoenix Dockerfile from the same repo <https://github.com/plamb/deploying-elixir/blob/master/docker/Dockerfile.build.phoenix>.
* The Phoenix config came from the [Phoenix advanced deployment docs](http://www.phoenixframework.org/docs/advanced-deployment)
* The setup for Debian came from the [DigitalOcean docs](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-debian-8) although they should be valid for any platform.
* The nginx setup came from the [Phoenix serving as proxy docs](http://www.phoenixframework.org/docs/serving-your-application-behind-a-proxy)

Thanks to Paul for the help and direction and Scott for suggesting ideas on the env var front.

# Adding Travis support

We can use [Travis CI](https://travisci.org) as our Continuous Integration server. Create an account, attach it to your Github and then flip the switch for this project.

Now add the following to a **.travis.yml** file on the root of your umbrella project.

```elixir
language: elixir
elixir:
  - 1.4.0
otp_release:
  - 19.1
script:
  - mix test
  - mix credo

```

# Deploying using Distillery and Docker

## Distillery setup

First add Distillery to your root **mix.exs** file.

```elixir
[{:credo, "~> 0.5", only: [:dev, :test]},
 {:distillery, "~> 1.0"}]
```

Then run `mix do deps.get, deps.compile` to get it. Once Distillery is installed, run `mix release.init` to create a **rel/config.exs** file. The file should look something like this:

```elixir
# Import all plugins from `rel/plugins`
# They can then be used by adding `plugin MyPlugin` to
# either an environment, or release definition, where
# `MyPlugin` is the name of the plugin module.
Path.join(["rel", "plugins", "*.exs"])
|> Path.wildcard()
|> Enum.map(&Code.eval_file(&1))

use Mix.Releases.Config,
    # This sets the default release built by `mix release`
    default_release: :default,
    # This sets the default environment used by `mix release`
    default_environment: :dev

# For a full list of config options for both releases
# and environments, visit https://hexdocs.pm/distillery/configuration.html

# You may define one or more environments in this file,
# an environment's settings will override those of a release
# when building in that environment, this combination of release
# and environment configuration is called a profile

environment :dev do
  set dev_mode: true
  set include_erts: false
  set cookie: :"~ijtMQ~Dh7T4=9T(L?k,B5P&sXkRn9BZwiN)tM|h4k92Kp`,(/)ai|HTYnWUR^lQ"
end

environment :prod do
  set include_erts: true
  set include_src: false
  set cookie: :"N~<,J$S.Wda.PJ%2bM^a2I?oxw%hE?gU(K,jfi6L3X6*iL_SF&V2zEdVQ|(tM~=3"
end

# You may define one or more releases in this file.
# If you have not set a default release, or selected one
# when running `mix release`, the first release in the file
# will be used by default

release :my_app do
  set version: current_version(:my_app)
  set output_dir: './releases/my_app'
  set applications: [
    app_1: :permanent,
    app_2: :permanent,
    ...
  ]
end

```

In the `release` section at the bottom, make sure that you set the version and that you have all of the applications that you need to run the project. We're going to use a **releases** folder on the root of the project to store our releases rather than use the default of **_build/rel**. We'll create this folder shortly.

## Docker setup

Now that we have Distillery in place, we'll add the required files to get Docker setup.

At the root of your project add a **.dockerignore** file with the following content:

```dockerfile
.dockerignore
.gitignore
.git
.log
.credo
tmp

# docker directory with Dockerfiles and scripts
docker

# Mix artifacts
_build
deps
*.ez

# directory that contains the releases
releases

# dump files
erl_crash.dump

# Phoenix static artifacts
apps/**/node_modules
apps/**/priv # NOTE if you are not compiling your assets using Brunch or some other tool, don't exclude priv or you'll not get any assets.

# any other directories that have files that don't need to be included in the build

```

Create a folder called releases and ensure that it has enough privs for docker build to be able to write to it.

```sh
mkdir releases && chmod 0777 releases
```

Now add this folder to **.gitignore** because we don't want built artifacts clogging up our source control.

```
# The directory Mix will write compiled artifacts to.
/_build
/releases

# etc...
```

Now create a file **Dockerfile.build.elixir** with the following contents:

```dockerfile
FROM elixir:1.4.0

MAINTAINER Your Name <you@yourdomain.com>

ENV MIX_ENV prod

# Install hex
RUN /usr/local/bin/mix local.hex --force && \
    /usr/local/bin/mix local.rebar --force && \
    /usr/local/bin/mix hex.info

WORKDIR /app
COPY . .

RUN mix do deps.get, deps.compile, compile

CMD ["bash"]

```

This uses the `elixir:1.4.0` docker container (which includes Erlang) as the base that we will build on. It sets the `MIX_ENV` environment variable on the docker instance to `prod` and installs `hex` and `rebar`.

Then it copies all files not ignored by the **.dockerignore** file into a working directory called `app` that we create.

The `CMD` instruction sets the command to be executed when running the image. You can only have one per dockerfile. In this instance it runs all instructions using Bash.

## Phoenix application setup

There are a couple of extra steps that you need to take if the project that you are installing contains one or more phoenix applications.

First step is to configure the production environment. In the **config/prod.exs** file of your Phoenix application(s), make the following changes:

```elixir
config :interface_web, InterfaceWeb.Endpoint,
  http: [port: {:system, "PORT"}],
  url: [host: "my-domain.com", port: {:system, "PORT"}],
  cache_static_manifest: "priv/static/manifest.json"
  root: ".",
  version: Mix.Project.config[:version]
```

This lets you get the port from an environment variable on the host server, which gives you flexibility when deploying to different VMs.

From the Phoenix exrm docs:

> When generating a release and performing a hot upgrade, the static assets being served will not be the newest version without setting the root to . and setting the endpoint version. If root or version are not set then we will be forced to perform a restart so that the correct static assets are served, effectively changing the hot upgrade to a rolling restart.

Add the following line to **config/config.exs** in your Phoenix application(s).

```elixir
config :interface_web, InterfaceWeb.Endpoint,
  # etc ...
  server: true
```

Setting `server` to `true` starts the Phoenix web server when the supervision tree starts. This allows you to run the web server when running `mix run --no-halt` for the umbrella project.

We now want to ensure that all static assets are compiled and built when a release is made. In the **docker/Dockerfile.build.elixir** file, make the following change:

```dockerfile
RUN mix do deps.get, deps.compile, compile

# install node dependencies and output static assets
# do this after mix deps.get since the phoenix & phoenix_html
# node modules reference files in these dependencies
RUN npm install \
  && node node_modules/brunch/bin/brunch build \
  && mix phoenix.digest

CMD ["bash"]

```

## Setting up a VM

You'll now need to setup a server to deploy to. It's up to you how you want to do this. I use either AWS or DigitalOcean, but you can use any. I would recommend using Debian 8 as the OS as that's what the Docker instance we use to build the project will be using, but you can probably use any Linux OS, especially if it's based on Debian (i.e. Ubuntu).

Once you've provisioned the machine, SSH in as the root user and create a non-root user with sudo access. This user will be used by the deploy script to deploy to the VM.

```sh
# creat a non-root user
adduser <user_name>

# on Debian you need to install sudo, most other distros come
# with sudo pre-installed
apt-get update
apt-get install sudo

# now grant sudo access to your new user
usermod -a -G sudo <user_name>

# generate an SSH key on your local machine
ssh keygen -t RSA -C <your email address>
cat ~/.ssh/id_rsa.pub | pbcopy

# and add it to the user you just created on the host machine
su - <user_name>
mkdir .ssh
chmod 700 .ssh
vi .ssh/authorized_keys # and paste in the key you copied
chmod 600 .ssh/authorized_keys
exit
```

Now you can log out as root and SSH back in as the user you just created.

If you have a Phoenix application in your umbrella you will want to install nginx to make your application available to the outside world. On Debian this is just a case of `sudo apt-get install nginx -y`. Once nginx is installed, create a configuration for your project in **/etc/nginx/sites-available/<project_name>**:

```sh
upstream phoenix {
  server 127.0.0.1:8080 max_fails=5 fail_timeout=60s;
}

server {
  server_name your-domain.com  www.your-domain.com;
  listen 80;

  location / {
    allow all;

    # Proxy Headers
    proxy_http_version 1.1;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_set_header X-Cluster-Client-Ip $remote_addr;

    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";

    proxy_pass http://phoenix;
  }
}
```

This setup allows you to proxy Phoenix from the port it is running on (in this case 8080) to port 80. Any traffic hitting either **your-domain.com** or **www.your-domain.com** will be directed to your Phoenix application.

In order to get it working you will need to symlink this file into **/etc/nginx/sites-enabled** and restart nginx.

```sh
ln /etc/nginx/sites-available/<project_name> /etc/nginx/sites-enabled/<project_name>
sudo service nginx restart
```

## Deploying to the VM

We're going to use a bash script to run our deployments so that they are repeatable. From the root of the umbrella project create a **deploy/deploy.sh** file and add the following contents:

```sh
#!/usr/bin/env sh

cd `dirname $0`/..
version=`head VERSION`
project="my_project"
ssh_to="project@host"

remote_ex(){
  ssh -A $ssh_to $1
}

docker build -t build-project -f docker/Dockerfile.build.elixir .
docker run -v $PWD/releases:/app/releases build-project mix release --env=prod

scp "releases/$project/releases/$version/$project.tar.gz" "$ssh_to:"

remote_ex "rm -rf ${project}_release.old"
remote_ex "mv ${project}_release ${project}_release.old"
remote_ex "mkdir -p ${project}_release"
remote_ex "cd ${project}_release && tar zxvf ../${project}.tar.gz"

remote_ex "${project}_release.old/bin/$project stop"
remote_ex "${project}_release/bin/$project start"

```

This file first sets some variables (you'll want to adjust these to match your project and the host machine that you setup) and then creates a convenience function for running commands on the host machine.

Now the project is built using `docker build` and `docker run`. In case you are wondering why we use both `docker build` and `docker run`, the tutorial that I followed for doing Docker and Distillery states the following:

> You can only mount volumes when running the docker run command. To get around this, we could have done a RUN mix release in the Dockerfile and then used a docker cp to copy the file out. But this means that the commands run on the container are hard coded into the container. Instead, we use a CMD [bash] to tell the container what to execute when a docker run is issued without any arguments, we then override that command with the final option mix release --env=prod. This gives us an easy way to specify options and commands to the container, i.e. mix release.upgrade --env=prod.

After creating the release in the Docker instance we secure copy the resulting **<project_name>.tar.gz** file to the host machine that we set up.

**PLEASE NOTE** we are only using Docker as a convenient way to package up the release. We are **not** creating a Docker image and then deploying the image to the VM.

The rest of the commands
* get rid of the previous old release
* make the current release the old release
* unpack the tarballed new release and make it the current release
* stop the old application
* start the new application

Once this script finishes running you should be able to see your application running.
