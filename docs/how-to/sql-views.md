# Create Views with SQL and Jinja

**Note**: This is the SQL version of how to create views, for the Python template version see: [Create Views with Python](python-views.md)

There are two types of views that are relavent to a squirrels project: database views, and final views. The "database view" refers to the view that's directly selected from the database using a connection as described here [Creating a Database Connection](../how-to/database.md), while on the other hand, the final view query joins the database views to generate the final dataset that is sent to the front end.

## Creating database views 

Regardless whether you use Python or SQL for your database view, you'll need create the parameters in the parameters.py file. This is delineated here: [Creating Parameters](writing-parameters.md). 

To create a database view, here are a few simple steps to follow:

1. Create sample file by using `squirrels init`.
2. Copy over the file from the `sample_dataset` folder to your desired dataset folder, and rename the file accordingly. 
3. Update the file name in `squirrels.yaml` to the name of the file you've just copied over.
4. Populate the file with the query template for your dataset.

### Creating an empty file and copying (Step 1 - 2)

Run the `squirrels init --core --db-view sql` command to create a `database_view1.sql.j2` file in the `sample_dataset` folder, or run `squirrels init --final-view sql` to create a `final_view.sql.j2` file. These files can then serve as templates for the subsequent SQL database views that you'll need to create for your project. After you've created and located the file(s), you can then copy it over to your desired dataset folder, and rename your file(s) accordingly.

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

After you've copied over the file, feel free to delete them and make your own view from scratch.

The database view queries the database, and the final view is joins all the results of the database views together in a SQLite query. 

Both of these views are Jinja templates for SQL, and the standard Jinja syntax applies. For database views, the specific SQL syntax is determined by the database as specified in either `connection.py` or `squirrels.yaml`. See [Configure Database Connections](/how-to/database.md) for more details regarding database connections and how to create it. 

Both the final and database views have access to the following keywords:

1. prms
2. ctx
3. args

The first `prms` variable contains all the parameters as specified in the `parameters.py` file in the form of a dictionary, where the key is the name of the parameter (the first argument passed into the parameter's constructor). Likewise, the `ctx` variable is a dictionary as well consisting of parameter options returned by `context.py`. The `args` is also a similar dictionary, but contains the project information in the `squirrels.yaml` file, see [Manifest File (squirrels.yaml)](../topics/manifest.md) for more information on "project variables" and "args". 

A parameter can be retrieved in the template like a standard Python map within a double curly bracket `{{}}` by specifying the key like `prms['single_select_param']`, and calling a method for the selected parameter, such as `prms['single_select_param'].get_selected_label_quoted()`.

In the future, we also plan to add an optional argument analogus to the sql `USE` statement in the sql template in order to have a way to specify which database is to be used dynamically. Currently, it can only be specified in the `squirrels.yaml` file as a fixed value using `db_connection`.
