# Injecting Secrets with Vault Helm Sidecar

This guide assumes that you have installed vault from the helm chart onto a kubernetes cluster, with access to the pod it is running on. For instructions on this, follow parts 1 and 2 of the Table of Contents in this guide.

This guide is heavily based on the official HashiCorp Learn guide posted [here](https://learn.hashicorp.com/tutorials/vault/kubernetes-sidecar) with a focus on the developer experience and some omitted sections. If looking for further learning, look at this article or others at <https://learn.hashicorp.com/tutorials/vault/>

## Set a secret in Vault

Exec into the vault pod.

```bash
kubectl exec -it vault-0 -- /bin/sh
```

Enable a new kv-v2 secrets engine in the path `internal/`, and put a secret at the path `internal/database/` in the file `config`.

```bash
vault secrets enable -path=internal kv-v2
vault kv put internal/database/config username="db-readonly-username" password="db-secret-password"
```

Verify that the secret is defined at the path `internal/database/config`

```bash
vault kv get internal/database/config
```

## Configure Kubernetes Authentication

The [kubernetes authentication method](https://www.vaultproject.io/docs/auth/kubernetes.html) enables clients to authenticate with a Kubernetes Service Account Token. This token is provided to each pod when it is created. The following commands assume that we are still in an interactive shell inside the `vault-0` pod.

Enable the Kubernetes authentication method

```bash
vault auth enable kubernetes
```

Vault accepts this service token from any client within the Kubernetes cluster. During authentication, Vault verifies that the service account token is valid by querying a configured Kubernetes endpoint.

Configure the Kubernetes authentication method to use the service account token, the location of the Kubernetes host, and its certificate.

The `token_reviewer_jwt` and `kubernetes_ca_cert` are mounted to the container by Kubernetes when it is created. The environment variable `KUBERNETES_PORT_443_TCP_ADDR` is defined and references the internal network address of the Kubernetes host.

```bash
vault write auth/kubernetes/config \
    token_reviewer_jwt="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
    kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443" \
    kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
```

We created a secret at `internal/database/config`. To read this data, we need to grant read capability for the path `internal/data/database/config` (this is the path used by the api to read this data, read more [here](kv_secret/vault_api.md)). This is done by setting a [policy](https://www.vaultproject.io/docs/concepts/policies.html), which defines a set of capabilities.

Write out a policy named `internal-app` that enables `read` capability for secrets at path `internal/data/database/config`

```bash
vault policy write internal-app - <<EOF
path "internal/data/database/config" {
  capabilities = ["read"]
}
EOF
```

Create a Kubernetes authentication role also named `internal-app`

```bash
vault write auth/kubernetes/role/internal-app \
    bound_service_account_names=internal-app \
    bound_service_account_namespaces=default \
    policies=internal-app \
    ttl=24h
```

This role connects the Kubernetes service account `internal-app` and namespace `default` with the Vault policy `internal-app`. The tokens returned after authentication are valid for 24 hours.

Finally, exit from the `vault-0` pod.

```bash
exit
```

## Define a Kubernetes service account

Though we have defined a Kubernetes authentication role for Vault, we still need to define the service account named `internal-app` in Kubernetes itself, because it doesn't yet exist.

Verify that the Kubernetes service account named `internal-app` does not exist.

```bash
kubectl get serviceaccounts
```

cd into the `sidecar-files` directory and look at the file `service-account-internal-app.yml`. This will create the account with the name `internal-app`.

```bash
cd sidecar-files
cat service-account-internal-app.yml
```

Apply the service account definition to create it.

```bash
kubectl apply --filename service-account-internal-app.yml
```

Verify that the service account has been created.

```bash
kubectl get serviceaccounts
```

Notice that the name of the service account is the same as the name assigned to the `bound_service_account_names` field when the `internal-app` role in Vault was created.

## Launch an application

There is a simple application provided by Hashicorp Vault that we will be using to test our secrets agent. The deployment for this can be found in this repository, under `sidecar-files`. These commands will assume that you are in the `sidecar-files` directory.

Let's look at the deployment for the `orgchart` application.

```bash
cat deployment-orgchart.yml
```

The `spec.template.spec.serviceAccountName` states that the service account `internal-app` will run this container.

Now we will apply the deployment from `deployment-orgchart.yml`

```bash
kubectl apply --filename deployment-orgchart.yml

kubectl get pods
```

We can now see the orgchart deployment running in our namespace.

The Vault-Agent injector looks for deployments that define specific annotations. None of these annotations exist within the current deployment. This means that no secrets are present on the orgchart container within the orgchart pod.

We can verify that by going into the pod and checking ourselves.

```bash
kubectl exec \
$(kubectl get pod -l app=orgchart -o jsonpath="{.items[0].metadata.name}") --container orgchart -- ls /vault/secrets
```

This will state that there is no such file or directory named `/vault/secrets`

## Inject secrets into the pod

Our deployment is running the pod with the `internal-app` Kubernetes service account in the default namespace. The Vault Agent Injector only modifies a deployment if it contains a specific set of annotations. Since we already have our deployment running, we need to patch our deployment to get it working.

Take a look at the deployment patch `patch-inject-secrets.yml`

```bash
cat patch-inject-secrets.yml
```

The annotations in this patch define a partial structure of the deployment schema. They are all prefixed with `vault.hashicorp.com`

- `agent-inject` enables the Vault Agent Injector service
- `role` is the Vault Kubernetes authentication role
- `agent-inject-secret-FILEPATH` prefixes the path of the file `database-config.txt` written to the `vault/secrets` directory. The value is the path to the secret defined in Vault

Detailed info on annotations can be found [here](https://www.vaultproject.io/docs/platform/k8s/injector#annotations)

Patch the `orgchart` deployment defined in `patch-inject-secrets.yml`

```bash
kubectl patch deployment orgchart --patch "$(cat patch-inject-secrets.yml)"
```

A new `orgchart` pod starts alongside the existing pod. When it is ready the original terminates and removes itself from the list of active pods.

Get all the pods within the default namespace.

```bash
kubectl get pods
```

Wait until the re-deployed `orgchart` pod reports that it is `Running` and `ready (2/2)`.

Let's take a look at the logs of the `vault-agent` container that is now running in the new `orgchart` pod.

```bash
kubectl logs \
    $(kubectl get pod -l app=orgchart -o jsonpath="{.items[0].metadata.name}") \
    --container vault-agent
```

Vault Agent will manage token lifecycle and secret retrieval. As configured, the secret is rendered in the `orgchart` container at the path `/vault/secrets/database-config.txt`.

Finally, we can see the secret written to the `orgchart` container.

```bash
kubectl exec \
    $(kubectl get pod -l app=orgchart -o jsonpath="{.items[0].metadata.name}") \
    --container orgchart -- cat /vault/secrets/database-config.txt
```

## Applying a template to the injected secrets

Looking at the data output, we will probably want to format it so that it is easily readable by our application. We can accomplish this with a template that is added to our annotations for the pod in Kubernetes.

There is already a file included that contains a template definition. Display the annotations file that contains a template definition.

```bash
cat patch-inject-secrets-as-template.yml
```

This contains two new annotations:

- `agent-inject-status` set to `update` informs the injector to reinject these values
- `agent-inject-template-FILEPATH` prefixes the file path. The value defines the [Vault Agent template](https://www.vaultproject.io/docs/agent/template/index.html) to apply to the secret's data

The template formats the username and password as a PostgreSQL connection string.

Apply the updated annotations.

```bash
kubectl patch deployment orgchart --patch "$(cat patch-inject-secrets-as-template.yml)"
```

Now check the pods to see your new pod initializing

```bash
kubectl get pods
```

Wait until the re-deployed `orgchart` pod reports that it is `Running` and `ready (2/2)`

Finally, display the secret written to the `orgchart` container in the `orgchart` pod

```bash
kubectl exec \
    $(kubectl get pod -l app=orgchart -o jsonpath="{.items[0].metadata.name}") \
    -c orgchart -- cat /vault/secrets/database-config.txt
```

## Secrets are bound to the service account

Pods run with a Kubernetes service account that does not have a corresponding Vault Kubernetes authentication role are **NOT** able to access any Vault secrets.

If a pod is taking a long time to initialize, it may be because the service account that it is using is not authorized. In order to check this, we can display the logs of the `vault-agent-init` container of the pod we are checking. The command for this is below.

```bash
kubectl logs -f your-pod -c vault-agent-init
```

If the error looks similar to this, your service account is not authorized.

```
[INFO]  auth.handler: authenticating
[ERROR] auth.handler: error authenticating: error="Error making API request.

URL: PUT http://vault.default.svc:8200/v1/auth/kubernetes/login
Code: 500. Errors:

* service account name not authorized" backoff=1.562132589
```

## Secrets are bound to the namespace

Pods run in a namespace other than the ones defined in the Vault Kubernetes authentication role are NOT able to access the secrets defined at that path. Using the same method as above, we can see an error message in the pod if we are attempting to access a secret from an unauthorized namespace.

```
[INFO]  auth.handler: authenticating
[ERROR] auth.handler: error authenticating: error="Error making API request.

URL: PUT http://vault.default.svc:8200/v1/auth/kubernetes/login
Code: 500. Errors:

* namespace not authorized" backoff=1.9882590740000001
```