# Modifying Dates Using Squirrel's dateutils

The squirrels library has a submodule called dateutils that can be used to transform date variables in Python with ease. To import the dateutils module, simply do:

```python
from squirrels import dateutils as du
```

The examples below demonstrates how to modify a given date by number of time periods and/or get some day of a calendar cycle. The format of the date can be datetime objects, string dates, or unix timestamps (as float). Assume we have a datetime object variable called `date_obj`.

## Offset by Time Period

The following classes can be used to offset a datetime object by some time period.

- OffsetYears
- OffsetMonths
- OffsetWeeks
- OffsetDays

Each of their constructors take an argument `offset` for the number of time periods to offset by (can be positive or negative), and each class contains a modify method to modify an input date.

To add 3 weeks to `date_obj`, the code you write can look something like this:

```python
new_date = OffsetWeeks(3).modify(date_obj)
```

## Get N-th Day of Calendar Cycle

The following classes can be used to get a certain day of some calendar cycle from some datetime object.

- DayIdxOfMonthsCycle
- DayIdxOfYear
- DayIdxOfQuarter
- DayIdxOfMonth
- DayIdxOfWeek

Each of their constuctors take an parameter `idx` to specify the day number of cycle. Positive numbers like 1 and 2 represent the first and second day, while negative numbers like -1 and -2 represent the last and second last day. 0 cannot be used.

For DayIdxOfMonthsCycle, the length of the cycle in months can be specified with the parameter `num_months_in_cycle`. This is unlike DayIdxOfMonth where the length of the cycle is always one month. DayIdxOfYear, DayIdxOfQuarter, and DayIdxOfMonth are equivalent to DayIdxOfMonthsCycle when the `num_months_in_cycle` values 12, 3, and 1 respectively.

For DayIdxOfWeek, the first day of the week (using the DayOfWeek enum which contains values Monday, Tuesday, etc. until Sunday) can be specified using the `first_day_of_week` parameter. The first month of the year can be specified (using the Month enum which contains values January, February, etc. until December) with the parameter `first_month_of_X` where X is "cycle", "year", and "quarter" for DayIdxOfMonthsCycle, DayIdxOfYear, and DayIdxOfQuarter respectively.

Each of these classes contain a modify method to modify an input date as well.

Here are some problems and solutions in code using these classes.

|Problem|Solution|
|:------|:-------|
|Get the same or prior Friday|`DayIdxOfWeek(idx=1, first_day_of_week=DayOfWeek.Friday).modify(date_obj)`|
|Get the same or next Friday|`DayIdxOfWeek(idx=-1, first_day_of_week=DayOfWeek.Saturday).modify(date_obj)`|
|If Wednesday to Friday, round up to Friday. <br>Otherwise round down to Friday|`DayIdxOfWeek(idx=3, first_day_of_week=DayOfWeek.Wednesday).modify(date_obj)`|
|Get the third last day of month|`DayIdxOfMonth(idx=-3).modify(date_obj)`|
|Suppose a "Third Year" occurs every 4 months from <br>February 1st. Get the beginning of current Third Year|`DayIdxOfMonthsCycle(idx=1, num_months_in_cycle=4, first_month_of_cycle=Month.February).modify(date_obj)`|

## Date Modification Pipeline

A class called `DateModPipeline` lets you stitch together multiple instances of the date modification classes above into a single date modification class. Simply specify a sequence of date modification class instances in the constructor. Using this pipeline approach allows one to make almost any date transformation possible.

Here are some more examples of problems and solutions.

|Problem|Solution|
|:------|:-------|
|Get the next Friday|`DateModPipeline([DayIdxOfWeek(1, DayOfWeek.Friday), OffsetWeeks(1)]).modify(date_obj)`|
|Get the prior Friday|`DateModPipeline([DayIdxOfWeek(-1, DayOfWeek.Saturday), OffsetWeeks(-1)]).modify(date_obj)`|
|Get the second Friday of the current quarter. <br>First month of quarter is January|`DateModPipeline([DayIdxOfQuarter(1), DayIdxOfWeek(-1, DayOfWeek.Saturday), OffsetWeeks(1)]).modify(date_obj)`|

In addition to the `modify` method, this class also lets you get a list of date objects with the method `get_date_list`. It takes an input start date and a step (as an Offset* date modifier), modifies the start date to get the end date, and returns the dates from start to end by step. Below is an example of getting every Friday going back from June 15th 2023 to beginning of Quarter.

```python
modifier = DateModPipeline([DayIdxOfQuarter(1), DayIdxOfWeek(-1, DayOfWeek.Saturday)])
date_list = modifier.get_date_list(datetime(2023, 6, 15), OffsetWeeks(-1))
```

## Modifying Date Strings or Timestamps

More often then not, the dates that you're working with are date formatted strings or timestamps as floats. Although it should be easy enough to convert to datetime object and back, the dateutils module also provides `DateStringModifier` and `TimestampModifier` classes to simplify your code. Just like `DateModPipeline`, both these classes have a constructor that takes a sequence of date modification class instances, a `modify` method, and a `get_date_list` method.

For `DateStringModifier`, the constructor also takes an optional `date_format` argument for the date format of the outputs. The methods `modify` and `get_date_list` take an input date as string, and an optional `input_format` argument for the date format of the input date.

For `TimestampModifier`, the input date for methods `modify` and `get_date_list` are floats representing the UNIX timestamp of the date. The output dates are also in the same format.
