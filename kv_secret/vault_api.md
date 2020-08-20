# Creating a KV Secret with the Vault API

For every request to the API, we will need a header `X-Vault-Token: ...`. Because we are working on a dev instance, our token will be `root`. If you work on a production vault instance, you may need to authenticate with a different method. If you need to do that, you can find details on the authentication login method in [the official vault API authentication docs](https://www.vaultproject.io/api-docs/auth).

Documentation for each of the queries made here can be found in the [vault kv-v2 api docs](https://www.vaultproject.io/api-docs/secret/kv/kv-v2)

## Add a file

First, add a file named `my-secret` under the path `secret/database/`. In this path, `secret/` refers to the secret engine we are using, and `database/` is a subpath in that engine, where our file `my-secret` will be stored.

```bash
curl \
    --header "X-Vault-Token: root" \
    --request POST \
    --data '{"data":{"username":"my_username","password":"my_password"}}' \
    http://localhost:8200/v1/secret/data/database/my-secret | jq
```

The breakdown of this request URL is `${domain}:8200/v1/${secret_engine_path}/data/${path_to_file}/${file}`. This is similar with all other requests in the kv-v2 engine api.

## List Secrets in a path

Now we can see our file listed under the path `database/` in the `secret/` engine using the following command to list all secrets in that path.

```bash
curl \
    --header "X-Vault-Token: root" \
    --request LIST \
    http://localhost:8200/v1/secret/metadata/database/ | jq
```

## Get a secrets file

If version is not set, it defaults to the latest version.

```bash
curl \
    --header "X-Vault-Token: root" \
    http://localhost:8200/v1/secret/data/database/my-secret?version=1 | jq
```
