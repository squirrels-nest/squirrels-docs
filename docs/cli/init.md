# Squirrels init
The `squirrels init` command is the initializer command for the squirrels framework, it is used to set up a new project or to reinitialize an existing one. When used, it will populate the current directory with a barebones version of the squirrels framework, depending on the user's preference. By default, the files created will overwrite the files in the directory, but other pre-existing files that don't share the same name as those selected to be made by the init command will not be touched. 

After executing the command, the user would be able to spcify the what and which files should be initialized by answering the following prompts:

```bash
[?] Include all core project files? (Y/n): 

[?] What's the file format for the database view?: sql
 > sql
   py

[?] Do you want to add a 'connections.py' file? (y/N): 
[?] Do you want to add a 'context.py' file? (y/N): 
[?] Do you want to add a 'selections.cfg' file? (y/N): 

[?] What's the file format for the final view (if any)?: none
 > none
   sql
   py

[?] What sample sqlite database do you wish to use (if any)?: none
 > none
   sample_database
   seattle_weather

```

The first question answers whether all project files should be generated, if `Y` is specified, it will then ask the user which file format the database view should be based off, where the user can specify either python or sql. It will then create all the core files, including a dataset folder which will house a sample `database_view1.sql.j2 file`, and the `parameters.py` file if sql is chosen, and `database_view1.py` otherwise. It will also create the `requirements.txt` and `squirrels.yaml` files. If `n` is specified, then none of the above mentioned files and folders will be created, and quirrels will then ask individually, whether the optional files `connections.py`, `context.py`, and `selections.cfg` should be generated, regardless whether the core files exist. 

The second to last question allows the user to specify what file format is to be used for the final view. Currently, we're only allowing for `sql` or python, and none if the user whishes to add a final view file themselves later on. 

In the last question, the user can chose to which sample database to add, if any at all. Current options include the `sample_database` and `seattle_weather` sqlite databases.

Instead of going through the prompts, the user can also specify which files to add by providing additional options to the init command:

```bash
usage: squirrels init [-h] [--no-overwrite] [--core] [--db-view {sql,py}] [--connections] [--context] [--selections-cfg] [--final-view {sql,py}] [--sample-db {sample_database,seattle_weather}]

options:
  -h, --help            show this help message and exit
  --no-overwrite        Don't overwrite files if they already exist
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