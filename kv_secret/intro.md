# KV v2 Secret Engine

## Features

The KV v2 Secret engine allows for the creation of arbitrary key-value entries. By default, it is enabled and accessable at the `secrets/` path, but other paths can also be created for it. Each secret that is stored in it has an associated path, separated by forward slashes, for example `secrets/database/admin_auth`. Multiple secrets can be associated with a single path, for example, in the `secrets/database/admin_auth` path we can have a `username` key and a `password` key.

The secret engine also has version control, which can be used to access previous versions of the data.

There are two types of deletion: `delete` and `destroy`. The `delete` command will perform a soft delete, mark the version as deleted and populate a `deletion_time` timestamp. This doesn't remove the underlying version data from storage, and allows for undeletion by using `undelete`. On the other hand, `destroy` cannot be undeleted.

## [Further reading]([https://link](https://www.vaultproject.io/docs/secrets/kv/kv-v2))