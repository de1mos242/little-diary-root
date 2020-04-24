# little-diary-root
Root repository with description and common things like k8s

Install repositories:
```shell script
helm repo add bitnami https://charts.bitnami.com/bitnami
```

## Auth service

Initialize:
```shell script
kubectl create ns auth-namespace
helm dependency update k8s/auth_service/
```

Install/Update auth chart:

`helm upgrade --install --namespace auth-namespace auth-chart k8s/auth_service/`


## Family service

Initialize:

```shell script
kubectl create ns family-namespace
helm dependency update k8s/family_service/
```

Install/Update family chart:

`helm upgrade --install --namespace family-namespace family-chart k8s/family_service/`

## Measurement service

Initialize:

```shell script
kubectl create ns measurement-namespace
helm dependency update k8s/measurement_service/
```

Install/Update measurement chart:

`helm upgrade --install --namespace measurement-namespace measurement-chart k8s/measurement_service/`

## Web application

Initialize:

```shell script
kubectl create ns web-namespace
```

Install/Update measurement chart:

`helm upgrade --install --namespace web-namespace web-chart k8s/web_app/`


## Dashboard
```shell script
kubectl apply -f dashboard/role.yaml
kubectl apply -f dashboard/accout.yaml
kubectl proxy
```

http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

To get token

`kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')`


or using minikube:

`minikube dashboard`
