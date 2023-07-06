# Configure Database Connections

All database connections are currently handled by sqlalchemy in the current version, with future expansion into no-sql databases on the horizon. Database connections in the squirrels framework can be added and configured either in the `connections.py` file, or under the `db_connections` field in the manifest file (`squirrels.yaml`). These two options are provided to allow for greater flexibility in terms of project design, and one can work without the other.

## Adding a database connection in squirrels.yaml

To add a database connection in the squirrels.yaml file, you'll need to add the connection in the form of a collection under `db_connections`. There are two things that you'll need to add:

1. The name of the connection. This is the name that we'll refer to for this connection in the other files.
2. Under the name that defines the collection, you'll need to provided a sqlalchemy url named `url` The syntax for the URL uses [sqlalchemy database URLs](https://docs.sqlalchemy.org/en/20/core/engines.html#database-urls).
3. If you decide to use credentials, you'll also need to provide a `credential_key` in the field as well. Credentials can be set using the `squirrels set-credential` command covered here: [credential management](../cli/credentials.md)

The `db_connections` section should look something like this:

```yaml
db_connections: # optional if connections.py exists
  default:
    url: 'sqlite:///./database/seattle_weather.db'
```

If you decide to use credentials, a new field for the credential_key should be provided, and depending on the type of database used, masks for username and password can also be used. This should look something like below.

```yaml
db_connections: # optional if connections.py exists
  default: 
    credential_key: example # optional if null
    url: 'sqlite://${username}:${password}@/./database/sample_database.db'
```

A connection named "default" must be specified here unless it's specified in the connections.py file. Any database view or DataSource parameter that doesn't specify a database connection will use "default".

## Adding a database connection in the connections.py file

To add a connection in the `connections.py` file, you'll need to create a connection engine/pool using the same sqlalchemy URL as one would do in the `squirrels.yaml` file as well. Again, the schema for the URL can be found here: [sqlalchemy database URLs](https://docs.sqlalchemy.org/en/20/core/engines.html#database-urls). After that's created, it will then need to be passed to the return statement in the form of a python dictionary, where the key corresponding to the connection engine is the name that should be referred to in the `db_connection` field under each dataset in the `squirrels.yaml` files.

If credentials are needed, they can also be added by providing the credential key to the get_credential function. And the username and passwords can be accessed with cred.username and cred.password respectively. Refer to the SQLAlchemy URL guide on how to use the credentials in the urls.

The squirrel framework also allows the user to use a QueuePool in place of the create_engine function. 

```python
def main(proj: Dict[str, Any], *p_args, **kwargs) -> Dict[str, Union[Engine, Pool]]:

    ## Example of getting the username and password set with "$ squirrels set-credential [key]"
    # cred = get_credential('my_key') # then use cred.username and cred.password to access the username and password

    # Create a connection pool / engine
    pool = create_engine('sqlite:///./database/sample_database.db')

    ## Example of using QueuePool instead for a custom db connector:
    # connection_creator = lambda: sqlite3.connect('./database/sample_database.db', check_same_thread=False)
    # pool = QueuePool(connection_creator)
    
    return {'default': pool}
```

A connection named "default" must be specified here unless it's specified in the squirrels.yaml file. Any database view or DataSource parameter that doesn't specify a database connection will use "default".
