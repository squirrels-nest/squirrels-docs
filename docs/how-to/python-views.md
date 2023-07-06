# Create Views with Python

**Note**: This is the Python version of how to create views, for the SQL template version see: [Create Views with SQL and Jinja](sql-views.md)

There are two types of views that are relavent to a squirrels project: database views, and final views. The "database view" refers to the view that's directly selected from the database using a connection as described here [Creating a Database Connection](../how-to/database.md), while on the other hand, the final view query joins the database views to generate the final dataset that is sent to the front end.

## Creating a database view

Regardless whether you use Python or SQL for your database view, you'll need create the parameters in the parameters.py file. This is delineated here: [Creating Parameters](writing-parameters.md). 

After that's done, these are the following steps to create views with python:

1. Create sample file by running the following command in the terminal: `squirrels init --core --db-view py` for creating a database view,
`squirrels init --final-view py` for creating a final view.
2. Copy over the file from the `sample_dataset` folder to your desired dataset folder, and rename the file accordingly. 
3. Update the file name in `squirrels.yaml` to the name of the file you've just coppied over.
4. Populate the file with the query template for your dataset.

### Creating an empty file and copying (Step 1 - 2)

Run the `squirrels init --core --db-view py` command to create a `database_view1.py` file in the `sample_dataset` folder, or run `squirrels init --final-view py` to create a `final_view.py` file. These files can then serve as templates for the subsequent SQL database views that you'll need to create for your project. After you've created and located the file(s), you can then copy it over to your desired dataset folder, and rename your file(s) accordingly.

### Adding the view to squirrels.yaml (Step 3)

After a view has been created, regardless of whether it's using SQL or Python, the views will need to be added to the manifest files in order for the project to be able to see the files. Under `datasets`, add a name for the dataset folder the views are housed in, and provide the label for the dataset. Under `database_views`, provide each database views' file name, name of the connection under `db_connection`, and any additional arguments. Make sure to add the file suffixes.

The datasets field should have at least this much:

```yaml
datasets:
  <your_dataset>:
    label: <your_dataset label>
    database_views:
      database_view1: 
        file: <new_database_view_name>.sql.j2
        db_connection: default # optional if default
        args: {} # optional if empty
    final_view: <new_final_view_name>.sql.j2
    args: {} # optional if empty
```

### Populating the file (Step 4)

Inside the `main()` function for the database view, you'll be able to access the database connection by first using `connection_set.get_connection_pool("<connection_name>")` to get the connection pool/sqlalchemy engine (from which you can get a DBAPI/sqlalchemy connection to query databases), and create a pandas dataframes out of it to return as output. 

On the other hand, the final view holds whatever additional transformation that needs to happen after the queries for the database views have been returned. The results from the database views are stored in the `database_views` dictionary, and individual dataframes can be retrieved by referring to the name of the view. The final view is not strictly needed, but can be useful depending on the project.  

Both the final and database views have access to the following keywords:

1. prms
2. ctx
3. args

The first `prms` variable contains all the parameters as specificed and returned in the `parameters.py` file in the form of a dictionary, where the key is the name of the parameter (the first argument passed into the parameter's constructor). The values selected from the parameters can be retrieved by using a method that starts with "get_selected_". The available "get_selected_" method(s) depend on the type of the parameter (for instance, SingleSelectParameter has `get_selected_id`, `get_selected_label`, and more). 

The `ctx` variable is a dictionary as well, returned by `context.py`, to create any derived value from the parameter selections. The `args` is also a similar dictionary, but contains the project information in the `squirrels.yaml` file, see [Manifest File (squirrels.yaml)](../topics/manifest.md) for more information. 

A parameter can be retrieved in the template like a standard Python map within a double curly bracket `{{}}` by specifying the key like `prms['single_select_param']`, and calling a method for the selected parameter, such as `prms['single_select_param'].get_selected_label_quoted()`.
