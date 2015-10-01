# Table selection in reverse engineering

Right now we have the syntax 

`-t|--tables <filters>`

where `<filters>` looks like `*:table,schema:*,schema1:table1...`

Decided to move to `-t|--table` and `-s|--schema` where both `-t` and `-s` are allowed multiple times. `-s` will represent what was previously `schema:*`, `-t` will be some combination of `schema_name` and `table_name` uniquely identifying a specific table. The list will be just sent to each provider as is and it will be up to the provider to interpret the syntax.

So e.g. for SqlServer it will be:

`-s schema1 -s schema.with.dots ...`

`-t schema1.table1 -t schema2.table2 -t [schema.with.dots].[table.with.dots] ...`

For SQLite there is no concept of schema so it will be:

`-t table1 -t table2 ...`

and any `-s` entries will be ignored.

In PowerShell, we'll have `-Tables` and `-Schemas` which take an array of strings. For example:

`-Schemas schema1 -Tables ('table1', 'table2')`

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
