# Use the Context File

Let's revisit the SQL/Jinja files. In both files, we use `prms["group_by"].get_selected()` to get the selected parameter option from the `group_by` parameters, and access the `dim_col` attribute from the object. Doing all this in a SQL/Jinja file can be a poor developer experience, especially if you're using an IDE that can provide auto-completion for Python files.

Instead, we can use the `context.py` file to improve the developer experience. We use its `main` function to transform all selected parameter options into useful values (usually strings) returned as dictionary values. Then, in the SQL/Jinja files, the dictionary can be referenced using the `ctx` keyword.

For example, we can update the `context.py` file contents to look like this:

```python
from typing import Dict, Any
import squirrels as sq

from datasets.weather_by_time.parameters import GroupByOption

def main(prms: sq.ParameterSet, args: Dict[str, Any], *p_args, **kwargs) -> Dict[str, Any]:
    group_by_param: sq.SingleSelectParameter = prms["group_by"]
    selected_group_by: GroupByOption = group_by_param.get_selected()
    return {
        "dim_col": selected_group_by.dim_col,
        "order_col": selected_group_by.order_by_col
    }
```

Notice that type hints were added for variables `group_by_param` and `selected_group_by`. This is useful to provide the IDE information on suggesting methods to auto-complete. For instance, with a list of suggestions, we don't have to memorize that the `.get_selected()` method exists for SingleSelectParameter objects to get the selected parameter option, or memorize what the method names would be for other parameter classes. Auto-completion would allow these methods to be discovered more easily.

Now the SQL queries can be written more concisely to reference the context variables instead.

The contents for `aggr_weather_metrics.sql.j2` can now be changed to the following.

```sql
SELECT {{ ctx["dim_col"] }}
    , {{ ctx["order_col"] }} as ordering
    , avg(temp_max) as temperature_high_C
    , avg(temp_min) as temperature_low_C
    , avg(precipitation) as precipitation_inches
    , avg(wind) as wind_mph
FROM weather
GROUP BY {{ ctx["dim_col"] }}, {{ ctx["order_col"] }}
```

In addition, the contents for `final_view.sql.j2` can now be changed to the following.

```sql
SELECT {{ ctx["dim_col"] }}
    , temperature_high_C
    , temperature_low_C
    , precipitation_inches
    , wind_mph
FROM aggregate_weather_metrics
ORDER BY ordering
```

## Final Thoughts

A Squirrels project may also contain multiple datasets, or datasets with multiple database views, but that's outside the scope of this tutorial. For an expanded version of the "seattle weather example" project, see this git repo [here](https://github.com/squirrels-nest/weather-analytics-api-example). It serves as an example of how to share common Python code or SQL across multiple datasets while allowing their parameters or query definitions to not be identical. You can also eee this page for how to "[Design for Commonality and Variability]".

[Design for Commonality and Variability]: ../how-to/common-vary.md
