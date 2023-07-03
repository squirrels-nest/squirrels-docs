# Creating Parameters

All views, regardless of whether it's written using Python or SQL, saved in the dataset folder that the parameters specified in the `parameters.py` and `context.py` files. Before writing the templates, you'll need to specify the kinds of parameters that would be used. Below is an example of a `parameters.py` file included in the project: 

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

    multi_select_example = sq.MultiSelectParameter('colors', 'Colors', multi_select_options,  parent=single_select_example)
    
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


In order to create a parameter, here are some basic steps:

1. Create a `parameters.py` file for you dataset, this should be automatically generated in a sample dataset when you ran `squirrels init`.
2. Copy/move the `parameters.py` file from the original sample dataset folder to your own dataset folder.
3. In the file, create/Specify your parameter options
4. Initialize parameters
5. Add parameter to the return list


Steps 1 - 2 are self explanatory, so we will not go in detail for those. 

## Specifying parameter options (Step 3)

All parameters need options to choose from, and in the squirrels framework, most parameter options come in the form of a `SelectParameterOption` object, which houses all the data that pertains to that specific option, ie. its ID, label, and other custom attributes. We then use a list to contain all the parameter options for a single parameter, which will get passed through to the parameter constructor. 

Using the example above, the parameter options for the single select and multi select is as following:

```python
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
```

where the first argument is the SelectParameterOption's ID, and the next is its label. Those arguments are necessary, while the rest are optional. 

An example of a useful optional argument is the `parent_option_id` that's also in the example above. When specified, the parameter option would only show up, if the parameter id with the `parent_option_id` is selected. 

## Creating the parameters (Step 4)

After the parameter options are specified and aggregated, the list can then get passed to the parameters parameter constructor.

```python 
    single_select_example = sq.SingleSelectParameter('color_type', 'Color Type', single_select_options)

    multi_select_example = sq.MultiSelectParameter('colors', 'Colors', multi_select_options,  parent=single_select_example)
```

In general, each constructor to create a parameter object follows the following form: `sq.XXXParameter(<id>, <label>, <Options>)` however, please refer to the docstrings for the most up-to-date documentation on the specific functions. 

Note how since the multiselect parameter options uses the `parent_option_id` field, the call to the MultiSelectParameter's constructor also specifies the parent parameter to which the options refer.

For the Date, Number, and NumRange parameters, please refer to the docstrings.

TODO -- Add explanation for the Dataset parameter.

For more information on what parameters can do, please refer to the [Parameter Option Custom Fields](custom-fields.md) page for information on how to create custom fields.

## Adding the parameters to the return set (Step 5)

All parameters need to be added to return list in order to be used in the database and final views. This is very straight forward, as you simply need to aggregate all the parameters you've specified into a list, and return it. 

```python
return [single_select_example, multi_select_example, date_example, number_example]
```