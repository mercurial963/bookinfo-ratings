## How to run with Docker

```bash
# build Docker Image for rating service
docker-compose up -d --build

```
### check result

[localhost:8080/health](http://localhost:8080/health)  
[localhost:8080/ratings/ratings/1](http://localhost:8080/ratings/ratings/1)

## How to run with Helm (dev)

### Prepare helm package mongodb
```console
# Add bitnami charts repository
helm repo add bitnami https://charts.bitnami.com/bitnami
# List repo
helm repo list
# Update repo
helm repo update

```
### Create Configmap & Secret mongodb

```console
# Create configmap
kubectl create configmap bookinfo-dev-ratings-mongodb-initdb --from-file=databases/ratings_data.json --from-file=databases/scripts.sh

# Create secret mongodb 
kubectl apply -f bookinfo-secret/bookinfo-dev-ratings-mongodb-secret.yaml

kubectl get secret,configmap

```
### Install mongodb

```console
# install mongodb
helm install -f helm-values/values-bookinfo-dev-ratings-mongodb.yaml bookinfo-dev-ratings-mongodb bitnami/mongodb

<!-- # check
export MONGODB_ROOT_PASSWORD=$(kubectl get secret bookinfo-dev-ratings-mongodb-secret -o jsonpath="{.data.mongodb-root-password}" | base64 --decode)

kubectl run mongodb-client --rm --tty -i --restart='Never' --image bitnami/mongodb:4.4.4-debian-10-r5   --command -- mongo admin --host bookinfo-dev-mongodb --authenticationDatabase admin -u root -p $MONGODB_ROOT_PASSWORD

show dbs

use ratings-dev

show collections

db.ratings.find() -->

```
### Install bookinfo-ratings

```console
# create secret for pull image from ghcr.io
kubectl apply -f bookinfo-secret/ghcr-secret.yaml

# install bookinfo-ratings
helm install -f helm-values/values-bookinfo-dev-ratings.yaml bookinfo-dev-ratings helm/

# forward port
kubectl port-forward svc/bookinfo-dev-ratings 8080:8080

```
### Check result

[localhost:8080/health](http://127.0.0.1:8080/health)  
[localhost:8080/ratings/ratings/1](http://localhost:8080/ratings/ratings/1)

## Add Security 
### Code review
```console
# shiftleft
sh <(curl https://slscan.sh)

# njsscan
docker run -v $(pwd)/src:/src opensecurity/njsscan:latest /src

# Sonarqube
docker run --rm -v "$PWD:/usr/src" \
    -e SONAR_HOST_URL="http://34.87.71.125:9000" \
    sonarsource/sonar-scanner-cli \
    -D sonar.projectKey=bookinfo-ratings \
    -D sonar,projectName=bookinfo-ratings \
    -D sonar.sources=./src \
    -D sonar.login=admin \
    -D sonar.password=sonarqube

rm -rf .scannerwork
```

### SCA (Software Composition Analysis)
```console
# OWASP Dependency Check
  docker run --rm -v "$PWD:/usr/src" \
    -e SONAR_HOST_URL="http://34.87.71.125:9000" \
    sonarsource/sonar-scanner-cli \
    -D sonar.projectKey=bookinfo-ratings\
    -D sonar,projectName=bookinfo-ratings \
    -D sonar.sources=./src \
    -D sonar.login=admin \
    -D sonar.password=sonarqube
```

### Docker image security 
```console
# Syft
syft ghcr.io/mercurial963/bookinfo-dev-ratings:dev

# Grype
grype db update
grype ghcr.io/mercurial963/bookinfo-dev-ratings:dev

# Anchore-engine

Install on kubernetes with helm chart
add registry in Anchore-engine with anchore-cli
```

### Infrastructure as a code security
```console
# Kubesec

helm template -f helm-values/values-bookinfo-dev-ratings.yaml bookinfo-dev-ratings helm/ | \
docker run -i --rm kubesec/kubesec:2.11.0 scan /dev/stdin

# sKan

helm template -f helm-values/values-bookinfo-dev-ratings.yaml bookinfo-dev-ratings helm/ | \
docker run -i --rm -v $(pwd)/result:/result alcide/skan:v0.9.0 \
pipe> manifest --outputfile /result/skan-result.html -f -
```
