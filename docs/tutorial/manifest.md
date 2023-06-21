# Configure the Manifest File

Go to the `squirrels.yaml` file. This is the manifest file to configure most of the properties of the Squirrels project.

In this tutorial, we will focus on the `project_variables`, `db_connections`, and `datasets` sections. We will also assume that all comments in the [yaml] file (every '#' to end of line) are removed.

## Setting the Project Variables

The project variables `product`, `major_version`, and `minor_version` are required. In additional to those, you are free to add any of your own project variable.

In this tutorial, we will be making dataset APIs related to historical weather data in Seattle. Change the `product` to `seattle_weather`. We can leave `major_version` and `minor_version` unchanged.

The `project_variables` section should now look like this:

```yaml
project_variables:
  product: seattle_weather
  major_version: 1
  minor_version: 0
```

## Setting the Database Connections

This section is where we set all the database connection details that we need. We provide a list of connection names here and refer to them in other files. The connection name `default` must be provided for database views (and `DataSourceParameter`, which will be mentioned later) that don't specify a connection name explicitly.

Under `default`, change the url from `sqlite://${username}:${password}@/./database/sample_database.db` to `sqlite:///./database/seattle_weather.db`.

The syntax for the URL uses [sqlalchemy database URLs](https://docs.sqlalchemy.org/en/20/core/engines.html#database-urls). Since sqlite databases don't require a username and password, the `credential_key` field can be omitted. More details on setting and using credential keys can be found on the pages for "[credential management]" command line reference and how to "[Configure Database Connections]".

The `db_connections` section should now look like this:

```yaml
db_connections:
  default:
    url: 'sqlite:///./database/seattle_weather.db'
```

!!! Note 
    The `db_connections` section is optional if you choose to set the connection details in Python with a `connections.py` file or if your squirrels project doesn't require any database connections. More details can be found on the "[Configure Database Connections]" page.

## Defining the Datasets

This section is where we define the attributes for all datasets created by the squirrels project. Every dataset defined will have their own "parameters API" and "dataset API".

Currently, there is one dataset with the name `sample_dataset` (and label `Sample Dataset`). Change the name to `weather_by_time`, and change the label to `Weather by Time of Year`.

Every dataset defined would have 0 or more `database_views` and exactly 1 `final_view`. Each of the `database_views`/`final_view` can either be 

1. A string for the file name of the SQL file (templated with [Jinja]) or Python file to run, or 
2. An object with the `file` key and additional optional keys such as `db_connection` and `args`. 

In addition to being a templated SQL or Python file, the `final_view` can also be the name of one of the database views. 

!!! Note
    At runtime, all the database views for the dataset are run in parallel. If the final view is a SQL file, all the database view results are loaded into an in-memory sqlite database where the table names are the same as the database view names. If the final view is a Python file, the database view results are accessible as a dictionary of database view names to pandas Dataframes. More details on writing final views will come later in the tutorial.

Change the name `database_view1` to `aggregate_weather_metrics`, and replace its value from an object to the string `aggr_weather_metrics.sql.j2`. Change the value of `final_view` to `final_view.sql.j2`.

You may also remove the optional `args` key to the `weather_by_time` dataset. The `datasets` section should now look like this:

```yaml
datasets:
  weather_by_time:
    label: Weather by Time of Year
    database_views:
      aggregate_weather_metrics: aggr_weather_metrics.sql.j2
    final_view: final_view.sql.j2
```

Every dataset that's set in the `datasets` section must also have a matching folder name in `datasets` folder. In the `datasets` folder, rename `sample_dataset` to `weather_by_time`.

More details on the manifest file can be found in the [Manifest File] topic guide.

[credential management]: ../cli/credentials.md
[Configure Database Connections]: ../how-to/database.md
[Manifest File]: ../topics/manifest.md

[yaml]: https://yaml.org/
[Jinja]: https://jinja.palletsprojects.com/
