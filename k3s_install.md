# k3s

## Installing k3s

The fastest way to install k3s is by using the provided install script. To use this, run the below command.

```bash
curl -sfL https://get.k3s.io | sh -
```

Now we need to move the default kubeconfig file to our own `~/.kube/config` file. If you already have a `~/.kube/config` file, then integrate the file at `/etc/rancher/k3s/k3s.yaml` into your own kubeconfig file. Otherwise, run the below commands.

```bash
# Create kubeconfig file and give ownership to current user
mkdir ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown -R $USER ~/.kube

# Append kubeconfig to .bashrc file and export to current shell
echo "export KUBECONFIG=~/.kube/config" >> ~/.bashrc
export KUBECONFIG=~/.kube/config
```

Now we have a k3s instance running on our host machine.

## Installing helm

Installing helm should be easy, just run the following command. If there is an error, ensure that you have created `~/.kube/config`, have set `KUBECONFIG=~/.kube/config`, and that the current user owns `~/.kube/config`.

```bash
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```

## Uninstalling k3s

After you are finished with this guide, if you want to uninstall k3s, you can do so by running the following command.

```bash
/usr/local/bin/k3s-uninstall.sh
```
