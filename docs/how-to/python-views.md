# Create Views with Python

There are two types of views that are relavent to a squirrels project: database views, and final views. The "database view" refers to the view that's directly selected from the database using a connection as described here [Creating a database connection], while on the other hand, final view queries from the said database view to generate the final result that is presented to the front end.

## Creating a database view

Regardless whether you use Python or SQL for your database view, you'll need create the parameters in the parameters.py file. This is delineated in the SQL version of creating a database view here: [Create Views with SQL and Jinja]. 

After those have been created, save a python script into the dataset folder, and populate it with the following template: 

```python
from typing import Dict, Any
import pandas as pd

import squirrels as sq


def main(connection_set: sq.ConnectionSet, 
         prms: sq.ParameterSet, ctx: Dict[str, Any], args: Dict[str, Any], 
         *p_args, **kwargs) -> pd.DataFrame:
    # pool = connection_set.get_connection_pool("default")
    # conn = pool.connect() # use this to get a DBAPI connection from a Pool or 
    # sqlalchemy connection from an Engine
    # conn = pool.raw_connection() # use this to get a DBAPI connection from an Engine

    # limit: str = ctx['limit'] # use this instead if context.py is defined
    return None
```

Inside the `main()` function, you'll be able to access the database connection by frist using `connection_set.get_connection_pool("<connection_name>")` to get the connection pool, from which you can get a DBAPI/sqlalchemy connection, where you can query databases, and make tranformations on dataframes using any Pandas function. 

The parameters are stored in a Python dictionary named `prms`, which is what is returned from the `parameters.py` file, and individual parameters can be retrieved by referring to the name of the parameter as specified there. The values selected from the parameters can be retrieved by using the `get_selected_value()` method, or similar methods on the parameter. 

Below is an example using a hard-coded dataframe. 

```python
from typing import Dict, Any
import pandas as pd
import squirrels as sq

def main(connection_set: sq.ConnectionSet, 
         prms: sq.ParameterSet, ctx: Dict[str, Any], args: Dict[str, Any], 
         *p_args, **kwargs) -> pd.DataFrame:
    # pool = connection_set.get_connection_pool("default")
    # conn = pool.connect() # use this to get a DBAPI connection from a Pool or 
    # sqlalchemy connection from an Engine
    # conn = pool.raw_connection() # use this to get a DBAPI connection from an Engine

    df = pd.DataFrame({
        'dim1': ['a', 'b', 'c', 'd', 'e', 'f'], 
        'metric1': [1, 2, 3, 4, 5, 6], 
        'metric2': [2, 4, 5, 1, 7, 3]
    })
    limit_parameter: sq.NumberParameter = prms['upper_bound']
    limit = limit_parameter.get_selected_value() 
    # limit: str = ctx['limit'] # use this instead if context.py is defined

    return df.query(f'metric1 <= {limit}')
```

## Creating the final view
The final view holds whatever additional transformation that needs to happen after the queries for the database views have been returned. The results from the database views are stored in the `database_views` dictionary, and individual dataframes can be retrieved by referring to the name of the view. The final view is not strictly needed, but can be useful depending on the project.  Use the following template as an starting point:

```python
from typing import Dict, Any
import pandas as pd
import squirrels as sq


def main(database_views: Dict[str, pd.DataFrame], 
         prms: sq.ParameterSet, ctx: Dict[str, Any], proj: Dict[str, Any], 
         *p_args, **kwargs) -> pd.DataFrame:
    df = database_views['database_view1']
    return df
```



## Adding the view(s) to squirrels.yaml

Like the sql counterpart, the views will need to be added to the manifest files in order for the project to be able to see the files. Under `datasets`, construct a field named after the folder the dataset is housed in, `sample_dataset` in this case, and provide the label for the dataset. Under `database_views`, provide each database views' file name (including the final view), name of the connection under `db_connection`, and any additional arguments {TBA}. Make sure to add the file suffixes.

The datasets field should have at least this much:

```yaml
datasets:
  sample_dataset:
    label: Sample Dataset
    database_views:
      database_view1: 
        file: database_view1.py
        db_connection: default # optional if default
        args: {} # optional if empty
    final_view: final_view.py
    args: {} # optional if empty
```




