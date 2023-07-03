# Create Views with Python

**Note**
This is the Python version of how to create views, for the SQL template > version see: [Create Views with SQL and Jinja](sql-views.md)

There are two types of views that are relavent to a squirrels project: database views, and final views. The "database view" refers to the view that's directly selected from the database using a connection as described here [Creating a Database Connection](../how-to/database.md), while on the other hand, final view queries from the said database view to generate the final result that is presented to the front end.

## Creating a database view

Regardless whether you use Python or SQL for your database view, you'll need create the parameters in the parameters.py file. This is delineated here: [Creating Parameters](writing-parameters.md). 

After that's done, these are the following steps to create views with python:

1. Create sample file by running the following command in the terminal: `squirrels init --core --db-view py` for creating a database view,
`squirrels init --final-view py` for creating a final view. 
 This will create a sample database/final view in the `sample_dataset` folder.
2. Copy over the file from the `sample_dataset` folder to your desired dataset folder, and rename the file accordingly. 
3. Update the file name in `squirrels.yaml` to the name of the file you've just coppied over.
4. Populate the file with the query template for your dataset.


### Creating an empty file and copying (Step 1 - 2)
Once you run the `squirrels init --core --db-view py` command, a `database_view1.py` file should be generated in the `sample_dataset` folder, or if you ran `squirrels init --final-view py`, you'll get `final_view.py`. These files can then serve as  templates for the subsequent SQL database views that you'll need to  create for your project. 

After you've created and located the file(s), you can then copy it over to your desired dataset folder, and rename your file(s) accordingly. At this point, your file system should look something like this:

```
<root_project_folder>
...
- \datasets
-- \sample_dataset
--- \database_view1.py
--- \...
-- \<your_dataset>
--- \<new_database_view_name>.py
--- \<new_final_view_name>.py
--- \...
-- \..
```

### Adding the view to squirrels.yaml (Step 3)

After a view has been created, regardless of whether it's using SQL or Python, the views will need to be added to the manifest files in order for the project to be able to see the files. Under `datasets`, construct a field named after the folder the dataset is housed in, and provide the label for the dataset. Under `database_views`, provide each database views' file name, name of the connection under `db_connection`, and any additional arguments {TBA}. Make sure to add the file suffixes.

The datasets field should have at least this much:

```yaml
datasets:
  <your_dataset>:
    label: <your_dataset lable>
    database_views:
      database_view1: 
        file: <new_database_view_name>.sql.j2
        db_connection: default # optional if default
        args: {} # optional if empty
    final_view: <new_final_view_name>.sql.j2
    args: {} # optional if empty
```
### Populating the file (Step 4)
After you've copied over the file as created, you'll see the follwing Python template for the database view:

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

If you had created a final view, it should looks something like this, which just returns everything from the database view: 

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

Inside the `main()` function for the database view, you'll be able to access the database connection by frist using `connection_set.get_connection_pool("<connection_name>")` to get the connection pool, from which you can get a DBAPI/sqlalchemy connection, where you can query databases, and make tranformations on dataframes using any Pandas function/methods. 

On the other hand, the final view holds whatever additional transformation that needs to happen after the queries for the database views have been returned. The results from the database views are stored in the `database_views` dictionary, and individual dataframes can be retrieved by referring to the name of the view. The final view is not strictly needed, but can be useful depending on the project.  

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

The parameters are stored in a Python dictionary named `prms`, which is returned from the `parameters.py` file, and individual parameters can be retrieved by referring to the name of the parameter as specified there. The values selected from the parameters can be retrieved by using the `get_selected_value()` method, or similar methods on the parameter. 

These views are just examples, feel free to delete them and make your own view from scratch. 

Both the final and database views have access to the following keywords:

1. prms
2. ctx
3. args

The first `prms` variable contains all the parameters as specificed and returned in the `parameters.py` file in the form of a dictionary, where the key is the name of the parameter (the first argument passed into the parameter's constructor). Likewise, the `ctx` variable is a dictionary as well consisting of parameter options returned by `context.py`. The `args` is also a similar dictionary, but contains the project information in the `squirrels.yaml` file, see [Manifest File (squirrels.yaml)](../topics/manifest.md) for more information. 

As shown in the example above, a parameter can be retrieved in the template like a standard Python map within a double curly bracket `{{}}` by specifying the key like `prms['upper_bound']`, and calling the method format the selected parameters, ie `prms['upper_bound'].get_selected_value()`






