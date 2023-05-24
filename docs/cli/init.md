# Squirrels init
The `squirrels init` command is the initializer command for the squirrels framework, it is used to set up a new project or to reinitialize an existing one. When used, it will populate the current directory with a barebones version of the squirrels framework, depending on the user's preference. (Make sure with Tim whether this deletes the files in the current directory)

After executing the command, the user would be able to spcify the what and which files should be initialized by answering the following questions:

```bash
[?] Include all core project files? (Y/n): 
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

The first question answers whether all project files should be generated, if `Y` is specified, it will ask the user which file format the database view would be based off, where the user can specify either python or sql. It will then create all the core files, including a dataset folder which will house a sample database_view1.sql.j2 file, a sample final view file, and the parameters.py file. It will also create the requirements.txt and squirrels.yaml files. If `n` is specified, then none of the above mentioned files and folders will be created. Squirrels will then ask individually, whether the optional files `connections.py`, `context.py`, and `selections.cfg` should be generated, regardless of the core files exist. 

The second to last question allows the user to specify what file format is to be used for the final view. Currently, we're only allowing for `sql` or python, and none if the user whishes to add a final view file himself later on. 

In the last question, the user can chose to which sample database to add, if at all. Current options include the `sample_database` and `seattle_weather` sqlite databases. 
