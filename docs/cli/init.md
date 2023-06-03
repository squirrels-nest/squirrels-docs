# The init CLI

The `squirrels init` command is the initializer command for the squirrels framework, it is used to set up a new project or to reinitialize an existing one. When used, it will populate the current directory with a barebones version of the squirrels framework, depending on the user's preference. By default, the files created will not overwrite files that already exist. This behaviour can be changed by using the `--overwrite` option.

After executing the command, the user would be able to specify which files should be initialized by answering the following prompts:

```bash
[?] Include all core project files? (Y/n): 

[?] What's the file format for the database view?:
 > sql
   py

[?] Do you want to add a 'connections.py' file? (y/N): 

[?] Do you want to add a 'context.py' file? (y/N): 

[?] Do you want to add a 'selections.cfg' file? (y/N): 

[?] What's the file format for the final view (if any)?:
 > none
   sql
   py

[?] What sample sqlite database do you wish to use (if any)?:
 > none
   sample_database
   seattle_weather

```

The first question answers whether all project files should be generated. If `Y` or `y` is specified, it will then ask the user whether the `database_view1` file should be either python or sql. It will then create all the core files, which include:

- A sample `parameters.py` file in the `datasets/sample_dataset` folder
- A sample `database_view1.sql.j2` or `database_view1.py` file in the `datasets/sample_dataset` folder
- A `.gitignore` file
- A `requirements.txt` file
- A sample `squirrels.yaml` file

Additional prompts are also provided to inquire whether the following files are needed.

|File|Description|
|:---|:----------|
|`connections.py`|Used to specify keys to SQL database connections by providing a dictionary of <br>strings to sqlalchemy Engines or Pools. This can be used in addition to, or in place <br>of, the `db_connections` section of `squirrels.yaml`|
|`context.py`|At runtime, after parameter selections are applied, this file can be used to transform <br>selected parameter options into any python variable|
|`selections.cfg`|Can be used with the "squirrels test" command to apply parameter selections to test <br>the rendered SQL queries|
|`final_view.sql.j2` or `final_view.py`|A "final view" file can be specified (as sql or python) to join all the database views <br>into a single dataframe|

In the last question, the user can choose to include a sample sqlite database. Current options include the `sample_database` and `seattle_weather` databases.

Further usage details can be found by running `squirrels init -h` which prints the text below. The user can also choose which files to add by providing additional command line options, avoiding the prompts above. This applies to all command line options below except for `-h` and `--overwrite`.

```bash
usage: squirrels init [-h] [--overwrite] [--core] [--db-view {sql,py}] [--connections] [--context] [--selections-cfg] [--final-view {sql,py}] [--sample-db {sample_database,seattle_weather}]

options:
  -h, --help            show this help message and exit
  --overwrite           Overwrite files that already exist
  --core                Include all core files
  --db-view {sql,py}    Create database view as sql (default) or python file if "--core" is specified
  --connections         Include the connections.py file
  --context             Include the context.py file
  --selections-cfg      Include the selections.cfg file
  --final-view {sql,py}
                        Include final view as sql or python file
  --sample-db {sample_database,seattle_weather}
                        Sample sqlite database to include
```
