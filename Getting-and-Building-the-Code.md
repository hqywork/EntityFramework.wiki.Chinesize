## Prerequisites
In order to work with the code base, you'll first need to install the following.

* [Visual Studio 2015](http://www.visualstudio.com/en-us/downloads/visual-studio-2015-downloads-vs)
* [K Version Manager](https://github.com/aspnet/Home/wiki/version-manager)

## Clone the repository
Using your favorite [git](http://git-scm.com/) client, clone the repository.

``` Batchfile
git clone https://github.com/aspnet/EntityFramework.git
```

## Build from Visual Studio
Before opening the solution, you'll need run the following from Command Prompt.
Navigate to the root directory of the solution (~"/EntityFrameWork") in your command prompt, and issue the command:

``` Batchfile
build initialize
```

This will download the [NuGet](http://www.nuget.org/) packages that we depend on, and reference them from the projects.

You should be able to open the solution in Visual Studio now.

### Run tests

Our tests are written using [xUnit.net 2.0](https://github.com/xunit/xunit), and can be run using your favorite runner. We recommend either the [xUnit.net runner for Visual Studio 2013](http://visualstudiogallery.msdn.microsoft.com/463c5987-f82b-46c8-a97e-b1cde42b9099) or [TestDriven.Net](http://www.testdriven.net/).

## Build from Command Prompt
You can also build and run tests from Command Prompt using the following.
``` Batchfile
build
```