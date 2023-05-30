# Credentials

There are three commands used to access and govern credentials:
1. `squirrels set-credential`
2. `squirrels get-all-credentials`
3. `squirrels delete-credential`


To set a credential, the user can use the `squirrels set-credential [--values VALUES VALUES] key` command to do so. Replace the two `VALUES` in the above command after the `--value` flag with the username and the password for the connection you wish to use respectively, and the `key` positional argument with the name of the connection as specified in either the `squirrels.yaml` manifest file, or the `connections.py` file.

On the other hand, `get-all-credentials` and `delete-credential` will return all the credentials and delete them respectively.

