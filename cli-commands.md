### These commands are used in the lecture "Deploying Images in Kubernetes from private Docker repository"

##### Print full docker login command for aws ecr

`aws ecr get-login-password`

#### Login to docker private repo

`docker login -u username -p password`

Base64 encode config file

`cat .docker/config.json | base64`

Create docker login secret from config.json file

```kubectl create secret generic my-registry-key \
--from-file=.dockerconfigjson=.docker/config.json \
--type=kubernetes.io/dockerconfigjson
```
`kubectl create secret generic my-registry-key --from-file=.dockerconfigjson=.docker/config.json --type=kubernetes.io/dockerconfigjson`

`kubectl get secret`

Create docker login secret with login credentials

```kubectl create secret docker-registry my-registry-key \
--docker-server=https://private-repo \
--docker-username=user \
--docker-password=pwd 
```

`kubectl create secret docker-registry my-registry-key --docker-server=https://private-repo --docker-username=user --docker-password=pwd`

Access minikube console

`minikube ssh`

Copy config.json file from Minikube to my host

`minikube cp minikube:/home/docker/.docker/config.json /users/nana/.docker/config.json`
