# Images
## Pull image
```sh
docker image pull <image-name>
```

## Build image
```sh
docker image build -t <image-tag> -f <path-to-dockerfile> <path-to-exec-dir>
# Examples
docker image build -t my-nginx -f nginx/Dockerfile nginx/
docker image build -t my-image -f docker/Dockerfile .
```


## Containers
## Run container
```sh
docker container run --name <container-name>  <image-name>
#Examples:
# -P -> Picks random available host port
docker container run --name my-python-service -P -d my-python
# -d -> Runs container in detached mode
docker container run --name my-python-service -P -d my-python
# -e -> Runs container with additional environment variables
docker container run -e REDIS_HOST=192.160.0.2 -d -P -e LOG_LEVEL=DEBUG --name my-python-service my-python
# -p -> <host-port:container-port>
docker container run -d -p 5002:5002 --name my-python-service my-python
```

## List containers
```sh
docker container ls
docker container ls -a
docker container ls -a -f status=exited
docker container ls -a -f label=type=workshop
```

## Check logs
```sh
docker container logs <container-name|container-id>
```

## Remove container
```sh
docker container rm <container-name>
```

## Remove all containers
```sh
docker rm -f $(docker ps -q) 
docker rm -f $(docker ps -q -a) 
```

## Exec command in container
```sh
# -it stands for interactive terminal
docker containet exec -it <container-name> <command-to-exec>
# Example
docker container exec -it my-nginx /bin/sh
```

## Stop container
```sh
docker container stop <container-name>
```

## Container port
```sh
docker container port my-python-service
```

## Diff
Displays diff between image and container files.
```sh
docker container diff my-nginx
```

## History
```sh
docker image history my-nginx
```

## Inspect
```sh
docker container inspect my-redis
```
# Volumes
## List volumes
```sh
docker volume ls
```

## Create volume
```sh
docker volume create my-volume
```

## Run container with volume
```sh
# -v <host-dir:container-dir>
docker container run --rm -v /tmp:/data my-volume /data
# -v <volume-name:container-dir>
docker run -v my-volume:/data -d --name container2 my-nginx
# -v <volume-name:container-dir>:ro -> Read only access to volume
```

# Networks
## List networks
```sh
docker network ls
```
## Create network
```sh
docker network create python-redis-network
```

## Run container in network
```sh
docker container run --network python-redis-network -d --name my-redis redis
```

# docker compose

## Run docker compose
```sh
# Run all services
docker compose up
# -d -> Run in detached mode
docker compose up -d
# Run single service
docker compose up <service-name>
```

## List running services
```sh
docker compose ps
```

## Run command in service
```sh
docker compose exec <service-name> <command>
docker compose exec python wget -qO- web
```

## Logs
```sh
# Logs from all services 
docker compose logs
# Logs from single service
docker compose logs -f <service-name>
```

## Load balancing
```sh
# Run 3 services with "web" name
docker compose up -d --scale web=3
```

## Merge compose files
Files provided with `-f` will be loaded in order of passing in command. 
Lists will be extended. Other values will be overriden.

```sh
docker compose -f docker-compose.yml -f dev.yml up -d
```

# Registry

Local registry:
```sh
# Run service with local registry
docker container run -p 5001:5000 -d registry:2
# Check images in registry -> {"repositories":[]}
curl localhost:5001/v2/_catalog
# Tag existing image 
docker image tag my-python:latest localhost:5001/my-python
# Push image with specified tag
docker image push localhost:5001/my-python
# Check images in registry again -> {"repositories":["my-python"]}
curl localhost:5001/v2/_catalog
```

# kubernetes
```sh
# Create cluster
kind create cluster --config ./kind.yaml --name workshop
# Load pod from pod.yaml
kubectl apply -f pod.yaml
# List of pods in default namespace
kubectl get pods
# List of pods in all namespaces
kubectl get pods -A
# List all objects in default namespace
kubectl get all
# List of pods with filter
kubectl get pod -A --selector="app=myapp"
kubectl get pods -A --show-labels
kubectl get pods -l environment=production,tier=frontend
kubectl get pods -A -l 'tier in(control-plane)'
kubectl get pods -A -l 'tier in(control-plane),component notin(kube-scheduler)'
# Exec command on pod
kubectl exec -ti myapp-pod -- curl localhost
# Display logs for pod
kubectl logs myapp-pod
# Enable to access pod in 8080 on localhost
kubectl port-forward pod/myapp-pod 8080:80
# Pod details
kubectl describe pod <pod-name>
# List nodes
kubectl get nodes -o wide
# Node details
kubectl describe node <node-name>
# Expose app by NodePort
kubectl expose pod myapp-pod --type=NodePort --port=80
# Check host port of app exposed by NodePort
kubectl get service
```

## Resources for set based selectors
- Job
- Deployment
- ReplicaSet
- DaemonSet
- StatefullSet

```yml
selector:
  matchLabels:
    component: redis
  matchExpressions:
    - {key: tier, operator: In, values: [cache]}
    - {key: environment, operator: NotIn, values: [dev]}
```

```sh
kubectl get pod
kubectl apply -f replica-set.yaml
kubectl get all
kubectl get events
```
## change pod name
```sh
kubectl apply -f pod.yaml
kubectl get pod
```

## delete pod
```sh
kubectl delete pod myapp-pod
kubectl get pod
```
## scale replica
```sh
kubectl scale rs replicate-my-app --replicas=3
kubectl get all
```
## delete replica set
```sh
kubectl delete rs replicate-my-app
kubectl get all
```
## delete & keep pods
```sh
kubectl apply -f replica-set.yaml
kubectl get rs
kubectl describe rs replicate-my-app
kubectl delete rs replicate-my-app --cascade=false
kubectl get rs
kubectl get pod
```

## Create deployment

```sh
kubectl apply -f deployment.yaml
kubectl get all
kubectl scale deployment/nginx-deployment --replicas=0
kubectl get all
```

## Use env variable
add `env` list

```sh
kubectl rollout history deployments/nginx-deployment
kubectl apply -f deployment.yaml

kubectl rollout status deployment/nginx-deployment
kubectl rollout history deployment/nginx-deployment --revision=1
kubectl rollout history deployment/nginx-deployment --revision=2
kubectl annotate deployment/nginx-deployment kubernetes.io/change-cause="env updated"
kubectl rollout history deployments/nginx-deployment
```

## Exec container and check env exists:

```sh
kubectl get pods
kubectl get pods -l app=myapp -o jsonpath='{.items[0].metadata.name}'
kubectl exec -ti $(kubectl get pods -l app=myapp -o jsonpath='{.items[0].metadata.name}') -- env | grep TEST_ENV
```

## Rollback to previous version

```sh
kubectl rollout undo deployment/nginx-deployment
kubectl annotate deployment/nginx-deployment kubernetes.io/change-cause="env removed"
kubectl rollout history deployments/nginx-deployment

kubectl exec -ti $(kubectl get pods -l app=myapp -o jsonpath='{.items[0].metadata.name}') -- env | grep TEST_ENV

kubectl rollout undo deployment/nginx-deployment --to-revision=2
kubectl exec -ti $(kubectl get pods -l app=myapp -o jsonpath='{.items[0].metadata.name}') -- env | grep TEST_ENV
```

## Scale deployment 
```sh
kubectl describe svc my-app-service
kubectl scale deployment nginx-deployment --replicas=3
kubectl describe svc my-app-service
```
 - check endpoints


## Debug information:
```sh
kubectl logs -l app=myapp
kubectl exec -ti $(kubectl get pods -l app=myapp -o jsonpath='{.items[0].metadata.name}') -- cat /etc/resolv.conf
kubectl exec -ti $(kubectl get pods -l app=myapp -o jsonpath='{.items[0].metadata.name}') -- curl my-app-service
kubectl logs -l app=myapp
kubectl get rs
```

## Namespaces
```sh
kubectl get namespaces
kubectl create namespace <namespace-name>
kubectl apply -f . -n <namespace-name>
```

# Config maps

## From file
```sh
kubectl create configmap configuration --from-file=./
kubectl get configmap/configuration -o yaml > cm.yaml
```

## From env
```sh
kubectl create configmap fromenv --from-env-file=env-file-example
kubectl get configmap/fromenv -o json
kubectl get configmap/fromenv -o yaml
```
## Data as file

```sh
kubectl create configmap test-config --from-file=<my-key-name>=<path-to-file>
kubectl create configmap test-config --from-file=s.json=service.json
kubectl get configmap/test-config -o yaml
```
## Create pods

```sh
kubectl apply -f pod-config.yaml
kubectl logs configmap-pod
kubectl logs configmap-pod | grep line
```

```sh
kubectl apply -f pod-config-volume.yaml
kubectl logs configmap-volume-pod
```

## Autoupdates for mounted configmaps
```sh
kubectl edit configmap configuration
# change service-b.config
kubectl logs -f configmap-volume-pod
```
