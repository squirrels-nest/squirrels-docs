# Use the Context File

Let's revisit the SQL/Jinja files. In both files, we use `prms["group_by"].get_selected("dim_col")` to get the `dim_col` attribute from the selected parameter option. Doing all this in a SQL/Jinja file can be a poor developer experience, especially if you're using an IDE that can provide auto-completion for Python files.

Instead, we can use the `context.py` file to improve the developer experience. We use its `main` function to transform all selected parameter options into useful values (usually strings) returned as dictionary values. Then, in the SQL/Jinja files, the dictionary can be referenced using the `ctx` keyword.

For example, we can update the `context.py` file contents to look like this:

```python
from typing import Dict, Any
import squirrels as sr


def main(prms: Dict[str, sr.Parameter], args: Dict[str, Any], *p_args, **kwargs) -> Dict[str, Any]:
    group_by_param: sr.SingleSelectParameter = prms["group_by"]
    
    return {
        "dim_col": group_by_param.get_selected("dim_col"),
        "order_col": group_by_param.get_selected("order_by_col", default_field="dim_col")
    }
```

Notice that type hints were added to `group_by_param`. This is useful to provide the IDE information on suggesting methods to auto-complete. For instance, with a list of suggestions, we don't have to memorize that the `.get_selected()` method exists for SingleSelectParameter objects to get the selected parameter option, or memorize what the method names are available for other parameter classes. Auto-completion would allow these methods to be discovered more easily.

At this point, the SQL queries can be written more concisely to reference the context variables instead.

The contents for `aggr_weather_metrics.sql.j2` can be changed to the following.

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

Congratulations, you have reached the end of the tutorial! 
