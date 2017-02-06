# Applications
> "In OTP, application denotes a component implementing some specific functionality, that can be started and stopped as a unit, and which can be re-used in other systems as well."
> *OTP documentation*

> **IMPORTANT**
> Every Mix project is an OTP application, whether or not it is supervised.

That's what the configuration in the `mix.exs` file is all about. Apps have a name, a version, any dependencies that they might require and, importantly, a list of applications that need to be booted up to help this application do its job.

You can learn more about mix's OTP options by typing `mix help compile.app` at the command line.

## Buzzword compliance
Long before the age of microservices, OTP:
- encouraged isolated components
- allowed these components to run on separate hardware
- solved the microservice communication problem

Mix just makes these things a bit easier to manage through the use of umbrella apps.

So, by using mix and OTP, you get all of the benefits of microservices with none of the pain.

## Demo
![diagram](/Users/alan/Dropbox/Screenshots/todo_diagram.png)

Create a new supervised project using `mix new todo --sup`

See [the demo code](git@bitbucket.org:urfolomeus/learnelixir.tv/ch16/teedee) from more info.