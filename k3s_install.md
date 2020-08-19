# k3s

## Installing k3s

The fastest way to install k3s is by using the provided install script. To use this, run the below command.

```bash
# If not creating a kubeconfig file
curl -sfL https://get.k3s.io | sh -s - --write-kubeconfig-mode 644

# If creating a kubeconfig file
curl -sfL https://get.k3s.io | sh -
```

**Note:** `--write-kubeconfig-mode` allows us to not need to create a `~/.kube/config` file, but will be readable by anyone as a result. If you would like to preserve this file's secrecy, do the following:

```bash
# Create kubeconfig file
mkdir ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config

# Append kubeconfig to .bashrc file and export to current shell
echo "export KUBECONFIG=~/.kube/config" >> ~/.bashrc
export KUBECONFIG=~/.kube/config
```

Now we have a k3s instance running on our host machine.

## Uninstalling k3s

After you are finished with this guide, if you want to uninstall k3s, you can do so by running the following command.

```bash
/usr/local/bin/k3s-uninstall.sh
```
