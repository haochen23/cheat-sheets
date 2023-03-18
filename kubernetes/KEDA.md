## Install KEDA
```bash
helm repo add kedacore https://kedacore.github.io/charts
helm repo update

kubectl create namespace keda
helm install keda kedacore/keda --namespace keda

```
1. [sample-rabbitmq-aqq](https://github.com/kedacore/sample-go-rabbitmq)
2. 