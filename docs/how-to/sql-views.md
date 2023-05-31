# Create Views with SQL and Jinja

There are two types of views that are relavent to a squirrels project: database views, and final views. The "database view" refers to the view that's directly selected from the database using a connection as described here [Creating a database connection], while on the other hand, final view queries from the said database views to genearte the final dataset that is sent to the front end.

## Creating database views 

SQL database views are Jinja templates saved in the dataset folder that use the parameters specified in the `parameters.py` file. Before writing the templates, you'll need to specify the kinds of parameters that would be used. Below is the example included in the project: 

```python
def main(args: Dict[str, Any], *p_args, **kwargs) -> sq.ParameterSet:
    single_select_options = (
        sq.SelectParameterOption('a0', 'Primary Colors'),
        sq.SelectParameterOption('a1', 'Secondary Colors')
    )
    
    multi_select_options = (
        sq.SelectParameterOption('x0', 'Red',    parent_option_id='a0'),
        sq.SelectParameterOption('x1', 'Yellow', parent_option_id='a0'),
        sq.SelectParameterOption('x2', 'Blue',   parent_option_id='a0'),
        sq.SelectParameterOption('x3', 'Green',  parent_option_id='a1'),
        sq.SelectParameterOption('x4', 'Orange', parent_option_id='a1'),
        sq.SelectParameterOption('x5', 'Purple', parent_option_id='a1')
    )

    single_select_example = sq.SingleSelectParameter('color_type', 'Color Type', single_select_options)

    multi_select_example = sq.MultiSelectParameter('colors', 'Colors', multi_select_options,
                                                   parent=single_select_example)
    
    date_example = sq.DateParameter('as_of_date', 'As Of Date', '2020-01-01')
    
    number_example = sq.NumberParameter('upper_bound', 'Upper Bound', min_value=1, max_value=10, 
                                        increment=1, default_value=5)
    
    return sq.ParameterSet(
        [single_select_example, multi_select_example, date_example, number_example]
    )
```

Currently, there are four supported types of parameters:
1. Single Select Parameter
Among the options provided, only one would be able to be selected from the UI.
2. Multi-select Parameter
Among the options provided, one or more options can be selected. 
3. Date Parameter
This is a range parameter, that allows a range of dates to be selected.
4. Number Parameter
This is also a range parameter, where a range of numbers can be selected. 

In general, each constructor to create a parameter object follows the following form:
`sq.XXXParameter(<name>, <display_name>, <Options>)` however, please refer to the docstrings for the most up-to-date documentation on the specific functions. 

After you've spcified the paramters in the `parameters.py` file, the parameters can then be accessed by specifying the name of the parameters in the returned ParameterSet, the specify how the selected parameters should be presented/concatinated in the sql template. Again, refer to the docstrings for the specific parameter that is being used. For example, to get the `upper_bound` parameter from the example `parameter.py` above:

```sql
SELECT dim1, avg(metric1) as metric1, avg(metric2) as metric2
FROM fact_table
WHERE metric1 <= {{ prms['upper_bound'].get_selected_value() }}
GROUP BY dim1
```

In the future, we also plan to add an optional argument analogus to the sql `USE` statement in the sql template in order to have a way to specify which database is to be used. 

## Creating the final view
The final view is what is presented, or the view that is sent accross the API to the front end.
This can be as simple as a `select * from database_view` statement, or a more complex join spanning multiple database views. 


## Adding the view to squirrels.yaml

After a view has been created, regardless of whether it's using SQL or Python, the views will need to be added to the manifest files in order for the project to be able to see the files. Under `datasets`, construct a field named after the folder the dataset is housed in, `sample_dataset` in this case, and provide the label for the dataset. Under `database_views`, provide each database views' file name, name of the connection under `db_connection`, and any additional arguments {TBA}. Make sure to add the file suffixes.

The datasets field should have at least this much:

```yaml
datasets:
  sample_dataset:
    label: Sample Dataset
    database_views:
      database_view1: 
        file: database_view1.sql.j2
        db_connection: default # optional if default
        args: {} # optional if empty
    final_view: final_view.sql.j2
    args: {} # optional if empty
```