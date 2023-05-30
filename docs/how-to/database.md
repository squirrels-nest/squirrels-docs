# Configure Database Connections

All database connections are currently handeled by sqlalchemy in the current version, with future expansion into no-sql databases on the horizon. Database connections in the squirrels framework can be added and configured either the `connections.py` file, or under the `db_connections` field in the manifest file (`squirrels.yaml`). These two options are provided to allow for greater flexibility in terms of project design. They have the equal functionalities, one can work without the other.

## Adding a database connection in squirrels.yaml

To add a database connection in the squirrels.yaml file, you'll need to add the connection in the form of a collection under the `db_connections` collection. There are two things that you'll need to add:
1. The name of the connection, which, for example, is `default` in the seattle_weather.db. This is the name that we'll refer to for this connection in the other files
2. Under the name that defines the collection, you'll need to provided a sqlalchemy url named `url`
3. If you decide to use credentials, you'll also need to provide a `credential_key` in the field as well. TODO, ask Tim how this actually works

The syntax for the URL uses [sqlalchemy database URLs](https://docs.sqlalchemy.org/en/20/core/engines.html#database-urls). Since sqlite databases don't require a username and password, the `credential_key` field can be omitted. More details on setting and using credential keys can be found on the pages for "[credential management]" command line reference and how to "[Configure Database Connections]".

The `db_connections` section should now look like this:

```yaml
db_connections:
  default:
    url: 'sqlite:///./database/seattle_weather.db'
```
if you decide to use credentials, the section should look something like this:

```yaml
db_connections: # optional if connections.py exists
  default: 
    credential_key: null # optional if null
    url: 'sqlite://${username}:${password}@/./database/sample_database.db'
```

## Adding a database connection in the connections.py file

To add a connection in the connections.py file, you'll need to initialize a connector using the `partial()` function on whichever connection function is used by your database, and add in the connector in the form of a QueuePool as a python library in the return statement of the `main()` function in the `connections.py` script.

TODO, make sure with Tim on the requirements of the connector

```python
    # cred = sq.get_credential('my_key')
    # user, pw = cred.username, cred.password
    
    sample_db_connector = partial(sqlite3.connect, './database/colleges.db', check_same_thread=False)
    return ConnectionSet({
        'my_db': QueuePool(sample_db_connector)
    })
```

Next, create a squirrels database connection key. To do this, go in the `connections.py` file, and change the sqlalchemy url to the `seattle_weather.db` database.

In the return statement, you may change the `my_db` key to a new name of your choice for the database connection key. For the rest of this tutorial, we will assume the profile name is `sqlite_db`.

At this point, your `connections.py` file should look something like this (ignoring comments).

```python
from typing import Dict
from sqlalchemy import create_engine, QueuePool

from squirrels import ConnectionSet, get_credential

def main(proj: Dict[str, str], *args, **kwargs) -> ConnectionSet:
    pool = create_engine('sqlite:///./database/seattle_weather.db')
    return ConnectionSet({'sqlite_db': pool})
```

In the `squirrels.yaml` file, set the `db_connection` to `sqlite_db`.