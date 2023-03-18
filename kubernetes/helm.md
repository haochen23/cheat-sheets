## Install helm
```bash
curl -o /tmp/helm.tar.gz -LO https://get.helm.sh/helm-v3.10.1-linux-amd64.tar.gz tar -C /tmp/ -zxvf /tmp/helm.tar.gz mv /tmp/linux-amd64/helm /usr/local/bin/helm chmod +x /usr/local/bin/helm
```

## Get full yaml out of template
```bash
helm template sealed-secrets --version 2.7.1 -n kube-system sealed-secrets/sealed-secrets \ > ./kubernetes/secrets/sealed-secrets/controller-helm-v0.19.2.yaml
```

