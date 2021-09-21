SQLite quickref
================
Chris Johnson

-   [Setting up SQLite on Windows](#setting-up-sqlite-on-windows)
-   [Dot commands overview](#dot-commands-overview)
-   [Working with databases](#working-with-databases)
-   [Describing things](#describing-things)
-   [Working with data](#working-with-data)
-   [Running SQL scripts](#running-sql-scripts)
-   [Running SQL interactively](#running-sql-interactively)
-   [Enforcing relationships between
    tables](#enforcing-relationships-between-tables)
-   [Gotchas](#gotchas)
-   [Backing up, restoring, and
    related](#backing-up-restoring-and-related)
-   [References](#references)

<!-- Planned improvements:

The following commands allow for "|do-a-thing":

* `.output`
* `.once`
* `.import`

The following need their own section:

* `.dbconfig` 
* `.dbinfo`

SQLite configuration section:

* `.limit` allows modificaton of `SQLITE_LIMIT`
* `.bail` stop if error, on or off
* `.changes` print rows changed, on or off
* `.log` log, on or off
* `.nullvalue` value to use for null value
* `.trace` debugging

-->

## Setting up SQLite on Windows

Getting started with SQLite on Windows is very easy. First, download the
[precompiled SQLite binaries for
Windows](https://www.sqlite.org/download.html). Extract the contents of
the zip anywhere, and double-click `sqlite3.exe` to launch the command
line shell program.

## Dot commands overview

`sqlite3` provides *dot commands*—commands prefixed with a dot
(e.g. `.open`) to perform some action. Some dot commands change the
output format, while others seem to be wrappers (around SQL statements,
other dot commands, or both). Unlike SQL statements, dot commands don’t
support comments, nor can they span more than one line.

Arguably the most important dot command for the beginner is `.help`,
which lists all dot commands and a description of what they do. Dot
commands are very intuitive, but I’ll cover some fundamental ones here.

## Working with databases

`sqlite3.exe` can run from anywhere, so wherever it lives on the machine
dictates the current directory by default. To show this is true, submit
`.shell cd`. (`.shell` gives access to `cmd.exe`.) To navigate
elsewhere, use `.cd path/to/dir`. (Note: `.shell cd path/to/dir` has no
effect; must use `.cd`.)

The point of navigating is to get to a location where a SQLite database
exists or where you would like one to exist. (SQLite databases have the
extension `db`.) Once there, `.open` is the command to either create a
new database or open an existing one. For example, suppose we have a
database in the current directory named `toy.db`. Then `.open toy.db`
will connect you to that database. If `toy.db` *didn’t* exist in the
current directory, then `.open toy.db` would *create it*.

If a connection has not been made, one can do work in a temporary
database. This is useful for test driving `sqlite3`, but not advised. If
you happen to want to save work done using the temporary database, use
`.save`. Note: `.save` will overwrite without warning!

## Describing things

`.databases` lists all attached databases. `.tables` queries the
`sqlite_schema` table for all attached databases, and neatly returns all
tables. `.indexes` without an argument returns the indexes for all
tables from attached databases. `.indexes that_table` will return the
indexes for just that table.

`.schema` alone will return the schema for the all attached databases.
To see the schema for a specific database, use `.schema name_of_db.*`.
To see the schema for a specific table, use `.schema name_of_table`. Use
`--indent` to prettify the resulting `create` statements.

## Working with data

`.mode` is used to change between formats. The available formats can be
listed with `.help mode`. The default mode is `list`, and its default
delimiter is `|`. The delimiter can be changed with `.separator`.
Formats can be useful for both printing to the terminal or writing data
to a file (such as CSV).

The modes `column`, `box`, `table`, and `markdown` print data in
columns, and the width of those columns is automatically determined
based on the data. The `.width` command allows the user to specify a
vector of fixed column widths as integers which can improve the
readability of a table when printed to the terminal. Positive integers
will left-justify columns while a negative integers will right-justify
them. A value of `0` indicates the column width should be automatically
determined. To wipe column width definitions and resume automatically
determining column widths, submit `.width` without any values.

The mode `csv` is self-explanatory. `.headers` can be turned `on` or
`off`. To re-route output until otherwise requested, use
`.output name_of_file`. To re-route only the next statement, use
`.once name_of_file`. This info, collectively, can be used to produce a
CSV. To view the export, use `.system path/to/name_of_file.csv`.

`.excel` creates a CSV in a temporary directory, reroutes the output of
the next statement or command to it, and automatically launches the
default application to view it (often Excel).

CSV files to be *imported* into a table with `.import`, assuming the
mode is set to `csv`. If the table doesn’t exist in the database, it
will create it, using the first row as the header. If the table *does*
exist, the data in the CSV will be appended to the it. If the CSV
includes a one-row header, append `--skip 1` to the `.import` statement.

## Running SQL scripts

There are a few ways to execute SQL stored in a script. From outside the
shell,

    sqlite3 toy.db < toy.sql

An alternative suggestion for outside the shell is

    sqlite3 toy.db ".read toy.sql"

From inside the shell

    .read toy.sql 

Dot commands can be included in the script.

## Running SQL interactively

All statements must be terminated with a semicolon. Statements can span
multiple lines. To do this, press Enter. The cursor will appear on the
next line with a continuation prompt (`...>`).

Ctrl + C produces the interrupt character, which will interrupt a
running SQL command. Ctrl + D produces the end of file (eof) character,
which will terminate `sqlite3`. (One can also use `.exit` to do the
same.)

## Enforcing relationships between tables

Foreign key constraints are supported, but they’re not enabled by
default. To enable foreign key constraints, use
`pragma foreign_keys = on;`. To check the state, submit
`pragma foreign_keys;`. Though foreign key constraints can be enabled,
this setting only persists during a connection.

The boilerplate foreign key constraint is

    foreign key (foreign_key_columns)
      references parent_table (parent_key_columns)
        on update action
        on delete action;

When a primary key is deleted from a table, a foreign key constraint
action occurs. The actions are `set null`, `set default`, `restrict`,
`no action`, and `cascade`.

See SQLite Tutorial’s [tutorial on foreign key
constraints](https://www.sqlitetutorial.net/sqlite-foreign-key/) for a
demo.

## Gotchas

SQLite uses dynamic typing, i.e. does not enforce data type constraints.
In other words, the data type specified in a `create table` statement
doesn’t restrict one from inserting an entry of a different data type.
The developers call this “feature” [*type
affinity*](https://www.sqlite.org/faq.html#q3). The workaround is to use
`check` constraints in the `create table` statement:

    create table customer (
      id primary key,
      first_name text check(typeof(first_name) = 'text'),
      last_name text check(typeof(last_name) = 'text'),
      is_rewards_member = integer check(is_aggressive in (0, 1))  
    );

## Backing up, restoring, and related

`.backup` allows the database to be backed up to a file, while
`.restore` does the reverse: restores the database from the file.

To generate `insert` statements for a given, populated table, use
`.mode insert name_of_table`, followed by a `select` statement. The
entire database can be rendered to SQL with `.dump`. Finally, an
existing database can be cloned into a new database with `.clone`.

## References

<div id="refs" class="references csl-bib-body hanging-indent">

<div id="ref-c" class="csl-entry">

“SQLite.” n.d. <https://sqlite.org/cli.html>.

</div>

</div>
