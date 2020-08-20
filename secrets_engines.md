# Secrets Engines

A secret is stored or otherwise accessed in a Secrets Engine. Some secrets engines store and read data, such as the [kv-v2](https://www.vaultproject.io/docs/secrets/kv/kv-v2) secrets engine, which is accessed from the path `secret/` (though different locations can be configured). Other engines connect to services to dynamically generate credentials, such as the [AWS Secrets Engine](https://www.vaultproject.io/docs/secrets/aws), which generates AWS access credentials dynamically based on IAM policies. Other secrets engines provide encryption as a service, totp generation, certificates, and much more. You can have multiple instances of the same engine type running at different paths, with their own unique configuration if needed. More information on all the engines can be found in the [documentation](https://www.vaultproject.io/docs/secrets)

## Using the kv-v2 engine

The kv-v2 engine allows for storing key-value pairs in secret files. Like in a filesystem on a computer, these files can be stored in different paths within the secrets engine. For instance, we can have the file `my-secrets` stored on the path `database/auth` with two key value pairs inside it: `user=admin` and `password=my_pass`. In addition to this, the kv-v2 engine supports versioning, and delete/undelete (though data can also be permanently "destroyed").
