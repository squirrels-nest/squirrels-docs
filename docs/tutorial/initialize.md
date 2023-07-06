# Initialize a Squirrels Project

You can initialize the project files using:

```bash
squirrels init
```

Prompts will appear for the various files you wish to include in your project. Answer the prompts as follows:

```config
[?] Include all core project files? (Y/n): y

[?] What's the file format for the database view?: sql
 > sql
   py

[?] Do you want to add a 'connections.py' file? (y/N): n

[?] Do you want to add a 'context.py' file? (y/N): n

[?] Do you want to add a 'selections.cfg' file? (y/N): n

[?] What's the file format for the final view (if any)?: none
 > none
   sql
   py

[?] What sample sqlite database do you wish to use (if any)?: sample_database
   none
 > sample_database
   seattle_weather
```

Once the command is executed, the following files are created.

- `.gitignore`, `requirements.txt`, and `squirrels.yaml` at the project root
- `parameters.py` and `database_view.sql.j2` in the `datasets/sample_dataset` subfolder
- `sample_database.db` sqlite database in the `database` folder

If we exclude the sqlite database and files that are common for all Python git projects, then `squirrels.yaml`, `parameters.py`, and `database_view.sql.j2` are essentially all you need to have a working squirrels project!

Go ahead and take a quick glance at the new files (no need to fully understand them now). Then, run the API server using the command:

```bash
squirrels run
```

In a web browser, go to `http://localhost:8000/` or `http://127.0.0.1:8000/`. This leads you to the Squirrels UI, a convenient interface for testing the dataset APIs. Without changing the widget parameters, click the "Apply" button to display the dataset for the default parameter selections (take some time now to generate various datasets using different parameter selections).

To see the API endpoints that provides the information on the parameters and tabular results on the "Sample Dataset", you can use the following URLs to access the JSON results for the default parameter selections:

1. Parameters API: `http://localhost:8000/squirrels0/sample/v1/sample-dataset/parameters`
2. Dataset API: `http://localhost:8000/squirrels0/sample/v1/sample-dataset`

After you're done with the API server, you can terminate it in the terminal by pressing "Ctrl+C".

Now, we will use the `init` command again to add more files that'll come in handy for the rest of the tutorial. Run:

```bash
squirrels init --context --selections-cfg --final-view sql --sample-db seattle_weather
```

The following files have been added.

- `context.py`, `final_view.sql.j2`, and `selections.cfg` in the `datasets/sample_dataset` subfolder
- `seattle_weather.db` sqlite database in the `database` folder

Note that when specifying arguments to the `squirrels init` command, prompts no longer show up. You can see `squirrels init --help` for more details on what various command line arguments do. Currently, all arguments except "--help" and "--overwrite" disables the prompts.

See the next tutorial page to [Configure the Manifest File](manifest.md).
