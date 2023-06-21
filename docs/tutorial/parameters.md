# Create the Dataset Parameters

Go into the `parameters.py` file in the `weather_by_time` dataset folder. This file contains the definitions of all the widget parameters used in the dataset through a `main` function. The possible widget parameter classes supported today are `SingleSelectParameter`, `MultiSelectParameter`, `DateParameter`, `NumberParameter`, and `DataSourceParameter`. The `DataSourceParameter` represents a parameter based on a lookup table in a database, and can convert itself into one of the other parameter types. The other parameter types should be self-explanatory.

The classes (with constructors) are imported from the `squirrels` library, and all the widget parameter class constructors take `name` and `label` as required arguments. The `name` is what we use to reference the parameter in the dataset queries (as you'll see later in the tutorial), and it's also the name of the corresponding API query parameter. The `label` represents the human-readable display name to show on the user interface. Different parameter constructors have different required parameters, and most Python IDEs should be able to provide information on the required vs. optional parameters for each constructor.

We will start from scratch, so remove all the existing code in the `main` function body.

## Define a Custom Class for Parameter Options

We will create a single-select parameter called "group_by" which will allow us to aggregate our weather metrics by year, quarter, month, etc. After the imports (before the `main` function), we can create our own "parameter option" class by extending from the existing `SelectParameterOption` class from `squirrels` and defining our own attributes. Here is the class definition:

```python
class GroupByOption(sr.SelectParameterOption):
    def __init__(self, id, label, dim_col, order_by_col = None):
        super().__init__(id, label)
        self.dim_col = dim_col
        self.order_by_col = order_by_col if order_by_col is not None else dim_col
```

The `dim_col` specifies the column name to group by, and the `order_by_col` lets us pick a column for ordering the results (or orders by `dim_col` if not specified). If we group by month for instance, we do not want to order by month name alphabetically.

## Define the Parameter Options

Inside the `main` function, we can now use the new `GroupByOption` class to define a list of parameter options as such:

```python
group_by_options = [
    GroupByOption('0', 'Year', 'year'),
    GroupByOption('1', 'Quarter', 'quarter'),
    GroupByOption('2', 'Month', 'month_name', 'month_order'),
    GroupByOption('3', 'Day of Year', 'day_of_year'),
    GroupByOption('4', 'Condition', 'condition')
]
```

The first parameter to the constructor for each `SelectParameterOption` (or `GroupByOption` in this case) is an ID that must be distinct across options and would never change in the future. For example, once the API consumers associates ID "0" to mean "group by year" then that association should never be broken even if the label or the ordering of the dropdown options change.

!!! Note
    The SelectParameterOption class has an `is_default` attribute to specify the parameter option(s) that are selected by default. By default, `is_default` is set to False. When none of the parameter options have `is_default` as True, the first option becomes default for single select parameters.

## Define the Parameters

The `main` function must return a `Sequence[Parameter]` (where `Parameter` is a class from the squirrels library). We can define our "group_by_parameter" as such:

```python
group_by_parameter = sr.SingleSelectParameter('group_by', 'Group By', group_by_options)
```

Since this is the only parameter we are making for this dataset, we can return the parameter set like this:

```python
return [group_by_parameter]
```

The contents of the `parameters.py` file should now look like this:

```python
from typing import Sequence, Dict, Any
import squirrels as sr

class GroupByOption(sr.SelectParameterOption):
    def __init__(self, id, label, dim_col, order_by_col = None):
        super().__init__(id, label)
        self.dim_col = dim_col
        self.order_by_col = order_by_col if order_by_col is not None else dim_col


def main(args: Dict[str, Any], *p_args, **kwargs) -> Sequence[sr.Parameter]:
    group_by_options = [
        GroupByOption('0', 'Year', 'year'),
        GroupByOption('1', 'Quarter', 'quarter'),
        GroupByOption('2', 'Month', 'month_name', 'month_order'),
        GroupByOption('3', 'Day of Year', 'day_of_year'),
        GroupByOption('4', 'Condition', 'condition')
    ]

    group_by_parameter = sr.SingleSelectParameter('group_by', 'Group By', group_by_options)
    
    return [group_by_parameter]
```
