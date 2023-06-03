# Passing Parameter Selections

The method that the selected value(s) for a parameter is specified as a query parameter depends on the "widget_type" of the parameter. See table below for details:

|Widget Type|Example|Format Description|
|:----------|:------|:-----------------|
|SingleSelectParameter|a0|The name of the selected value|
|MultiSelectParameter|["x0","x2"]|The JSON list of the selected values|
|DateParameter|2022-01-01|The selected date in the format 'yyyy-MM-dd'|
|NumberParameter|6|The selected value as a number|
