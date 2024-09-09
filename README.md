[Notes overview](https://github.com/TheAbys/devops-bootcamp-certification-project/blob/master/README.md)

# 17 - Deploying Images in Kubernetes from private Docker repository

At first login into aws ecr (again company proxy is required)
I use zsh:

    vim ~/.zshrc

Add following lines

    export NO_PROXY=localhost,127.0.0.1,10.96.0.0/12,192.168.59.0/24,192.168.49.0/24,192.168.39.0/24
    export HTTP_PROXY=<http-proxy>
    export HTTPS_PROXY=<https-proxy>

Reload shell

    source ~/.zshrc

Check if variables are set

    env | grep PROXY

## login

For this to work the aws cli must be preconfigured

    aws configure

    aws ecr get-login-password --region eu-central-1 | docker login --username AWS --password-stdin 561656302811.dkr.ecr.eu-central-1.amazonaws.com

This is not sufficent, as minikube won't have access to this stored credentials.

## ssh into minikube and configure there

Get the login password

    aws ecr get-login-password

SSH into minikube cluster

    minikube ssh

Login to ECR within the cluster

    docker login --username AWS -p <password> 561656302811.dkr.ecr.eu-central-1.amazonaws.com

Copy from minikube cluster to local

    minikube cp minikube:/home/docker/.docker/config.json /home/mueller/.docker/config.json

Convert to base64

    ~/.docker/config.json | base64

or use kubectl

    kubectl create secret generic my-registry-key \
    --from-file=.dockerconfigjson=/home/mueller/.docker/config.json \
    --type=kubernetes.io/dockerconfigjson

## doing all in one step

    kubectl create secret docker-registry my-registry-key-two \
    --docker-server=https://561656302811.dkr.ecr.eu-central-1.amazonaws.com \
    --docker-username=AWS \
    --docker-password=<password>

Last version is only one step, but also only for one specific repository.
If there are multiple repositories accessable through the same authentication the first solution would be more practical.

    kubectl delete -f my-app-deployment.yaml

When it doesn't work it shows an error ImagePullBackOff or ErrImagePull

    kubectl get pods

Secrets must be in the same namespace.

# 18 - Kubernetes Operators for Managing Complex Applications

Kubernetes can completely manage lifecycle of stateless apps.
But stateful apps are not that easy to automate. Therefore you can use the operator.
Mostly required for stateful applications that cannot really be hosted outside the cluster.

The operator can work with CRD (custom resource defintions) which are specifically defined for the stateful app like a specific database.

# 19 - Secure your cluster - Authorization with RBAC

RBAC Role Based Access Control

## Ressource Role

It is required to give each user the least privilege.
This means a developer user should maybe not have the rights to create a new storage and also should not be allowed to edit specific services or deployments.

For this we can use a namespace specific rolebinding on a user or group.

    rules:
        - apiGroups
        resources -> resource types pods, services, deployments,...
        verbs -> actions like create or delete deployment

## Ressource ClusterRole

Administrators need permissions to administer the cluster, therefore here we can't use the namespace specific rolebindings.
We use ClusterRole for that.

API Server handles the authentication of all requests and checks if a user is allowed to connect.

An Administrator manually creates certificates OR ldap for users which allows them to access the cluster.

## Ressource ServiceAccount

But we also need authorization for applications. Other services or tools do access the cluster.

## API Access

Check if the current user is able to execute a command

    kubectl auth can-i create deployments


User at first logs in through username/password, certificate, whatever