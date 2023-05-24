# Squirrels test

This is the test command used to facilitate the debugging process for the user. For a given dataset, the test command creates outputs for parameters API response and rendered sql queries. 

The test command takes in a mandatory `dataset` argument, where the user can spcify the name of the dataset to be tested, as provided in `squirrels.yaml` file. When executed, the command will also create an `outputs` folder, where all the results will be stored. 

There are also several optional commands that can be specified between the `test` command, but before the name of the dataset. These are as follows:

```bash
options:
  -h, --help            show this help message and exit
  -c CFG, --cfg CFG     Configuration file for parameter selections. Path is relative to the dataset's folder
  -d DATA, --data DATA  Excel file with lookup data to avoid making a database connection. Path is relative to the dataset's folder
  -r, --runquery        Runs all database queries and final view, and produce the results as csv files
```