# Create the SQL Queries

In the `weather_by_time` dataset folder, rename `database_view1.sql.j2` to `aggr_weather_metrics.sql.j2` if you haven't already. The `final_view.sql.j2` file name can remain the same.

In these files, we will write the analytical sql query to return tabular results for the dataset. These sql query can be templated using Jinja, with access to the `prms`, `ctx`, and `args` variables which stand for "Parameter Set", "Context", and "Arguments" respectively. More information about these variables can be found in the how to "[Create Views with SQL and Jinja]" page. For now, just know that we can access the selected parameter value(s) by using `prms["parameter name"]` in Jinja.

## Define the Database View

In `aggr_weather_metrics.sql.j2`, change its contents to the following:

```sql
{% set dim_col = prms["group_by"].get_selected("dim_col") -%}
{% set order_col = prms["group_by"].get_selected("order_by_col", default_field="dim_col") -%}

SELECT {{ dim_col }}
    , {{ order_col }} as ordering
    , avg(temp_max) as temperature_high_C
    , avg(temp_min) as temperature_low_C
    , avg(precipitation) as precipitation_inches
    , avg(wind) as wind_mph
FROM weather
GROUP BY {{ dim_col }}, {{ order_col }}
```

This query finds the average temperature, precipitation level, and wind speed by year, or by year of month, or by day of year, etc. based the selected value of the `group_by` parameter.

The `set` keyword is Jinja syntax for assigning variables. The `prms['group_by']` returns a `SingleSelectParameter` (as we previously defined in `parameters.py`), which contains the method `.get_selected()` for getting specific fields of the selected `SelectParameterOption`. We've previously defined `dim_col` in all the options in `parameters.py`, but only specified `order_by_col` for one of the option. The `.get_selected()` has the argument "default_field" to pick `dim_col` for the `order_by_col` if `order_by_col` does not exist as a custom field.

## Define the Final View

In `final_view.sql.j2`, change its contents to the following:

```sql
{% set dim_col = prms["group_by"].get_selected("dim_col") -%}

SELECT {{ dim_col }}
    , temperature_high_C
    , temperature_low_C
    , precipitation_inches
    , wind_mph
FROM aggregate_weather_metrics
ORDER BY ordering
```

A few things to notice here:

1. The table `aggregate_weather_metrics` in the "FROM" clause is the name of the database view we defined in `aggr_weather_metrics.sql.j2`. The name was specified in `squirrels.yaml`.
2. In this query, we are selecting all columns except the `ordering` column, which is what we use in the "ORDER BY" clause instead.
3. The first line where we set `dim_col` is repeated with the database view file. This can be avoided either by using [Jinja's include/import], or by using a `context.py` file which will be discussed later in the tutorial.

See the next tutorial page to [Test the REST APIs](testapi.md).

[Create Views with SQL and Jinja]: ../how-to/sql-views.md
[Jinja's include/import]: https://ttl255.com/jinja2-tutorial-part-6-include-and-import/
