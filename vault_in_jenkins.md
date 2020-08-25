# Setting up Vault authentication with Jenkins

Note: All the links on this page assume that you have Jenkins running on `localhost:8080` as outlined in the Jenkins install instructions.

First we will add the Hashicorp Vault plugin to Jenkins through the [plugin manager](http://localhost:8080/pluginManager/).

In the interests of time, we will authenticate with the default `root` token provided by the dev instance of Vault. In a production build, you can use the authentication method of your choice to allow jenkins to access the Vault instance - different methods are outlined [in the vault authentication documentation](https://www.vaultproject.io/docs/auth)

We will add the `root` token to the jenkins credential store (at the global scope), which can be done [here](http://localhost:8080/credentials/store/system/domain/_/newCredentials). The configuration will be as such:

```
Kind: Vault Token Credential
Scope: Global
Token: root
ID: jenkins-vault-token
```

The Vault plugin has 2 options to access secrets. The first technique uses a function called `withVault` which gives you access to the requested secrets when passed appropriate authentication credentials. This will be demonstrated below. There is another function called `withCredentials` which doesn't pull secrets, but which will set the `VAULT_ADDR` and `VAULT_TOKEN` variables, which can be useful to allow Terraform to work with Vault. However, this is beyond the scope of this guide, and will not be covered here.

## withVault Demo

First we will create a kv-v2 secret at the default `secret` path.

```bash
# Remote into the vault-0 pod
kubectl exec -it vault-0 -- sh

# Create a secret at secret/database/auth
vault kv put secret/database/auth username="my-user" password="my-password"

# Verify that the secret exists
vault kv get secret/database/auth

# Exit the pod
exit
```

Now we will create a new pipeline in Jenkins. From the [main menu](http://localhost:8080) click "New Item" and create a new "Pipeline" with the name `vault-test`. Copy the pipeline from `./jenkins/Jenkinsfile` into the "Pipeline" section. Because we are using Jenkins in the same kubernetes cluster as our Vault, we can use the `vault` service as our URL, as seen here: `vaultUrl: 'http://vault:8200'`. However, if they were not configured like this, we would need to use the actual URL of our Vault instance. Finally, click Save.

Click Build Now, and then click into the newly run build and click "Console Output". Your output should look something like this:

```
[Pipeline] {
[Pipeline] withVault
Retrieving secret: secret/database/auth
[Pipeline] {
[Pipeline] sh
+ echo ****
****
[Pipeline] sh
+ expr length ****
7
[Pipeline] sh
+ echo ****
****
[Pipeline] sh
+ expr length ****
11
[Pipeline] }
[Pipeline] // withVault
[Pipeline] }
```

As we can see, the secrets themselves are censored in the logs, however, we can see that we have the right secrets because the lengths of these strings match up with the length of the keys that we stored earlier.