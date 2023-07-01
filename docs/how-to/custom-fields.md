# Parameter Option Custom Fields

When building out multi-select or single-select parameters, you can associate all parameter option with a custom attribute, and access the attribute for the selected parameter option(s) at runtime in `context.py` or the query views.

Suppose you want to build a parameter for "time period" that has options "Year to Date" or "Year over Year", and you want to associate each option with one of squirrel's DateModifier classes. You would:

1. Add the custom field "date_modifier" to each parameter option in `parameters.py`. Can be specified as arbitrary keyword arguments or using the `custom_fields` argument.
2. Access the custom "date_modifier" at runtime for the selected option, and use it to transform some date string.

More details on squirrel's date modifiers can be found in the [Modifying Dates Using Squirrel's dateutils](../topics/modify-dates.md) topic guide.

Custom fields can also come from lookup tables as "custom columns" when using the `custom_cols` argument in the constructor of `SelectionDataSource` for a `DataSourceParameter`.

## 1. Associating Custom Fields to Parameter Options

In `parameters.py`, you can use arbitrary keyword arguments as follows to specify custom fields. Note the `date_modifier` keyword arguments to `SelectParameterOption`.

```python
from squirrels import dateutils as du
import squirrels as sr

...

ytd_date_modifier = du.DateStringModifier([du.DayIdxOfYear(1)])
yoy_date_modifier = du.DateStringModifier([du.OffsetYears(-1)])
time_period_options = [
    sr.SelectParameterOption("t0", "Year to Date", date_modifier=ytd_date_modifier),
    sr.SelectParameterOption("t1", "Year over Year", date_modifier=yoy_date_modifier)
]
time_period_param = sr.SingleSelectParameter("time_period", "Time Period", time_period_options)
```

You can also specify custom fields as a dictionary using the `custom_fields` argument. This is an alternative example for "Year to Date":

```python
sr.SelectParameterOption("t0", "Year to Date", custom_fields={"date_modifier":ytd_date_modifier})
```

## 2. Access the Custom Fields at Runtime

You can get the date modifier(s) of the selected option(s) using the `get_selected` method for `SingleSelectParameter` (returns one value) or the `get_selected_list` method for `MultiSelectParameter` (returns a list of values). Both methods take a string argument called `field` (first positional argument) that lets you get the custom field(s) for the selected option(s). If the `field` is not specified, the methods return the `SelectParameterOption` object(s) instead.

Assume we have a date string to modify called `latest_date` (perhaps derived from another parameter). The following example applies the selected date_modifier of the "time_period" parameter to `latest_date`, which can be done purely in Python in the `context.py` file.

```python
time_period_param: sr.SingleSelectParameter = prms["time_period"]
date_modifier = time_period_param.get_selected("date_modifier")
start_date = date_modifier.modify(latest_date)
```

Alternatively, the selected `date_modifier` can also be accessed like this:

```python
date_modifier = time_period_param.get_selected().custom_fields["date_modifier"]
```

!!! Note
    If you prefer to not use `context.py`, variables can be set in the Jinja SQL views with `{% set var_name = ... %}`.

## Specifying Custom Columns for DataSourceParameter

Instead of using a `date_modifier`, suppose that the start date comes from a lookup table in the database instead. For instance, the lookup table can look like the following where the "start_date_column" gets updated regularly.

|time_period_id|time_period_label|start_date_column|
|:-------------|:----------------|:---------|
|t0|Year to Date|2023-01-01|
|t1|Year over Year|2022-07-02|

Instead of specifying a `SingleSelectParameter` for `time_period_param`, it can be a `DataSourceParameter` (using `custom_cols`) that converts itself into a `SingleSelectParameter` instead.

```python
time_period_ds = sr.SelectionDataSource("time_period_table", "time_period_id", "time_period_label", custom_cols={"start_date": "start_date_column"})
time_period_param = sr.DataSourceParameter(sr.SingleSelectParameter, "time_period", "Time Period", time_period_ds)
```

Then, in `context.py`, the selected start date can be accessed as follows.

```python
time_period_param: sr.SingleSelectParameter = prms["time_period"]
start_date = time_period_param.get_selected("start_date")
```
