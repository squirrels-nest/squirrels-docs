# Configure the Manifest File

Go to the `squirrels.yaml` file. This is the manifest file to configure most of the properties of the Squirrels project.

In this tutorial, we will focus on the `project_variables`, `db_connections`, and `datasets` sections.

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

This section is where we set all the database connection details that we need. We provide a list of connection names here and refer to them in other files. The connection name `default` must be provided for components with sql queries that don't specify a connection name explicitly.

Under `default`, change the url from `sqlite://${username}:${password}@/./database/sample_database.db` to `sqlite:///./database/seattle_weather.db`.

The syntax for the URL uses [sqlalchemy database URLs](https://docs.sqlalchemy.org/en/20/core/engines.html#database-urls). Since sqlite databases don't require a username and password, the `credential_key` field can be either set to null or omitted entirely. More details on setting and using credential keys can be found in the pages for "[credential management]" command line reference and how to "[Configure Database Connections]".

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

Every dataset defined would have 0 or more `database_views` and exactly 1 `final_view`. Each of the `database_views` and `final_view` can either be: 

1. A string for the file name of the SQL file (templated with [Jinja]) or Python file to run. Or...
2. An object with the `file` key (for the same file name in item 1 above) and additional optional keys such as `db_connection` and `args`. 

In addition to being a templated SQL or Python file, the `final_view` can also be a string for the name of one of the database views. For database views, if `db_connection` is not specified (including if the value is string for file name instead of object), then "default" is used for `db_connection`.

Change the name `database_view1` to `aggregate_weather_metrics`, and replace its value from an object to the string `aggr_weather_metrics.sql.j2`. Change the value of `final_view` to `final_view.sql.j2`.

```yaml
datasets:
  weather_by_time:
    label: Weather by Time of Year
    database_views:
      aggregate_weather_metrics: aggr_weather_metrics.sql.j2
    final_view: final_view.sql.j2
```

Every dataset that's set in the `datasets` section must also have a matching folder name in `datasets` folder. In the `datasets` folder, rename `sample_dataset` to `weather_by_time`. Inside the folder, rename `database_view1.sql.j2` to `aggr_weather_metrics.sql.j2`.

More details on the manifest file can be found in the [Manifest File] topic guide. See the next tutorial page to [Create the Dataset Parameters](parameters.md).

[credential management]: ../cli/credentials.md
[Configure Database Connections]: ../how-to/database.md
[Manifest File]: ../topics/manifest.md

[yaml]: https://yaml.org/
[Jinja]: https://jinja.palletsprojects.com/
