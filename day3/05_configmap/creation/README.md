# From file
```sh
kubectl api-resources
kubectl create configmap configuration --from-file=./
kubectl get configmap/configuration -o yaml > cm.yaml
kubectl create configmap configuration --from-file=./ --dry-run=client -o yaml > cm.yaml
```

# From env
```sh
kubectl create configmap fromenv --from-env-file=env-file-example
kubectl get configmap/fromenv -o json
kubectl get configmap/fromenv -o yaml
```
# Data as file

```sh
kubectl create configmap test-config --from-file=<my-key-name>=<path-to-file>
kubectl create configmap test-config --from-file=s.json=service.json
kubectl get configmap/test-config -o yaml
```
# Create pods

```sh
kubectl apply -f pod-config.yaml
kubectl logs configmap-pod
kubectl logs configmap-pod | grep line
```

```sh
kubectl apply -f pod-config-volume.yaml
kubectl logs configmap-volume-pod
```

# Autoupdates for mounted configmaps
```sh
kubectl edit configmap configuration
# change service-b.config
kubectl logs -f configmap-volume-pod
```
