# Create Views with SQL and Jinja

There are two types of views that are relavent to a squirrels project: database views, and final views. The "database view" refers to the view that's directly selected from the database using a connection as described here [Creating a database connection], while on the other hand, final view queries from the said database view to generate the final result that is presented to the front end.

## Creating database views 
SQL database views are Jinja templates that use the parameters specified in the `parameters.py` file. Before writing the templates, you'll need to specify the kinds of parameters that would be used. Below is the example included in the project: 

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
`sq.XXXParameter(<name>, <display_name>, <Options>)` however, please refer to the doc strings for the most up-to-date documentation on this. 

After you've spcified the paramters in the parameters.py file, the parameters can then be accessed by specifying the name of the parameters in the returned ParameterSet, the specify how the selected parameters should be presented/concatinated in the sql template. Again, refer to the docstrings for the specific parameter that is being used. For example, to get the `upper_bound` parameter from the example `parameter.py` above:

```sql
SELECT dim1, avg(metric1) as metric1, avg(metric2) as metric2
FROM fact_table
WHERE metric1 <= {{ prms['upper_bound'].get_selected_value() }}
GROUP BY dim1
```