# Little Diary
Little diary helps to track children measures and activities, such as weight, height, sleep time.

User can register a family, add members to it by generating links and register babies inside family. 
For every baby user can add measurements with time link. 

A working app placed here: http://ld.de1mos.net/

Issues are managed in corresponding projects. 
For Kanban board [GitHub Project](https://github.com/users/de1mos242/projects/1) is used.

## Architecture
Application has 5 parts (for now):
1. [Root app](https://github.com/de1mos242/little-diary-root) (this repo). 
Contains Helm charts for all services and github workflows for deployment to GKE.
2. [Auth service](https://github.com/de1mos242/little-diary-auth-service).
Contains a backend micro-service responsible for authorization and user management processes.
3. [Family service](https://github.com/de1mos242/little-diary-family-service).
Contains a backend micro-service responsible for family, family members and babies management.
4. [Measurement service](https://github.com/de1mos242/little-diary-measurement-service).
Contains a backend micro-service responsible for storing kids measurements.
5. [Web app](https://github.com/de1mos242/little-diary-web-app).
Contains a frontend application.

#### Technologies
* Auth service - Python + Flask + SqlAlchemy + Postgres. Alembic for schema migrations. 
* Family service - Python + Aiohttp + aiopg + SqlAlchemy + Postgres. Alembic for schema migrations.
* Measurement service - Golang + GinGonic + Gorm + Postgres. GoMigrate for schema migrations.
* Web app - TypeScript + React + NextJS + MobX + React SemanticUI.
* Root app - Kubernetes + Helm.

#### Project specifics
* For authentication services use JWT token with RSA+SHA256 algorithm, so only auth service has private key.
* There is no direct communication between auth service and family service. 
Measurement service make a login call to auth service to acquire jwt token and use it to perform access checks in family service.
* All models that can be exported to a client have `external uuid` and use this field instead of internal id.
* There is no `POST` operations, create and update operations use `PUT`. 
Client have to generate `uuid` for resource before creation and send it to server. 
This makes all endpoints idempotent, so communication between a client and a server is tolerant for network errors and prevent creating duplicates on retries.



## Root service
Root repository contains Helm charts to deploy whole application into Kubernetes cluster and GitHub workflows to deploy application into Google Kubernetes Engine (GKE).

### Local development

With minikube it's useful to expose ingress to localhost:

```shell script
minikube addons enable ingress
minikube ip
```

Add records to /etc/hosts:
```
MINIKUBE_IP auth.ld.local
MINIKUBE_IP family.ld.local
MINIKUBE_IP measurement.ld.local
MINIKUBE_IP web.ld.local
```

Install repositories:
```shell script
helm repo add bitnami https://charts.bitnami.com/bitnami
```
Install dependencies and charts:
```shell script
kubectl create ns ld-dev

helm dependency update k8s/auth_service/
helm dependency update k8s/family_service/
helm dependency update k8s/measurement_service/

helm upgrade --install -n ld-dev auth-chart k8s/auth_service/
helm upgrade --install -n ld-dev family-chart k8s/family_service/
helm upgrade --install -n ld-dev measurement-chart k8s/measurement_service/
helm upgrade --install -n ld-dev web-chart k8s/web_app/
helm upgrade --install -n ld-dev root-chart k8s/root_app/
```

### Production deployment
Deployed application runs in GKE cluster through GitHub Actions. There are 4 workflows to deploy concrete services:
* gke-auth-service.yaml
* gke-family-service.yaml
* gke-measurement-service.yaml
* gke-web-app.yaml 

They listen to webhook events, that fired by workflows in corresponding repositories after deployment image to DockerHub.

Workflow `gke-root-service.yaml` deploys ingress and SSL certificate configurations into GKE. 

#### GKE usage specifics
* Using GCE ingress controller
* For SSL Certificate management GCE is used
* Used static external ip address with privilege level as ingress requires this level
* It needs to create custom load balancer in GCE with redirection rule from http to https and link it to static external ip address  
* Script to create kubeconfig for deployment from GitHub got from [here](https://gravitational.com/blog/kubectl-gke/) 
