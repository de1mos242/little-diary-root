# little-diary-root
Root repository with description and common things like k8s

Install repositories:
```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm dependency update k8s/auth_service/
```

Install/Update auth chart:

`helm upgrade --install --namespace auth-namespace auth-chart k8s/auth_service/`

## Dashboard
```
kubectl apply -f dashboard/role.yaml
kubectl apply -f dashboard/accout.yaml
kubectl proxy
```

http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

To get token

`kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')`




## Install postgres operator:
`kubectl create ns postgres-operator`
```
helm upgrade --install \
  --namespace postgres-operator \
  -f ./k8s/postgres-operator/values-crd.yaml \
  postgres-operator \
  ./k8s/postgres-operator
```
### Auth service
 