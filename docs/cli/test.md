# The test CLI

The `squirrels test <dataset>` command is used to facilitate the debugging process for the user. For a given dataset, the test command creates outputs for parameters API response and rendered sql queries. 

The test command takes in a mandatory `dataset` argument, where the user can spcify the name of the dataset to be tested, as provided in `squirrels.yaml` file. When executed, the command will also create an `outputs` folder, where all the results will be stored. 

All the details on command line arguments be found using `squirrels test -h`. The result is as follows:

```bash
usage: squirrels test [-h] [-c CFG] [-d DATA] [-r] dataset

positional arguments:
  dataset               Name of dataset (provided in squirrels.yaml) to test. Results are written in an "outputs" folder

options:
  -h, --help            show this help message and exit
  -c CFG, --cfg CFG     Configuration file for parameter selections. Path is relative to the dataset's folder
  -d DATA, --data DATA  Excel file with lookup data to avoid making a database connection. Path is relative to the dataset's folder
  -r, --runquery        Runs all database queries and final view, and produce the results as csv files
```

## The selections.cfg file

It is good practice to create a `selections.cfg` file for your datasets to use with the `test` command. You can create a sample `selections.cfg` file by running `squirrels init --selections-cfg`.

The `selections.cfg` file should specify all the parameter names and selected values under the "[parameters]" header. The parameter names (left of equal signs) should be specified in the same order specified in `parameters.py` (which avoids ambiguity especially if parameter dependencies are set). The selected values (right of equal signs) should follow the same format that the front-end uses to specify selected values for various parameter types, described [here](../front-end/selections.md).

Assuming that the `selections.cfg` file is created in a dataset folder called `example`, simply run `squirrels test example --cfg selections.cfg` to render the SQL queries with the specified parameter selections.

## The lu_data.xlsx file

In the absence of an internet connection, the `test` command would not work if there are datasource parameters linked to a dimension table/query from an external database. Thus, it is also a good practice to use an Excel file to contain all the dimension table data (or a sample of it).

In the Excel file, the sheet name must be the name of the datasource parameter, and the residing table (starting at cell A1) must have the same columns as the linked dimension table/query. Assuming the Excel file is named `lu_data.xlsx` in a dataset folder called `example`, simply run `squirrels test example --data lu_data.xlsx` to render the SQL queries without having to use an internet connection!
