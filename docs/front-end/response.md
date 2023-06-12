# Response Schema

The squirrels framework interacts with the front end by sending three types of JSON response: 

1. catalog
2. parameters
3. datasets

## Catalog
The catalog response is named squirrels0, and its API path is: `catalog: /squirrels0`, and includes the higher level information regarding the entire project, ie. major and minor version numbers, pahts to the datasets and parameters, etc. 

Its JSON schema is as follows:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "response_version": {
      "type": "integer",
      "description": "Version of response to control the logic for parsing"
    },
    "products": {
      "type": "array",
      "description": "The list of API products in the catalog",
      "items": {
        "type": "object",
        "properties": {
          "name": {
            "type": "string",
            "description": "Name of the product (may contain '/') specified in API path"
          },
          "versions": {
            "type": "array",
            "description": "List of major versions for the product and applicable datasets",
            "items": {
              "type": "object",
              "properties": {
                "major_version": {
                  "type": "integer",
                  "description": "Major version number for product, which is also specified in API path"
                },
                "datasets": {
                  "type": "array",
                  "description": "List of datasets applicable to major version",
                  "items": {
                    "type": "object",
                    "properties": {
                      "name": {
                        "type": "string",
                        "description": "Name of dataset specified in API path"
                      },
                      "label": {
                        "type": "string",
                        "description": "Display label dataset"
                      },
                      "parameters_path": {
                        "type": "string",
                        "description": "API path to dataset parameters. In the form '/<product>/v<major version>/<dataset>/parameters"
                      },
                      "result_path": {
                        "type": "string",
                        "description": "API path to dataset results. In the form '/<product>/v<major version>/<dataset>"
                      },
                      "first_minor_version": {
                        "type": "integer",
                        "description": "The first minor version (in current major version) where the dataset was included. Datasets can only be removed in a new major version"
                      }
                    },
                    "required": [
                      "name",
                      "label",
                      "parameters_path",
                      "result_path",
                      "first_minor_version"
                    ]
                  }
                }
              },
              "required": [
                "major_version",
                "datasets"
              ]
            }
          }
        },
        "required": [
          "name",
          "versions"
        ]
      }
    }
  },
  "required": [
    "response_version",
    "products"
  ]
}
```

Below is an example response from the weather example: 

```json
{"response_version":0,"products":[{"name":"seattle_weather","versions":[{"major_version":1,"latest_minor_version":0,"datasets":[{"name":"weather_by_time","label":"Weather by Time of Year","parameters_path":"/squirrels0/seattle-weather/v1/weather-by-time/parameters","result_path":"/squirrels0/seattle-weather/v1/weather-by-time","first_minor_version":0},{"name":"weather_trend","label":"Weather Trend","parameters_path":"/squirrels0/seattle-weather/v1/weather-trend/parameters","result_path":"/squirrels0/seattle-weather/v1/weather-trend","first_minor_version":0}]}]}]}
```

## Parameters

The parameters response provides the information regarding what parmeters is to be shown, and is named `parameters` Its API path is as follows:
`/squirrels0/<product>/v<major version>/<dataset>/parameters`

 Below is the JSON schema:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "response_version": {
      "type": "integer",
      "description": "Version of response to control the logic for parsing"
    },
    "parameters": {
      "type": "array",
      "description": "List of parameters application for the dataset",
      "items": {
        "type": "object",
        "properties": {
          "widget_type": {
            "type": "string",
            "description": "Type of the parameter (SingleSelectParameter, MultiSelectParameter, DateParameter, NumberParameter, etc.)"
          },
          "name": {
            "type": "string",
            "description": "Name of the parameter, matches the query parameter name for the dataset API"
          },
          "label": {
            "type": "string",
            "description": "Display label for the parameter"
          },
          "options": {
            "type": "array",
            "description": "List of selection options for SingleSelectParameter and MultiSelectParameter",
            "items": {
              "type": "object",
              "properties": {
                "id": {
                  "type": "string",
                  "description": "ID for the option, to be used to pass selected options in query parameters"
                },
                "label": {
                  "type": "string",
                  "description": "Display label for the selection option"
                }
              },
              "required": [
                "id",
                "label"
              ]
            }
          },
          "trigger_refresh": {
            "type": "boolean",
            "description": "Whether new parameter selections should trigger another parameters API call, which can be used to change the options of children parameters. Only applicable to (and required for) SingleSelectParameter and MultiSelectParameter"
          },
          "selected_id": {
            "type": "string",
            "description": "ID of the selected option. Only applicable to (and required for) SingleSelectParameter"
          },
          "selected_ids": {
            "type": "array",
            "description": "ID of the selected option(s). Only applicable to (and required for) MultiSelectParameter",
            "items": {
              "type": "string"
            }
          },
          "include_all": {
            "type": "boolean",
            "description": "Flag for whether no selections is same as all selected. Only applicable to (and required for) MultiSelectParameter"
          },
          "order_matters": {
            "type": "boolean",
            "description": "Flag for whether the order of the selection matters. Only applicable to (and required for) MultiSelectParameter"
          },
          "selected_date": {
            "type": "string",
            "description": "Selected date in yyyy-mm-dd format. Only applicable to (and required for) DateParameter"
          },
          "min_value": {
            "type": "string",
            "pattern": "/^\d*\.?\d*$/",
            "description": "Minimum number for a number slider. Only applicable to (and required for) NumberParameter"
          },
          "max_value": {
            "type": "string",
            "pattern": "/^\d*\.?\d*$/",
            "description": "Maximum number for a number slider. Only applicable to (and required for) NumberParameter"
          },
          "increment": {
            "type": "string",
            "pattern": "/^\d*\.?\d*$/",
            "description": "Steps / intervals for a number slider. Only applicable to (and required for) NumberParameter"
          },
          "selected_value": {
            "type": "string",
            "pattern": "/^\d*\.?\d*$/",
            "description": "Selected number on a number slider. Only applicable to (and required for) NumberParameter"
          }
        },
        "required": [
          "widget_type",
          "name",
          "label"
        ]
      }
    }
  },
  "required": [
    "response_version",
    "parameters"
  ]
}
```

Here's an example response from the weathers example: 

```json
{"response_version":0,"parameters":[{"widget_type":"SingleSelectParameter","name":"group_by","label":"Group By","options":[{"id":"0","label":"Year"},{"id":"1","label":"Quarter"},{"id":"2","label":"Month"},{"id":"3","label":"Day of Year"},{"id":"4","label":"Condition"}],"trigger_refresh":false,"selected_id":"0"}]}
```

## Dataset

After the parameters have been selected, the dataset reponse is used to send over the actual data to be displayed/used in the form of a json file. Its API path is: `dataset: /squirrels0/<product>/v<major version>/<dataset>` and has the same name as the dataset. 

Below is its JSON schema:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "response_version": {
      "type": "integer",
      "description": "Version of response to control the logic for parsing"
    },
    "schema": {
      "type": "object",
      "properties": {
        "fields": {
          "type": "array",
          "description": "The ordering of the columns and their data types",
          "items": {
            "type": "object",
            "properties": {
              "name": {
                "type": "string",
                "description": "Name of the column"
              },
              "type": {
                "type": "string",
                "description": "Data type of the column (string, number, etc.)"
              }
            },
            "required": [
              "name",
              "type"
            ]
          }
        },
        "dimensions": {
          "type": "array",
          "description": "The dimension columns in specific order",
          "items": {
            "type": "string",
            "description": "Name of the column"
          }
        }
      },
      "required": [
        "fields",
        "dimensions"
      ]
    },
    "data": {
      "type": "array",
      "description": "List of rows represented as JSON objects",
      "items": {
        "type": "object"
      }
    }
  },
  "required": [
    "response_version",
    "schema",
    "data"
  ]
}
```

Here's an example response from the weather example: 

```json
{"response_version":0,"schema":{"fields":[{"name":"year","type":"integer"},{"name":"temperature_high_C","type":"number"},{"name":"temperature_low_C","type":"number"},{"name":"precipitation_inches","type":"number"},{"name":"wind_mph","type":"number"}],"dimensions":[]},"data":[{"year":2012,"temperature_high_C":15.2767759563,"temperature_low_C":7.2896174863,"precipitation_inches":3.349726776,"wind_mph":3.4008196721},{"year":2013,"temperature_high_C":16.0589041096,"temperature_low_C":8.1539726027,"precipitation_inches":2.2684931507,"wind_mph":3.015890411},{"year":2014,"temperature_high_C":16.995890411,"temperature_low_C":8.6624657534,"precipitation_inches":3.3775342466,"wind_mph":3.3876712329},{"year":2015,"temperature_high_C":17.4279452055,"temperature_low_C":8.8356164384,"precipitation_inches":3.1210958904,"wind_mph":3.1597260274}]}
```

