## Spin up kubectl container for test
```bash
docker run -it --rm -v ${HOME}:/root/ -v ${PWD}:/work -w /work --net host alpine sh
```

Install kubectl:
```bash
apk add --no-cache curl curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl chmod +x ./kubectl mv ./kubectl /usr/local/bin/kubectl
```

Install helm
```bash
curl -o /tmp/helm.tar.gz -LO https://get.helm.sh/helm-v3.10.1-linux-amd64.tar.gz tar -C /tmp/ -zxvf /tmp/helm.tar.gz mv /tmp/linux-amd64/helm /usr/local/bin/helm chmod +x /usr/local/bin/helm:w
```