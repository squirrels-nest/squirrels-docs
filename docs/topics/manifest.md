# Manifest File (squirrels.yaml)

The manifest file contains the following sections that is described in detail below.

1. modules
2. project_variables
3. db_connections
4. datasets
5. settings

## modules

This section is used to specify the list of git repos (with tag) that the project uses as modules. This can be useful to share the same Python code or Jinja macros across projects. See the [load-modules CLI guide] for details.

## project_variables

This section contains project variables that can be referenced within the project. The project variables "product", "major_version", and "minor_version" are mandatory, and they must be of type string, integer, and integer respectively. Additional custom project variables can also be added. See example below:

```yaml
project_variables:
  product: sample
  major_version: 1
  minor_version: 0
  custom_var: example
```

The product and major_version are used in forming the appropriate REST API paths. In the future, a deployment service will use the product, major_version, and minor_version to update a catalog of products and versions available which can then be retrieved using the catalog API (more details of the catalog API are provided in the [Response Schema] front-end guide). Then, for the parameters or dataset paths, a request header called "x-minor-version" can be used to control the minor version used (or latest is used if omitted).

For information incrementing major version vs minor version best practices, see the [Major vs Minor Version] topic guide.

## db_connections

This section can be used to specify database connections by providing a map between keys and [SQLAlchemy Database URLs]. This section is optional if connections.py is provided, and both can be used at the same time (though connections.py takes precedence if there are common keys). And example of the section may look something like this:

```yaml
db_connections:
  default: 
    url: 'dialect+driver:////absolute/path/to/database1.db'
  custom_db_conn_key:
    url: 'dialect+driver:///./relative/path/to/database2.db'
```

Between this section and connections.py, the key "default" must be specified if any connection key is specified (it is possible to not specify any connection keys for certain use cases without database views in SQL). If a username and password is required in the URL, then they can be set using `squirrels set-credential <key>` with some credential key (see the [Credential Management CLI guide]). Then, you can specify the credential key under the connection key with the attribute `credential_key` and use the placeholders `${username}` and `${password}` in the URL. If `credential_key` is null or not set, the `${username}` and `${password}` placeholders will become empty strings. See example below for specifying a connection key with credentials.

```yaml
db_connections:
  default: 
    credential_key: some_key
    url: 'dialect+driver://${username}:${password}@//path/to/database1.db'
```

For details on using a connections.py file, see the [Configure Database Connections] how-to guide.

## datasets

In this section, the list of dataset names and dataset attributes can be defined. Under each dataset name, the attributes are the `label`, `database_views`, `final_view`, and `args`. Any dataset name specified here must also have the same folder name in the `datasets` folder.

The `label` is simply the display name for the dataset to appear as in a dropdown menu. This attribute is required.

The `database_views` specifies the list of database view names and database view attributes can be defined. This attribute can be optional if no database views are required. Under the database view name, the required attribute is `file`, and optional attributes are `db_connection` and `args`. Or for simplicity, the value mapped to the database view name can also be a string for the equivalent `file` attribute.
- The `file` must be a Jinja SQL or Python file relative to the dataset folder. 
- The `db_connection` is the database connection defined in the `db_connections` section or connections.py. If `db_connection` is not specified, it defaults to "default". 
- The `args` is for specifying arguments at the database view level, and can override arguments specified at the project level or dataset level. These arguments can be referenced by any database view or final view file.

The `final_view` represents the final query for the dataset. It can be a Jinja SQL or Python file relative to the dataset folder (which can reference the results of the database views), or it can be the name of a database view. This attribute is required.

The `args` attribute is for specifying arguments at the dataset level, and can override arguments specified at the project level (i.e., the project variables). This attribute is optional.

Below shows an example of using the datasets section to construct a single sample dataset with 2 database views.

```yaml
datasets:
  sample_dataset:
    label: Sample Dataset
    database_views:
      database_view1: 
        file: db_view1.sql.j2
        db_connection: custom_db_conn_key
        args:
          arg1: db_view_val1
          arg2: db_view_val2
      database_view2: db_view2.py
    final_view: final_view.sql.j2
    args:
      arg0: dataset_val0
      arg1: dataset_val1
```

## settings

Certain settings can be specified in this setting. This section is optional if no settings need to be changed from their defaults. Below is an example of using this section.

```yaml
settings:
  parameters.cache.size: 1024
  parameters.cache.ttl: 86400
```

Here are the current list of setting keys available. Deafult is used when the key is not specified.

|Setting Key|Default|Description|
|:----------|:------|:----------|
|parameters.cache.size|1024|The maximum number of responses that the LRU cache of the parameters API can store|
|parameters.cache.ttl|86400|The time to live (in seconds) for the LRU cache of the parameters API. Default is 1 day|
|results.cache.size|128|The maximum number of responses that the LRU cache of the results API can store|
|results.cache.ttl|3600|The time to live (in seconds) for the LRU cache of the results API. Default is 1 hour|


[load-modules CLI guide]: ../cli/load-modules.md
[Response Schema]: ../front-end/response.md
[Major vs Minor Version]: versioning.md
[Configure Database Connections]: ../how-to/database.md
[SQLAlchemy Database URLs]: https://docs.sqlalchemy.org/en/latest/core/engines.html#database-urls
