## Prerequisites
In order to work with the code base, you'll first need to install the following.

* **Visual Studio 2015**

## Clone the repository
Using your favorite [git](http://git-scm.com/) client, clone the repository.

``` Batchfile
git clone https://github.com/aspnet/EntityFramework.git
```

## Build from Visual Studio
## 使用 Visual Studio 构建
Before opening the solution, you'll need run the following from Command Prompt.
>在打开解决方案前，你需要在命令提示符中运行以下命令。
Navigate to the root directory of the solution (~"/EntityFramework") in your command prompt, and issue the command:
>在命令提示符中导航到解决方案的根目录（~"/EntityFramework"），并运行命令：

``` Batchfile
build initialize
```

This will download the [NuGet](http://www.nuget.org/) packages that we depend on, and reference them from the projects.
>这将下载我们依赖的  [NuGet](http://www.nuget.org/) 包，并从项目中引用它们。

You should be able to open the solution in Visual Studio now.
>现在你应该可以在 Visual Studio 中打开解决方案了。

**Solving common `build initialize` errors**
>**解决常见的 `build initialize` 错误**

1. Check that <https://myget.org/gallery/aspnetcidev> and <https://nuget.org> are accessible.
>1. 检查 <https://myget.org/gallery/aspnetcidev> 和 <https://nuget.org> 是否可以访问。
2. Clean the source directory. `git clean -xid` will clean files in the EF source directory. 
>2. 清理源代码目录。`git clean -xid` 将在 EF 源代码目录中清理文件。
3. Clear nuget packages and caches. `nuget.exe locals -clear all` will delete the NuGet caches. (You can get nuget.exe from <https://dist.nuget.org/index.html>).
>3. 清理 nuget 包以及缓存。`nuget.exe locals -clear all` 将删除 NuGet 缓存。（你可以从 <https://dist.nuget.org/index.html> 获取 nuget.exe）。
4. Reinstall .NET Core CLI. Our build script automatically installs a version to `%LOCALAPPDATA%\Microsoft\dotnet`.
>4. 重新安装 .NET Core CLI。我们的构建脚本自动安装一个版本到 `%LOCALAPPDATA%\Microsoft\dotnet`。

### Run tests
### 运行测试

Our tests are written using [xUnit.net 2.0](https://github.com/xunit/xunit), and can be run [TestDriven.Net](http://www.testdriven.net/). Given the rapid iteration of the pre-release platform we are building on top of, we do not ensure that other test runners work. Once things stabilize, we will support other test runners. 
>我们的测试是使用 [xUnit.net 2.0](https://github.com/xunit/xunit) 编写的，并且可以通过 [TestDriven.Net](http://www.testdriven.net/) 运行。
鉴于我们处在预发布平台的快速迭代期间，我们不保证在其它测试框架下可以工作。一旦事情稳定下来，我们将提供其它测试框架。

## Build from Command Prompt
## 使用命令提示符构建
You can also build and run tests from Command Prompt using the following.
>你还可以使用下面的命令来在命令提示符中构建和运行测试。
``` Batchfile
build
```