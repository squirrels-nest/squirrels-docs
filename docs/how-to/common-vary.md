# Design for Commonality and Variability

## Sharing common SQL syntax using Jinja import and macros

In order to avoid duplications, the Jinja2 framework supports the use of macros to save a piece of SQL code template that can be used by other templates. To those unfamiliar with the term macro, a macro is essentially Jinja's analogy to a function. As we are using Jinja2 for our SQL templates, the squirrels framework supports the use of macros in all its SQL/Jinja files. 

To provide an example, let's say that you are analyzing the geographical distribution of different phone makers in the United States. For that purpose, you have three tables from different datasources that you've imported into your database, each with different schemas: 

1. `tbl_android_users`
2. `tbl_users_apple`
3. `tbl_dumbphone_users`

All these tables have columns for user ID, location (city, county, state, etc), and phone ID, and you want a view with all the users, the state or the city (depending on the parameter), the type of phone they use, and a count of the number of phones that they have. For this example let's say that you have `parameters.py` defined in the following manner, where location is a single-select parameter that specifies the location with the following options `City`, `County`, and `State`.  

```python
def main(args: Dict[str, Any], *p_args, **kwargs) -> Sequence[sr.Parameter]:

    location_options = [
        sr.SelectParameterOption('0', 'City', column = 'CITY'),
        sr.SelectParameterOption('1', 'County', column = 'COUNTY'),
        sr.SelectParameterOption('2', 'State', column = 'STATE')
    ]

    location_param = sr.SingleSelectParameter('location', 'Location', location_options)
    
    return [location_param]
```

If you were to write out the full query, you'd have something like this.

```sql
SELECT 
    ID_USER as USER_ID,
    {{ prms['location'].get_selected("column") }},
    'Android' AS PHONE_TYPE,
    COUNT(DISTINCT PHONE_ID) AS CELL_COUNT  
FROM TBL_ANDROID_USERS
GROUP BY USER_ID, {{ prms['location'].get_selected("column") }}
UNION ALL
SELECT 
    ID_ACCT as USER_ID,
    {{ prms['location'].get_selected("column") }},
    'Apple' AS PHONE_TYPE,
    COUNT(DISTINCT PHONE_ID) AS CELL_COUNT  
FROM TBL_USERS_APPLE
GROUP BY USER_ID, {{ prms['location'].get_selected("column") }}
UNION ALL
SELECT 
    ID_CUST as USER_ID,
    {{ prms['location'].get_selected("column") }},
    'DumbPhone' AS PHONE_TYPE,
    COUNT(DISTINCT CELL_ID) AS CELL_COUNT  
FROM TBL_DUMBPHONE_USERS
GROUP BY USER_ID, {{ prms['location'].get_selected("column") }}
```

Pretty repetitive, huh? 

Instead of writing out the entire thing, one way we can simplify this is to write a generic subquery in the form of a macro in a seperate file, which we'll name `macros.sql.j2`. Inside this file, we will need to define a macro by writing:

```sql
{% macro phone_macro(phone_table, user_id_column, phone_id_name, phone_type_name, location) -%}
SELECT 
    {{ user_id_column }} as USER_ID,
    {{ location }},
    '{{ phone_type_name }}' AS PHONE_TYPE,
    COUNT(DISTINCT {{ phone_id_name }}) AS CELL_COUNT  
FROM {{ phone_table }}
GROUP BY USER_ID, {{ location }}
{%- endmacro -%}
```

Here, a macro named `phone_macro` is defined, followed by the required variables. These variables can then be used by the definition of the macro with double curly brackets. The final `{%- endmacro -%}` commands specifies the end of the definition. 

Suppose the file is saved in the "datasets" folder. The macro can be imported into the main script by specifying the path of the script (relative to the project root), and the name of the macro. 

```sql
{% from 'datasets/macros.sql.j2' import phone_macro -%}
```

The macro can then be called by using double curly braces, and passing in the variables in the following manner:

```sql
{{ phone_macro('TBL_ANDROID_USERS', 'ID_USER', 'Android', 'PHONE_ID', prms['location'].get_selected("column")) }}
```

An alternative way to refer to the "phone_macro" can also be done as such:

```sql
{% import 'datasets/macros.sql.j2' as m -%}

{{ m.phone_macro('TBL_ANDROID_USERS', 'ID_USER', 'Android', 'PHONE_ID', prms['location'].get_selected("column")) }}
```

Rewriting the original script to use macros, we get something with much less duplicate code:

```sql
{% from 'datasets/macros.sql.j2' import phone_macro -%}

{% set location_col = prms['location'].get_selected("column") -%}

{{ phone_macro('TBL_ANDROID_USERS', 'ID_USER', 'Android', 'PHONE_ID', location_col) }}
UNION ALL
{{ phone_macro('TBL_USERS_APPLE', 'ID_ACCT', 'Apple', 'PHONE_ID', location_col) }}
UNION ALL
{{ phone_macro('TBL_DUMBPHONE_USERS', 'ID_CUST', 'DumbPhone', 'CELL_ID', location_col) }}
```

Please note that this is only one possible use case, and that there are millions of other use cases. For some more possible use cases, this guide [here](https://towardsdatascience.com/jinja-sql-%EF%B8%8F-7e4dff8d8778) provides some other use cases of macros used in sql Jinja templates. Although this is not squirrels specific, it should still serve as a nice reference. 

## Sharing code with Python

Similar to using Jinja and SQL, you can also import functions from other scripts using Python as well. 

To give an example, suppose that you have multiple different dataframes from different sources that you'd like to standardize by renaming the columns, and return joined table of all of them. If you were to do that all in the same script with the power of copy/pasting, you might have something like this:

```python
from typing import Dict, Any
import pandas as pd
import squirrels as sr


def main(database_views: Dict[str, pd.DataFrame], 
         prms: Dict[str, sr.Parameter], ctx: Dict[str, Any], args: Dict[str, Any], 
         *p_args, **kwargs) -> pd.DataFrame:
    df1 = database_views['database_view1'].rename(columns={"user_id": "id_user", "geo_id": "location", "account_id": "id_acct"})
    df1 = df1[["id_user","location","id_acct"]]

    df2 = database_views['database_view2'].rename(columns={"us_id": "id_user", "state": "location", "id_a": "id_acct"})
    df2 = df2[["id_user","location","id_acct"]]

    df3 = database_views['database_view3'].rename(columns={"cust_id": "id_user", "city": "location", "account_num": "id_acct"})
    df3 = df3[["id_user","location","id_acct"]]

    return pd.concat([df1, df2, df3], ignore_index = True)
```

(This is also a prime example of horrible style, please don't write code like this).

In the seperate script, which we will name `functions.py`, we can create a function named `standardize`.

```python
import pandas as pd

def standardize(df: pd.DataFrame, id_user_col: str, location_col: str, id_acct_col: str) -> pd.DataFrame:
    df = df.rename(columns={id_user_col: "id_user", location_col: "location", id_acct_col: "id_acct"})
    return df[["id_user","location","id_acct"]]
```

Suppose `functions.py` is saved in the datasets folder. In the main script, we would then be able to import the file relative to the root of the project:

```python
from datasets import functions as f
```

Rewriting the first example would result in something like the following:

```python
from typing import Dict, Any
import pandas as pd
import squirrels as sr

from datasets import functions as f

def main(database_views: Dict[str, pd.DataFrame], 
         prms: Dict[str, sr.Parameter], ctx: Dict[str, Any], args: Dict[str, Any], 
         *p_args, **kwargs) -> pd.DataFrame:
   
    df1 = f.standardize(database_views['database_view1'])
    df2 = f.standardize(database_views['database_view2'])
    df3 = f.standardize(database_views['database_view3'])
    return pd.concat([df1, df2, df3], ignore_index = True)
```

Again, this is a pretty simple example, and importing scripts would allow for much more complex data manipulations to be organized and written in a much more readable manner.
