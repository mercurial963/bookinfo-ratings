## How to run with Docker

```bash
# Build Docker Image for rating service
docker-compose up -d --build

* Test with path `/ratings/1` and `/ratings/health`

Prepare Kubernetes and Helm

Once kubernetes and helm is set up properly, run as follows:
## Deploy mongodb
```console
# Add bitnami charts repository
helm repo add bitnami https://charts.bitnami.com/bitnami
# List repo
helm repo list
# Update repo
helm repo update
# Deploy MongoDB Helm Charts
helm install mongodb bitnami/mongodb --set persistence.enabled=false
```


## Create initial script with Config Map

```console
# Create configmap
kubectl create configmap bookinfo-dev-ratings-mongodb-initdb --from-file=databases/ratings_data.json --from-file=databases/scripts.sh

# Create secret mongodb 
kubectl apply -f bookinfo-secret/bookinfo-dev-ratings-mongodb-secret.yaml

kubectl get secret,configmap
```
```console
helm install -f helm-values/values-bookinfo-dev-ratings-mongodb.yaml bookinfo-dev-ratings-mongodb bitnami/mongodb
```
install bookinfo-ratings
```console

# secret for pull image from ghcr.io
kubectl apply -f bookinfo-secret/ghcr-secret.yaml

# install bookinfo-ratings
helm install -f helm-values/values-bookinfo-dev-ratings.yaml bookinfo-dev-ratings helm/

kubectl port-forward svc/bookinfo-dev-ratings 8080:8080


```



check
```console
export MONGODB_ROOT_PASSWORD=$(kubectl get secret bookinfo-dev-ratings-mongodb-secret -o jsonpath="{.data.mongodb-root-password}" | base64 --decode)


kubectl run mongodb-client --rm --tty -i --restart='Never' --image bitnami/mongodb:4.4.4-debian-10-r5   --command -- mongo admin --host bookinfo-dev-mongodb --authenticationDatabase admin -u root -p $MONGODB_ROOT_PASSWORD

show dbs

use ratings-dev

show collections

db.ratings.find()
```
kubectl port-forward svc/bookinfo-dev-ratings 8080:8080