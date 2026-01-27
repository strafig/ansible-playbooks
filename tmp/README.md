```
kubectl apply -f PV.yaml
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm upgrade --install prom-stack prometheus-community/kube-prometheus-stack --values values.yaml -n monitoring --create-namespace --version 81.2.2
```
