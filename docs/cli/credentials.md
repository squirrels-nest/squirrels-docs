# Credential Management CLIs

There are three commands used to access and govern credentials:

1. `squirrels set-credential`
2. `squirrels get-all-credentials`
3. `squirrels delete-credential`

To set a credential, the user can use the `squirrels set-credential <key>` command to do so. This will prompt for the username and password for the connection you wish to use associated with the specified `key`. The `key` positional argument is the name of the connection to be specified in either the `squirrels.yaml` manifest file, or the `connections.py` file

Alternatively, you can also use `squirrels set-credential <key> [--values VALUES VALUES]`. Replace the two `VALUES` in the above command after the `--value` flag with the username and the password.

For example, if one would like to set the credentials named `example` then one would run the following:

```bash
$  squirrels set-credential example --values username 12345678 
```

On the other hand, `squirrels get-all-credentials` can be used to all the credentials saved, and `squirrels delete-credential <key>` can be used to a key previously defined by `set-credential`.
