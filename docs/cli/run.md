# The run CLI

The `squirrels run` command is the main command that actually activates the APIs for each of the datasets, including their parameters and results. When run locally, those will be hosted on the specified host and port.  

The run command provides the following options (this can be found from `squirrels run -h`):

```bash
usage: squirrels run [-h] [--no-cache] [--debug] [--host HOST] [--port PORT]

options:
  -h, --help   show this help message and exit
  --no-cache   Do not cache any api results
  --debug      In debug mode, all "hidden parameters" show in the parameters response
  --host HOST
  --port PORT
```

The default host is `127.0.0.1` and the default port is `8000`. While running, you can be access [http://127.0.0.1:8000]() or [http://localhost:8000]() from the browser to interact with the squirrels UI. 

Assume you have a sample product named "sample_product", a sample dataset named "sample_dataset", and the major version is set to 1. You can append the following paths to [http://localhost:8000]() to retrieve various JSON results:

|URL Path|Description|
|:-------|:----------|
|/squirrels0|The catalog API|
|/squirrels0/sample_product/v1/sample_dataset/parameters|The parameters API|
|/squirrels0/sample_product/v1/sample_dataset|The dataset API|

You can press Ctrl + C to shut down the API server.
