# Package structure

```
Microsoft.EntityFrameworkCore.Tools
├── lib
│   ├── net451
│   │   └── Microsoft.EntityFrameworkCore.Tools.dll <-- PowerShell OperationExecutor is here
│   └── netcoreapp1.0
│       └── dotnet-ef.dll
└── tools
    └── EntityFramework.psm1

Microsoft.EntityFrameworkCore.Tools.Cli
└── lib
    ├── net451
    │   └── Microsoft.EntityFrameworkCore.Tools.Cli.exe <-- .NET Core CLI OperationExecutor is here
    └── netcoreapp1.0
        └── Microsoft.EntityFrameworkCore.Tools.Cli.dll <-- .NET Core CLI OperationExecutor is here

```

# How command execution currently works


## .NET Core CLI on .NET Core projects

```
> dotnet ef $ARGS
  └─ dotnet.exe \
           $NUGET_CACHE/dotnet-ef.dll \
           $ARGS
     |
     |  (Gathers project model info)
     |  (Triggers a build)
     |  (Launches corehost with the "inside man")
     |
     └─ dotnet exec \
           --runtimeconfig $appname.runtimeconfig.json \
           --depsfile $appname.deps.json \
           $BUILD_OUTPUT/Microsoft.EntityFrameworkCore.Tools.Cli.dll \
           --assembly $full_path_to_appname.dll \
           $ARGS
        |
        |  (Uses command line arguments to identify operation and options)
        |
        └─ .NET Core CLI OperationExecutor
```

### How tool and app dependencies are merged

Because "Microsoft.EntityFrameworkCore.Tools.Cli" comes from dependencies,
"dotnet-restore" takes care of merging dependencies. The result of that merge 
is found in `$appname.deps.json` when the app runs.


## .NET Core CLI on .NET Framework projects
```
> dotnet ef $ARGS
  └─ dotnet.exe \
           $NUGET_CACHE/dotnet-ef.dll \
           $ARGS
     |
     |  (Gathers project model info)
     |  (Triggers a build)
     |  (Launches the "inside man")
     |
     └─ $BUILD_OUTPUT/Microsoft.EntityFrameworkCore.Tools.Cli.exe \
           --assembly $full_path_to_appname.dll \
           $ARGS
        |
        |  (Uses command line arguments to identify operation and options)
        |
        └─ .NET Core CLI OperationExecutor
```

### How tool and app dependencies are merged
If dependency conflicts exist, `dotnet-build` also drops a file
 "Microsoft.EntityFrameworkCore.Tools.Cli.exe.config" containing binding redirects
 in the build output folder.


## PowerShell cmdlets on csproj
```
PS > Add-Migration *
     └─ EntityFrameworkCore.psm1
        |
        |  (Gather project model info)
        |  (Triggers a build)
        |  (Ensure Microsoft.EntityFrameworkCore.Tools.dll is in app dependencies)
        |  (Creates a new app domain and load users built output into the domain)
        |
        └─ EntityFrameworkDesignDomain (in powershell host)
           |
           | (Powershell cmdlet identifies operation with options)
           | (Create instance of operation executor from Microsoft.EntityFrameworkCore.Tools.dll)
           |
           └─ PowerShell OperationExecutor
```

### How tool and app dependencies are merged

NuGet `Install-Package` resolves dependencies of "Microsoft.EntityFrameworkCore.Tools" 
and the app. The resolution is automatically put into csproj and results in an output
directory and `App.Config` that handles binding redirects.

## PowerShell cmdlets on xproj
```
PS > Add-Migration $ARGS
     └─ EntityFrameworkCore.psm1
        |
        |  (Translates cmdlet arguments to dotnet-ef equivalent"
        |
        ├─ "dotnet ef $ARGS --json"
        |
        └─ (Parse JSON results and trigger VS window opens)
```
This is a shim into "dotnet ef".