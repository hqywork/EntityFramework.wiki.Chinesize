# Table selection in reverse engineering

# Command options naming

We also reviewed a few options that have been added to the commands since [the last review](https://github.com/aspnet/EntityFramework/wiki/Design-Meeting-Notes---July-23,-2015#nugetdnx-commands). Here are the notes.

NuGet | DNX
----- | ---
--OutputDir~~ectory~~ | -o, --output~~-path~~Dir
-Context~~ClassName~~ | -c, --context~~-class-name~~
-Tables | -t, --table~~s~~ *(multi-value option)*
~~-FluentApi~~ **-DataAnnotations** | ~~-u, --fluent-api~~ **-a, --dataAnnotations**
**-Schemas** | **-s, --schema** *(multi-value option)*
-StartupProject | -~~s~~**p**, --startupProject

# Discussion

Please use the [discussion issue](https://github.com/aspnet/EntityFramework/issues/3297) to provide feedback, ask questions, etc.
