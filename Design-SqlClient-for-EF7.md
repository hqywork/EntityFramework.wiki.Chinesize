# SqlClient for EF7

The purpose of this design document is to define what Entity Framework requires from a SqlClient implementation to be used in EF7 with support for ASP.NET vNext on CoreCLR.

There are two separate components required to make this happen, System.Data.Common and a SqlClient implementation of System.Data.Common. This document covers the requirements for both of these components.

## System.Data.Common
System.Data.Common is designed to be a cross platform version of the ADO.NET provider model. The API surface is stripped down to only include the surface needed to build modern applications (i.e. legacy APIs/features are removed).

An initial implementation of System.Data.Common lives in [aspnet/DataCommon](https://github.com/aspnet/DataCommon/) repository and builds of this code base are being published to the [AspNetvNext MyGet feed](https://www.myget.org/F/aspnetvnext/api/v2/) (as the System.Data.Common NuGet package). Moving forward the SQL team will take ownership of this package moving it to another source control location. Since there are existing public pre-release components that build on System.Data.Common, the EF team will continue to use its implementation until an equivalent NuGet package is ready to replace it.

System.Data.Common is key to the relational provider model for EF7 and there are some important characteristics of the package that need to be maintained if another team takes ownership.

#### Implementation

The System.Data.Common contract is jointly owned by the SQL, EF and CLR teams. The initial implementation, and any subsequent changes, should be agreed on by all three teams.

#### Packaging
It is delivered as a NuGet package

It is licensed with the standard [Microsoft .NET Library EULA](http://www.microsoft.com/web/webpi/eula/net_library_eula_enu.htm). This license imposes minimal restrictions, allowing use on non-Windows platforms. 

#### Platforms
It needs to support the following platforms: net451, aspnet50, aspnetcore50, win81, and wpa81. Mono and Xamarin also need to be supported, we need to work out how to do this since they are currently at net45 but System.Data.Common relies on changes that were made in net451.

**Note:** _The NuGet package only needs to explicitly support net451 (implicitly satisfies aspnet50), win81 (implicitly satisfies aspnetcore50), and wpa81._

Even though we do not require SqlClient on all these platforms, there are other relational providers (e.g. SQLite) that make use of System.Data.Common on these platforms.

.NET 4.5.1 support is important (as opposed to later versions of .NET) since it is widely adopted and allows EF7 to be used in existing application. There is currently some discussion among the EF/SQL/CLR teams around this point which needs to be finalized.

The NuGet package should include a portable version of the assembly that targets all platforms (including net451). This allows other packages that depend on the System.Data.Common contract to be truly portable (i.e. no need to cross-compile). An example of this is the EntityFramework.Relational NuGet package.

There should be a net451 implementation of System.Data.Common that type forwards to the existing types in the .NET Framework. This avoids there being duplicate types for folks targeting full .NET. This avoids compilation issues, and also allows types like ```DbConnection``` from existing app code to be used with EF7.

The previous two requirements are best achieved by having the following two folders in the lib folder of the NuGet package:
 * **portable-net451+win81+wpa81:** Acts as the portable contract and also the implementation that is used on win81, wpa81 and aspnetcore50.
 * **net451:** acts as the implementation (with type forwarding) that is used when targeting full .NET (and aspnet).


## SqlClient
The purpose of this component is to provide an implementation of SqlClient that can be used for the EF7 SQL Server provider. 

The EF team has a temporary implementation of this component which is maintained in an internal repository. Builds of this code base are being published to [AspNetvNext MyGet feed](https://www.myget.org/F/aspnetvnext/api/v2/) (as the System.Data.SqlClient NuGet package). This is throw away code that was used to provide SQL Server support in alphas of EF7. The implementation and the NuGet package have a number of known issues (i.e. don’t use the existing NuGet package as a template for what the real package should look like).

Key requirements of this component include the following.

#### Implementation

This spec does not contain an exhaustive list of all the SqlClient features that EF7 requires. The implementation should implement the APIs available in the System.Data.Common provider model, plus any additional functionality that the SQL team thinks is worth exposing in modern applications. 

#### Packaging
It is delivered as a NuGet package

It is licensed with the standard [Microsoft .NET Library EULA](http://www.microsoft.com/web/webpi/eula/net_library_eula_enu.htm). This license imposes minimal restrictions, allowing use on non-Windows platforms. 

The package includes any required native assemblies and does any setup required for those assemblies to be used by the application (i.e. the developer shouldn’t need to do anything other than install the NuGet package for this component to be useable).

#### Platforms
At a minimum, the package should support the net451, aspnet50, and aspnetcore50 platforms.

**Note:** _Supporting net451 implicitly provides support for aspnet50._

If we decide to require a later version of .NET for System.Data.Common then this package will also require that version.

The net451 implementation should type forward to the existing type in the .NET Framework. This avoids there being two duplicate types for folks targeting full .NET. This avoids compilation issues, and also allows types like ```SqlConnection``` from existing app code to be used with EF7.

While these platforms are required for EF7, we would strongly support SqlClient also targeting the win81 and wpa81 platforms.