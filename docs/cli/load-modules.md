# Loading modules

The `squirrels load-modules` loads all the modules spcified in the `squirrels.yaml` into squirrels project in the directory where the command is run. We currently only support modules in the format of git repositories. To add a python module to a squirrels project, add the git repository URL to the list variable named `modules` in the manifest (`squirrels.yaml`) file.