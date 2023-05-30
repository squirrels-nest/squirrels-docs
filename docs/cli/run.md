# Squirrels run

The `squirrels run` command is the main command that actually activates the APIs for each of the datasets, including their parameters and results. When run locally, those will be hosted on the specified host and port.  

The run command provides the following options:

```bash

usage: squirrels run [-h] [--no-cache] [--debug] [--host HOST] [--port PORT]

options:
  -h, --help   show this help message and exit
  --no-cache   Do not cache any api results
  --debug      In debug mode, all "hidden parameters" show in the parameters response
  --host HOST
  --port PORT

```

The default port is http://127.0.0.1:8000, and can be accessed through the browser. You can press Ctrl + C to shut it down.

