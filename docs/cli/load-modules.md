# The load-modules CLI

The `squirrels load-modules` command loads all the modules specified in the `modules` section of `squirrels.yaml`. Each module can be specified as a git repository in the format `git_url@tag` or `name=git_url@tag` for a different repo name than the default name.

For instance, suppose we have the following in the `squirrels.yaml` file.

```yaml
modules:
    - https://github.com/user/repo1.git@v1.0
    - second_repo=https://github.com/user/repo2.git@v2.0
```

Running `squirrels load-modules` in the project directory will create a new `modules` folder with subdirectories `repo1` and `second_repo`. 

Inside `repo1` is the contents of the `https://github.com/user/repo1.git` repo at the `v1.0` tag, and inside `second_repo` is the contents of the `https://github.com/user/repo2.git` repo at the `v2.0` tag.

Then, in any python file, you can use the new modules by a simple import (for instance, `from modules.repo1 import ...`). Note that the `modules` folder is intentionally included in the `.gitignore` file.
