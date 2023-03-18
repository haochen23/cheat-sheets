# Dapr

## Deploy Dapr

```bash
# Deploy dapr
dapr init --kubernetes --wait

# Verify dapr deployment
dapr status -k

# dapr dashboard
dapr dashboard -k -p 9999
```

## Setup local redis state store
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm install redis bitnami/redis --set image.tag=6.2

```

