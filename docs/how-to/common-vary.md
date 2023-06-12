# Design for Commonality and Variability

## Sharing common SQL syntax using Jinja import and macros
In order to avoid duplications, the Jinja2 framework supports the use of macros to save a piece of SQL code template that can be used by other templates. To those unfamiliar with the term macro, a macro is essentially Jinja's analogy to a function. As we are using Jinja2 for our SQL templates, the squirrels framework supports the use of macros in all its xxx.sql.j2. files. 

To provide an example, let's say that you are analyzing the geographical distribution of different phone makers in the United States. For that purpose, you have three tables from different datasources that you've imported into your database, each with different schemas: 
1. `tbl_android_users`
2. `tbl_users_apple`
3. `tbl_dumbphone_users`

All these tables have a column for user ID, state, and phone ID, and you want a view with all the users, the state or the city (depending on the parameter), the type of phone they use, and a count of the number of phones that they have. If you were to write out the full query, you'd have something like this.

```sql
SELECT 
    ID_USER as USER_ID,
    {{prms['location'].get_selected_value()}},
    'Android' AS PHONE_TYPE,
    COUNT(DISTINCT PHONE_ID) AS CELL_COUNT  
FROM TBL_ANDROID_USERS
GROUP BY USER_ID, {{prms['location'].get_selected_value()}}
UNION ALL
SELECT 
    ID_ACCT as USER_ID,
    {{prms['location'].get_selected_value()}},
    'Apple' AS PHONE_TYPE,
    COUNT(DISTINCT PHONE_ID) AS CELL_COUNT  
FROM TBL_USERS_APPLE
GROUP BY USER_ID, {{prms['location'].get_selected_value()}}
UNION ALL
SELECT 
    ID_CUST as USER_ID,
    {{prms['location'].get_selected_value()}},
    'DumbPhone' AS PHONE_TYPE,
    COUNT(DISTINCT CELL_ID) AS CELL_COUNT  
FROM TBL_DUMBPHONE_USERS
GROUP BY USER_ID, {{prms['location'].get_selected_value()}}
```
where location is a single select parameter that specifies the location, with the following options `CITY`, `COUNTY`, and `STATE`. 

Pretty repetitive, huh? 

Instead of writing out the entire thing, one way we can simplify this is to get write a generic subquery in the form of a macro in a seperate file, which we'll name `phone_macro.sql.j2`. Inside this file, we will need to define a macro by writing:

```sql
{% macro phone_macro(phone_table, user_id_name, phone_id_name, phone_type_name, location) %}
SELECT 
    {{user_id_name}} as USER_ID,
    {{location}},
    '{{phone_type_name}}' AS PHONE_TYPE,
    COUNT(DISTINCT {{phone_id_name}}) AS CELL_COUNT  
FROM {{phonetable}}
GROUP BY USER_ID, {{location}}
{% endmacro %}
```
Here a macro is declared by the `{% macro phone_macro(phonetable, user_id_name, phone_id_name, phone_type_name, location)}` line, where `phone_macro()` is the name of the macro, followed by the required variables. These variables can then be used by the definition of the macro with double curly brackets. The final `{% endmacro %}` commands specifies the end of the definition. 

Supposed it's saved in the same directory, the macro can be imported into the main script by specifying the name of the script, and the name of the macro. 

```sql
{% from 'phone_macro.sql.j2' import phone_macro %}
```

The macro can then be called by using double curly braces, and passing in the variables in the followin manner:
```sql
{{ phone_macro('TBL_ANDROID_USERS', 'ID_USER', 'Android', 'PHONE_ID', prms['location'].get_selected_value()) }}
```

Rewriting the original script to use macros, we get something with much less duplicate code:

```sql
{% from 'phone_macro.sql.j2' import phone_macro %}

{{ phone_macro('TBL_ANDROID_USERS', 'ID_USER', 'Android', 'PHONE_ID', prms['location'].get_selected_value()) }}
UNION ALL
{{ phone_macro('TBL_USERS_APPLE', 'ID_ACCT', 'Apple', 'PHONE_ID', prms['location'].get_selected_value()) }}
UNION ALL
{{ phone_macro('TBL_DUMBPHONE_USERS', 'ID_CUST', 'DumbPhone', 'CELL_ID', prms['location'].get_selected_value()) }}
```

Please note that this is only one possible use case, and that there are millions of other use cases. For some more possible use cases, this guide [here](https://towardsdatascience.com/jinja-sql-%EF%B8%8F-7e4dff8d8778) provides some other use cases of macros used in sql Jinja templates. Although this is not squirrels specific, it should still provide be a nice reference. 

## Sharing code with Python

Similar to using Jinja and SQL, you can also import functions from other scripts using Python as well. To do this, create a seperate script in the same directory as the main database/final view, then write the function that you'd like to use. (you can also create this script in another directory, but will need to specify the path later in the import statement) Then, by using an import statement, you'd able to use that function in your main script. 

To give an example, suppose that you have multiple different dataframes from different sources that you'd like to standardize by renaming the columns, and return joined table of all of them. If you were to do that in all in the same script with the power of copy pasting, you might have something like this:

```python
from typing import Dict, Any
import pandas as pd
import squirrels as sq


def main(database_views: Dict[str, pd.DataFrame], 
         prms: sq.ParameterSet, ctx: Dict[str, Any], proj: Dict[str, Any], 
         *p_args, **kwargs) -> pd.DataFrame:
    df = database_views['database_view1'].rename(columns={"user_id": "id_user", "us_id": "id_user", "cust_id": "id_user", "person_id":"id_user", "geo_id": "location", "state": "location", "city":"location", "country":"location", "account_id": "id_acct", "id_a": "id_acct","account_num": "id_acct"})

    df = df[["id_user","location","id_acct"]]

    df2 = database_views['database_view2'].rename(columns={"user_id": "id_user", "us_id": "id_user", "cust_id": "id_user", "person_id":"id_user", "geo_id": "location", "state": "location", "city":"location", "country":"location", "account_id": "id_acct", "id_a": "id_acct","account_num": "id_acct"})

    df2 = df2[["id_user","location","id_acct"]]

    df3 = database_views['database_view3'].rename(columns={"user_id": "id_user", "us_id": "id_user", "cust_id": "id_user", "person_id":"id_user", "geo_id": "location", "state": "location", "city":"location", "country":"location", "account_id": "id_acct", "id_a": "id_acct","account_num": "id_acct"})

    df3 = df3[["id_user","location","id_acct"]]
    return pd.concat([df1, df2, df3], ignore_index = True)
```

(This is also a prime example of horrible style, please don't write code like this)

In the seperate script, which we will name `functions.py`, we can create a function named `standardize`.

```python
import pandas as pd

def standardize(df: pd.DataFrame) -> pd.DataFrame:
    df.rename(columns={"user_id": "id_user", "us_id": "id_user", "cust_id": "id_user", "person_id":"id_user", "geo_id": "location", "state": "location", "city":"location", "country":"location", "account_id": "id_acct", "id_a": "id_acct","account_num": "id_acct"}, in_place = True)

    return df[["id_user","location","id_acct"]]
```

In the main script, we would then be able to import the function by using the following statement:

```python
from functions import standardize
## write import * if you have other functions and want to import everything
```

Rewriting the first example would result in something like the following:

```python
from typing import Dict, Any
import pandas as pd
import squirrels as sq
from functions import standardize

def main(database_views: Dict[str, pd.DataFrame], 
         prms: sq.ParameterSet, ctx: Dict[str, Any], proj: Dict[str, Any], 
         *p_args, **kwargs) -> pd.DataFrame:
   
    df1 = standardize(database_views['database_view1'])
    df2 = standardize(database_views['database_view2'])
    df3 = standardize(database_views['database_view3'])
    return pd.concat([df1, df2, df3], ignore_index = True)
```

Again, this is a pretty simple example, and importing scripts would allow for much more complex data manipulations to be organized and written in a much more readable manner.

