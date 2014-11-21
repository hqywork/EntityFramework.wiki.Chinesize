# EF NuGet Packages

In EF6 and earlier versions, installing the EntityFramework NuGet package installed everything needed to use EF with SQL Server, including Migrations functionality and PowerShell commands.

In EF7 the package structure is more granular making it possible to install just the parts of EF you need. Currently EF7 has the following packages:
* EntityFramework
* EntityFramework.Commands
* EntityFramework.Migrations
* EntityFramework.Relational
* EntityFramework.AzureTableStorage
* EntityFramework.InMemory
* EntityFramework.Redis
* EntityFramework.SQLite
* EntityFramework.SqlServer

The EntityFramework package is now just core functionality that cannot be used without installing at least one of the provider packages. The Commands package is also needed in order to use the Migrations PowerShell or K commands.

We had a tentative plan to rename the EF7 EntityFramework package to EntityFramework.Core and to then create an EntityFramework meta-package [(Issue #1049)](https://github.com/aspnet/EntityFramework/issues/1049) that installed everything needed to use EF with SQL Server, thus preserving the experience from EF6.

However, as we support more platforms and more stores it becomes less compelling for the default experience to automatically install the SQL Server provider. Some examples:
* Any application using a NoSQL provider, in which case the Migrations and Relational packages are also not needed
* Phone/store apps where SQLite is being used
* Mac/Linux platforms where using PostgreSQL is likely to be common

Therefore we will not have the EntityFramework package install the SQL Server provider. Instead it will throw up a nicely formatted HTML page indicating that a provider of choice should now be installed.

However, we will still create a meta-package called EntityFramework which will install the Core and Commands packages. Currently the commands package will bring in Migrations and Relational packages. However, we will make these soft dependencies once we have a build/packaging system that is more conducive to such things.

