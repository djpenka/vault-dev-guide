# Installing a local Jenkins instance

First we will add the repository with Jenkins in it, and install it using Helm to our already running k3s cluster.

```bash
helm repo add stable https://kubernetes-charts.storage.googleapis.com

helm install jenkins stable/jenkins
```

Now we set up port forwarding. Get the name of your jenkins pods using `kubectl get pods` and then set port forwarding for 8080 using the below command, replacing `$POD_NAME` with the name of your pod.

```bash
kubetl port-forward $POD_NAME 8080:8080
```

Get the admin password that was automatically created by the Jenkins Helm chart

```bash
printf $(kubectl get secret jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo
```

Navigate to `localhost:8080` and login with username `admin` and your admin password from above.