## Prerequisites
In order to work with the code base, you'll first need to install the following.

* **Visual Studio 2015**

## Clone the repository
Using your favorite [git](http://git-scm.com/) client, clone the repository.

``` Batchfile
git clone https://github.com/aspnet/EntityFramework.git
```

## Build from Visual Studio
Before opening the solution, you'll need run the following from Command Prompt.
Navigate to the root directory of the solution (~"/EntityFramework") in your command prompt, and issue the command:

``` Batchfile
build initialize
```

This will download the [NuGet](http://www.nuget.org/) packages that we depend on, and reference them from the projects.

You should be able to open the solution in Visual Studio now.

**Solving common `build initialize` errors**

1. Check that <https://myget.org/gallery/aspnetcidev> and <https://nuget.org> are accessible.
2. Clean the source directory. `git clean -xid` will clean files in the EF source directory. 
3. Clear nuget packages and caches. `nuget.exe locals -clear all` will delete the NuGet caches. (You can get nuget.exe from <https://dist.nuget.org/index.html>).

### Run tests

Our tests are written using [xUnit.net 2.0](https://github.com/xunit/xunit), and can be run [TestDriven.Net](http://www.testdriven.net/). Given the rapid iteration of the pre-release platform we are building on top of, we do not ensure that other test runners work. Once things stabilize, we will support other test runners. 

## Build from Command Prompt
You can also build and run tests from Command Prompt using the following.
``` Batchfile
build
```