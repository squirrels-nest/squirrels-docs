# Test the REST APIs

In the `weather_by_time` datasets folder, go into `context.py` and replace the body of the `main` function to:

```python
return {}
```

We will discuss the `context.py` file later in the tutorial, but for now, we are clearing it such that we can activate the REST APIs without errors.

To activate the API server, simply run:

```bash
squirrels run --debug
```

!!! Note
    The `--debug` argument is optional here. This is specified to be able to test with hidden parameters, though no hidden parameters were set up for this tutorial. See `squirrels run --help` or the "[run command line]" page for more details.

Then, in a web browser, go to `http://localhost:8000/` to interact with the dataset APIs you've just created using the Squirrels UI!

Remember to shut down the API by pressing Ctrl + C before proceeding to the next step.

## Test the Rendered SQL Queries

In practice, you may wish to review what the rendered SQL queries look like (for some set of parameter selections) before actually running the queries.

To do this for the `weather_by_time` dataset (using the default parameter selections), run:

```bash
squirrels test weather_by_time
```

This creates the folder path `outputs/weather_by_time` with the generated SQL queries (without actually running them). Take a look at the files in the folder to see what got generated. To run the generated queries as well, and write results as csv files, use the following to command.

```bash
squirrels test weather_by_time --runquery
```

We can also test on non-default parameter selections. In the dataset folder, change the contents of the `selections.cfg` file to: 

```config
[parameters]
group_by = 2
```

When we define multiple parameters, the order of the parameters specified here must be the same as the order as defined in the `parameters.py` file (especially if cascading parameters exist). The "2" is the ID of the "Month" option for the `group_by` parameter. Run the following to render the SQL queries with `selections.cfg`:

```bash
squirrels test weather_by_time --cfg selections.cfg
```

See `squirrels test --help` or the page for the "[test command line]" for more details. See the next tutorial page to [Use the Context File](context.md).


[run command line]: ../cli/run.md
[test command line]: ../cli/test.md
