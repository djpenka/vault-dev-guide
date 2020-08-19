# Creating a KV secret using the Vault Web UI

First, login to your vault instance, then navigate to the Secrets Engine Menu by clicking on `Secrets` in the top bar.

![secrets menu](../static/kv_secret/web_ui/secrets_main.png)

Click on the `secret/` path to see the secrets under the `secret/` backend, which is running using the `kv_v2` engine.

![secrets main](../static/kv_secret/web_ui/secret_menu.png)

Click on `Create secret`, and then fill in the data for your secrets. You can create a subpath to organize your secret under by separating tiers with a forward slash, as shown below. Under your subpath, you can create multiple secrets.

![created secret](../static/kv_secret/web_ui/created_secret.png)

After you create your secret, the final screen should look similar to this.

![final secret](../static/kv_secret/web_ui/final_secret.png)
