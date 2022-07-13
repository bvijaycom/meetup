# AZURE Kubernetes Automation

# Before start this excercise,you must have running Rocky linux 8 machine.

# Architecture

[![Watch the image](/architecture.png)]



# AZURE VM
 - Create 1 Rocky Linux 8.5 machine in the AZURE Cloud Portal
 - Note that this will work only in 'Pay-As-You-Go' Subscription and not on Free Tier
 - Make sure you open RDP,http,8080 ports for this new linux AZURE VM or you can also open any port by using port*any
 - Have Mobaxterm/ Putty installed. ** Note: We can use the PowerShell also, but in realtime production environment, Mobaxterm is the best option when we try to work on a junk server and push to the PROD.



# Steps

- Step 1:  Install AZURE CLI in Rocky Linux Virtual Machine
- Step 2:  First install docker in the local linux machine
- Step 3:  Download a sample application
- Step 4:  Test the sample application
- Step 5:  Deploy and use Azure Container Registry
- Step 6:  Install kubectl command 
- Step 7:  Deploy an Azure Kubernetes Service (AKS) cluster
- Step 8:  Run applications in Azure Kubernetes Service (AKS)
- Step 9:  Scale application in Azure Kubernetes Service (AKS)
- Step 10: Update the application in Azure Kubernetes Service (AKS)


#

# Step 1: Install AZURE CLI

- Add the azure CLI Repository keys
```
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
```
- Configure the azure CLI Repository

```
sudo sh -c 'echo -e "[azure-cli]
name=Azure CLI
baseurl=https://packages.microsoft.com/yumrepos/azure-cli
enabled=1
gpgcheck=1
gpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/azure-cli.repo'
```
- Install Azure CLI

```
sudo yum install azure-cli -y
```

- Login and sync your Azure CLI (Linux machine) to your portal.azure.com account

```
az login
```

- Now your linux machine has been fully authenticated with AZURE login.  

# Step 2 - first install docker in the local linux machine

```
 yum install epel-release -y
 yum repolist
```

## Step 2.1 - Install the required dependencies:

```
yum install yum-utils device-mapper-persistent-data lvm2  bash-completion -y
```

```
source /etc/profile.d/bash_completion.sh
```


## Step 2.2 - Add the stable Docker repository by typing:

```
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

## Step 2.3 - Now that we have Docker repository enabled, we can install the latest version of Docker CE (Community Edition) using yum by typing:

```
yum install docker-ce --allowerasing -y

```

## Step 2.4 - Install Docker Compose and Test the Docker-Compose Version:


- Run the below command to download the current stable release of Docker compose.

```
curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/bin/docker-compose
```

- Apply the executable permission for the binary file which we have downloaded.

```
chmod +x /usr/bin/docker-compose
```

- If the docker compose is installed on a different location For example: /usr/local/bin/ , You can copy the executable to /usr/bin directory.

- You can check the version of docker-compose using the below command.

```
docker-compose --version
```


## Step 2.5 - Once the Docker package is installed, we start the Docker daemon with:

```
systemctl start docker;systemctl status docker;systemctl enable docker
```

## Step 2.6 - At the time of the writing of this article, the current stable version of Docker is 20.10.15, we can check our Docker version by typing:

```
docker -v
```


# Step 3: Download sample application

```
yum install git tree -y

```

 
# Step 5: Deploy and use Azure Container Registry

- Create AZURE resource Group

```
az group create --name  cloudnloudrg --location eastus
```

- Create AZURE Container Registry Under the above Created Resource Group.


```
az acr create --resource-group cloudnloudrg --name cnlacr1 --sku Basic
```

- Login to the Azure Container registry


```
az acr login --name cnlacr1
```

- List the Docker Images


```
docker images
```

- List the no of ACR in your portal.azure.com


```
az acr list --resource-group cloudnloudrg --query "[].{acrLoginServer:loginServer}" --output table
```

- you will get the below output.So your AZURE Container Registry Name is below ...


```
cnlacr1.azurecr.io
```


- you need to change the docker image name towards to match your ACR repo name.Or else while you push the image will end up with error.


```
docker tag cloudnloud/azure-vote-front:v1 cnlacr1.azurecr.io/azure-vote-front:v1
```
- List the docker images

- Make sure you are seeing the docker image name with match to your ACR repository.

```
docker images
```
- Push your prepared custom image to your newly created ACR.

```
docker push cnlacr1.azurecr.io/azure-vote-front:v1
```

- list the ACR repository revisions

```
az acr repository list --name cnlacr1 --output table
```
- List your repository in ACR.

```
az acr repository show-tags --name cnlacr1 --repository azure-vote-front --output table
```


# Step 6: install kubectl command 

- Configure Kubernetes Software package repo.

```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```

- Install Kubectl

```
yum install -y kubectl
```

- To ensure all are ok the following command should work without any error

```
kubectl version --client
```

# Step 7: Deploy an Azure Kubernetes Service (AKS) cluster

- Create kubernetes Cluster with minimum 2 worker Node

- In your learning setup,if you have project portal.azure.com test account then increase node count from 1 to 3.


```
az aks create \
    --resource-group cloudnloudrg \
    --name myAKSCluster \
    --node-count 2 \
    --generate-ssh-keys \
    --attach-acr cnlacr1
	
```
- Install AZURE AKS CLI

```
az aks install-cli
```
- Reterive the AKS cluster credentials.This will help kubectl command to run without any issues.

```
az aks get-credentials --resource-group cloudnloudrg --name myAKSCluster
```
- List all the nodes in Kubernetes cluster.

```
kubectl get nodes
```


# now deploy sample application in kubernetes cluster in new namespace

```
kubectl create ns k8sdemo

kubectl apply -f https://raw.githubusercontent.com/bvijaycom/meetup/main/guestbook-all-in-one.yaml -n k8sdemo

```

## open putty in the 2nd window and run the below command

```
watch -n 1 kubectl get all -n k8sdemo -o wide


```

## now open new putty window and execute the below load script [replace the external ip with what you get from above command output]


```
while true; do
curl -m 1 http://20.102.24.82/;
sleep 5;
done

```

## go to virtual machine scale sets....

## you will see 2 nodes .After navigating to the pane of the scale set, go to the Instances view, select the instance you want to shut down, and then hit the Stop button



## access http://20.102.24.82/ and keep submit the entries..Keep watch the **watch -n 1 kubectl get all -n k8sdemo -o wide** command output

## What you see here is the following:
-  The Redis master pod running on node 2 got terminated as the host became unhealthy.
- A new Redis master pod got created, on host 0. This went through the stages **Pending, ContainerCreating, and then Running**.


# Solving out-of-resource failure

- Kubernetes uses requests to calculate how much CPU power or memory a certain pod requires. The guestbook application has requests defined for all the deployments. If you open the guestbook-all-in-one.yaml file in the folder. you'll see the following for the redis-replica deployment:

```
63 kind: Deployment
64 metadata:
65 name: redis-replica
...
83 resources:
84 requests:
85 cpu: 200m
86 memory: 100Mi

```

- This section explains that every pod for the redis-replica deployment requires 200m of a CPU core (200 milli or 20%) and 100MiB (Mebibyte) of memory. In your 2 CPU clusters (with node 1 shut down), scaling this to 10 pods will cause issues with the available resources. Let's look into this:

# Let's start by scaling the redis-replica deployment to 10 pods:

- kubectl scale deployment/redis-replica --replicas=10 -n k8sdemo


# This will cause a couple of new pods to be created. We can check our pods  using the following:
- now many are shown in pending state.This occurs if the cluster is out of resources.

# We can get more information about these pending pods using the following command:
 
```
kubectl describe pod redis-replica-5bc7bcc9c4-svcc8 -n k8sdemo
```

now you will get the following output

```
Events:
  Type     Reason            Age                  From               Message
  ----     ------            ----                 ----               -------
  Warning  FailedScheduling  46s (x2 over 2m14s)  default-scheduler  0/2 nodes are available: 1 Insufficient cpu, 1 node(s) had taint {node.kubernetes.io/unreachable: }, that the pod didn't tolerate.

```

# now start the stopped node from virtual scale set

## Keep watch the **watch -n 1 kubectl get all -n k8sdemo -o wide** command