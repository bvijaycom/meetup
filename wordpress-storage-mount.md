# AZURE Kubernetes Automation

# Fixing storage mount issues

# helm installation

wget https://get.helm.sh/helm-v3.6.3-linux-amd64.tar.gz
tar -xvf helm-v3.6.3-linux-amd64.tar.gz
mv linux-amd64/helm /usr/bin/helm
helm version


# Starting the WordPress installation

Let's start by installing WordPress. We will demonstrate how it works and then verify that storage is still present after a reboot.


# add the Helm repository for Bitnami:
helm repo add bitnami https://charts.bitnami.com/bitnami

# Begin reinstallation by using the following command:

helm install wp bitnami/wordpress -n k8sdemo

- This will take a couple of minutes to process. You can follow the status of this installation by executing the following command:


watch -n 1 kubectl get all -o wide -n k8sdemo


# login wordpress

- Helm showed you the commands you need to get the admin credentials for our WordPress site. Let's grab those commands and execute them to log on to the site as follows:
helm status wp
echo Username: user
kubectl get secret --namespace k8sdemo wp-wordpress -o jsonpath="{.data.wordpress-password}" | base64 -d

http://20.121.146.230/admin

username - user
password - JHNzMBMsGr

# Kill the two pods that have the PVC mounted: 

kubectl delete pod --all -n k8sdemo

- As you can see, the two original pods went into a Terminating state. Kubernetes quickly started creating new pods to recover from the pod outage. The pods went through a similar life cycle as the original ones, going from Pending to ContainerCreating to Running.


# after pods recreated go to virtual machine scale sets and stop some of the nodes.


# add the pvc volume github link

