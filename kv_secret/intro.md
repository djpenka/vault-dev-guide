# KV v2 Secret Engine

## Features

The kv-v2 Secret engine allows for the creation of arbitrary key-value entries stored in files. By default, it is enabled and accessable at the `secrets/` endpoint. Each secret file that is stored in it has an associated path such as `database/admin_auth`, and can store multiple secrets. For example, in the `database/admin_auth` path we can have a file named `my_auth` with two entries: `username=admin` and `password=my_pass`.

The secret engine also has version control, which can be used to access previous versions of the data.

There are two types of deletion: `delete` and `destroy`. The `delete` command will perform a soft delete, mark the version as deleted and populate a `deletion_time` timestamp. This doesn't remove the underlying version data from storage, and allows for undeletion by using `undelete`. On the other hand, `destroy` cannot be undeleted.

## [Further reading](https://www.vaultproject.io/docs/secrets/kv/kv-v2)