# Creating Parameters


SQL database views are Jinja templates saved in the dataset folder that use the parameters specified in the `parameters.py` and `context.py` files. Before writing the templates, you'll need to specify the kinds of parameters that would be used. Below is an example of a `parameters.py` file included in the project: 

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
    
    return [single_select_example, multi_select_example, date_example, number_example]
```

Currently, there are three supported types of parameters:

1. Single Select Parameter
Among the options provided, only one would be able to be selected from the UI.
2. Multi-select Parameter
Among the options provided, one or more options can be selected. 
3. Date Parameter
This is a range parameter, that allows a range of dates to be selected.
4. Number Parameter
This is also a range parameter, where a range of numbers can be selected. 
5. NumRange Parameter
6. Dataset Parameter

In general, each constructor to create a parameter object follows the following form: `sq.XXXParameter(<name>, <display_name>, <Options>)` however, please refer to the docstrings for the most up-to-date documentation on the specific functions. 