# Squirrels run

The `squirrels run` command is the main command to actually execute all the files in the squirrels project in the directory. It will connect to the databases using the connections provided in either `squirrels.yaml` or the `connections.py` files, execute the datasets, and host the results using Uvicorn running on the specified port, where the user can access the executed datasets using a browser. The run command provides the following options:

```bash

usage: squirrels run [-h] [--no-cache] [--debug] [--host HOST] [--port PORT]

options:
  -h, --help   show this help message and exit
  --no-cache   Do not cache any api results
  --debug      In debug mode, all "hidden parameters" show in the parameters response
  --host HOST
  --port PORT

```

The default port is http://127.0.0.1:8000, you can press Ctrl + C to shut it down.