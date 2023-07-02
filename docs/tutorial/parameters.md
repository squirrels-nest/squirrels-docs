# Create the Dataset Parameters

Go into the `parameters.py` file in the `weather_by_time` dataset folder. This file contains the definitions of all the widget parameters used in the dataset through a `main` function. The possible widget parameter classes supported today are `SingleSelectParameter`, `MultiSelectParameter`, `DateParameter`, `NumberParameter`, and `DataSourceParameter`. The `DataSourceParameter` represents a parameter based on a lookup table in a database, and can convert itself into one of the other parameter types. The other parameter types should be self-explanatory.

The classes are imported from the `squirrels` library, and all the widget parameter class constructors take `name` and `label` as required arguments. The `name` is what we use to reference the parameter in the dataset queries (as you'll see later in the tutorial), and it's also the name of the corresponding API query parameter. The `label` represents the human-readable display name to show on the user interface. Different parameter constructors have different required parameters, and most Python IDEs should be able to provide information on the required vs. optional parameters for each constructor.

We will start from scratch, so remove all the existing code in the `main` function body. We will create one single-select parameter to specify the dimension to group by.

## Define the Parameter Options

We first need to specify the list of parameter options. Inside the `main` function, specify the list of options as such:

```python
group_by_options = [
    sr.SelectParameterOption('0', 'Year', dim_col='year'),
    sr.SelectParameterOption('1', 'Quarter', dim_col='quarter'),
    sr.SelectParameterOption('2', 'Month', dim_col='month_name', order_by_col='month_order'),
    sr.SelectParameterOption('3', 'Day of Year', dim_col='day_of_year'),
    sr.SelectParameterOption('4', 'Condition', dim_col='condition')
]
```

The first two parameters to the `SelectParameterOption` constructors are the ID and label. The ID that must be distinct across options and would never change in the future. For example, once the API consumers associate ID "0" to mean "group by year" then that association should never be broken even if the label or the ordering of the dropdown options change.

Arbitrary keyword arguments such as `dim_col` and `order_by_col` can be specified to the `SelectParameterOption` constructor which will be treated as custom fields to the parameter option that can be used later. See topic guide on [Parameter Option Custom Fields](../topics/custom-fields.md) for more details.

!!! Note
    The SelectParameterOption class has an `is_default` attribute to specify the parameter option(s) that are selected by default. By default, `is_default` is set to False. When none of the parameter options have `is_default` as True, the first option is default for single-select parameters, and nothing is selected by default for multi-select parameter.

## Define the Parameters

The `main` function must return a `Sequence[Parameter]` (where `Parameter` is a class from the squirrels library). We can define our "group_by_parameter" as such:

```python
group_by_param = sr.SingleSelectParameter('group_by', 'Group By', group_by_options)
```

Since this is the only parameter we are making for this dataset, we can return the parameter set like this:

```python
return [group_by_param]
```

## End State

The contents of the `parameters.py` file should now look like this:

```python
from typing import Sequence, Dict, Any
import squirrels as sr


def main(args: Dict[str, Any], *p_args, **kwargs) -> Sequence[sr.Parameter]:
    group_by_options = [
        sr.SelectParameterOption('0', 'Year', dim_col='year'),
        sr.SelectParameterOption('1', 'Quarter', dim_col='quarter'),
        sr.SelectParameterOption('2', 'Month', dim_col='month_name', order_by_col='month_order'),
        sr.SelectParameterOption('3', 'Day of Year', dim_col='day_of_year'),
        sr.SelectParameterOption('4', 'Condition', dim_col='condition')
    ]

    group_by_param = sr.SingleSelectParameter('group_by', 'Group By', group_by_options)

    return [group_by_param]
```

See the next tutorial page to [Create the SQL Queries](queries.md).
