## Naming Guidelines
Packages names (and therefore assembly names) all begin with EntityFramework[major_version_number] (i.e. EntityFramework7.Core, EntityFramework7.Commands). 

Providers use EntityFramework[major_version_number].[provider_name] (i.e. EntityFramework7.SqlServer, EntityFramework7.npgsql). 

## Why Do This

### Package updates don't cross major version boundaries
This approach means updating a package will give you the latest release within the same major version. This means your code should still compile and run. 
* Moving to a new major version will be an explicit opt-in operation and folks will be consciously aware that they are moving to a new version. We saw folks frustrated that updating EF5 packages gave them EF6 which included breaking changes (and needed a new version of providers). We would definitely see the same (to a much larger extent) if EF7 was just an update to the EF6 packages.
* Major version changes usually include breaking changes. Updating across major versions will usually result in needing code changes and usually a new version of the provider.
* Providers usually only work for a given major version. Therefore if you change major versions you typically need to swap to the version of the provider that works with the EF version (historically this has been either a higher version of the same provider package or a different package altogether).

### Side-by-side major versions
This approach allows multiple major versions of EF to be used in the same app. We often see folks wanting to do this so that they can incrementally port an app over, or use the latest version for new data models. This would otherwise be blocked because you can't load two versions of an assembly with the same name.
* This is a scenario that doesn't apply to all technologies that ship via NuGet. For example, having two version of the ASP.NET MVC runtime makes no sense. But we are already seeing a lot of folks wanting to use EF6 and EF7 in the same app.

### Consistent provider naming
This approach provides a consistent way for providers to version and allows folks to easily work out which provider package/version to install.
* Historically some providers would version within the same package. This is difficult because you need to comb thru version history, looking at dependencies, to work out what version of the provider works with the EF version you are using. For example, if you want to use SqlServerCompact with EF5 you need to install version 4.3.6 of EntityFramework.SqlServerCompact. For EF6 you can install the latest (version 6.1.3).
* Other providers took the approach we are adopting for EF7, where they have a separate package for each major version of EF. This means you can safely update to the latest version of the package and be sure it will still work with the EF version you are using. For example, to use SQLite with EF6 you install System.Data.SQLite.EF6. 

## Downsides
There are a few disadvantages to this approach.
* The EntityFramework package has historically been in the top 3 packages in NuGet.org. We will lose this because there will no longer be a single package that represents all versions of EF. On the up side, the most commonly accepted version of EF will show up higher in the list (i.e. EF7 will only show up above EF6 once it becomes more popular).
* Our history doesn't follow this approach (i.e. there are no EntityFramework5.*, EntityFramework6.* packages). If we think there is value in having these then we can create these as meta packages and just have them pull in the appropriate version of the multi-major-version packages.
