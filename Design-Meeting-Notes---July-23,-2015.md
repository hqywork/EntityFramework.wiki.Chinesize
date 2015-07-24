# NuGet/DNX Commands

We reviewed the Entity Framework Commands and their parameters. Here is the outcome.

NuGet | DNX
----- | ---
Add-Migration<br />&nbsp;&nbsp;&nbsp;&nbsp;[Name]<br />&nbsp;&nbsp;&nbsp;&nbsp;-Context<br />&nbsp;&nbsp;&nbsp;&nbsp;-Project<br />&nbsp;&nbsp;&nbsp;&nbsp;-StartupProject | dnx [project] ef migration**s** add<br />&nbsp;&nbsp;&nbsp;&nbsp;[name]<br />&nbsp;&nbsp;&nbsp;&nbsp;-c, --context<br />&nbsp;&nbsp;&nbsp;&nbsp;-s, --startupProject
Remove-Migration<br />&nbsp;&nbsp;&nbsp;&nbsp;-Context<br />&nbsp;&nbsp;&nbsp;&nbsp;-Project<br />&nbsp;&nbsp;&nbsp;&nbsp;-StartupProject | dnx [project] ef migration**s** remove<br />&nbsp;&nbsp;&nbsp;&nbsp;-c, --context<br />&nbsp;&nbsp;&nbsp;&nbsp;-s, --startupProject
Script-Migration<br />&nbsp;&nbsp;&nbsp;&nbsp;~~[From]~~ **-From** *(required when -To)*<br />&nbsp;&nbsp;&nbsp;&nbsp;~~[To]~~ **-To**<br />&nbsp;&nbsp;&nbsp;&nbsp;-Idempotent<br />&nbsp;&nbsp;&nbsp;&nbsp;-Context<br />&nbsp;&nbsp;&nbsp;&nbsp;-Project<br />&nbsp;&nbsp;&nbsp;&nbsp;-StartupProject | dnx [project] ef migration**s** script<br />&nbsp;&nbsp;&nbsp;&nbsp;~~[from]~~ **-f, --from**<br />&nbsp;&nbsp;&nbsp;&nbsp;~~[to]~~ **-t, --to**<br />&nbsp;&nbsp;&nbsp;&nbsp;-i, --idempotent<br />&nbsp;&nbsp;&nbsp;&nbsp;-c, --context<br />&nbsp;&nbsp;&nbsp;&nbsp;-s, --startupProject
~~Apply-Migration~~ **Update-Database**<br />&nbsp;&nbsp;&nbsp;&nbsp;[Migration]<br />&nbsp;&nbsp;&nbsp;&nbsp;-Context<br />&nbsp;&nbsp;&nbsp;&nbsp;-Project<br />&nbsp;&nbsp;&nbsp;&nbsp;-StartupProject | dnx [project] ef ~~migration apply~~ **database update**<br />&nbsp;&nbsp;&nbsp;&nbsp;[migration]<br />&nbsp;&nbsp;&nbsp;&nbsp;-c, --context<br />&nbsp;&nbsp;&nbsp;&nbsp;-s, --startupProject
~~Reverse-Engineer~~ **Scaffold-DbContext**<br />&nbsp;&nbsp;&nbsp;&nbsp;[Connection~~String~~]<br />&nbsp;&nbsp;&nbsp;&nbsp;[Provider]<br />&nbsp;&nbsp;&nbsp;&nbsp;-Project<br />&nbsp;&nbsp;&nbsp;&nbsp;~~-StartupProject~~ | dnx [project] ef ~~revEng~~ **dbcontext scaffold**<br />&nbsp;&nbsp;&nbsp;&nbsp;[connection~~String~~]<br />&nbsp;&nbsp;&nbsp;&nbsp;[provider]
~~Customize-ReverseEngineer~~ **Scaffold-DbContextTemplate**<br />&nbsp;&nbsp;&nbsp;&nbsp;[Provider]<br />&nbsp;&nbsp;&nbsp;&nbsp;-Project | dnx [project] ef ~~revEng customize~~ **dbcontext scaffold-templates**<br />&nbsp;&nbsp;&nbsp;&nbsp;[provider]
Use-DbContext<br />&nbsp;&nbsp;&nbsp;&nbsp;[Context]<br />&nbsp;&nbsp;&nbsp;&nbsp;-Project | *(no session)*
*(tab expansion)* | dnx [project] ef **db**context list<br />&nbsp;&nbsp;&nbsp;&nbsp;-s, --startupProject
*(tab expansion)* | dnx [project] ef migration**s** list<br />&nbsp;&nbsp;&nbsp;&nbsp;-s, --startupProject

Here are some other, general notes we had.
* Cross-reference related commands in help
* For renamed commands, ensure errors are useful to early adopters

It was also interesting to view the DNX sub-commands as a hierarchy.

* migrations
    * list
    * add
    * remove
    * script
* dbcontext
    * list
    * scaffold
    * scaffold-templates
* database
    * update

# Cross-platform coding guidelines
We reviewed and discussed how to make the EF code more sensitive to Linux and OS X. The issues discussed have now been addressed in [Engineering Guidelines - Cross-platform coding](https://github.com/aspnet/Home/wiki/Engineering-guidelines#cross-platform-coding).

# Discussion

Please use the [discussion issue](https://github.com/aspnet/EntityFramework/issues/2706) to provide feedback, ask questions, etc.