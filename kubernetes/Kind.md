# kind
## Create and Destroy

```bash
kind create cluster --name webhook
kind delete cluster --name webhook


# create a three node cluster
cat <<EOF | kind create cluster --name webhook --config -
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
# One control plane node and three "workers".
#
# While these will not add more real compute capacity and
# have limited isolation, this can be useful for testing
# rolling updates etc.
#
# The API-server and other control plane components will be
# on the control-plane node.
#
# You probably don't need this unless you are testing Kubernetes itself.
nodes:
- role: control-plane
- role: worker
- role: worker
- role: worker
EOF
```


## Metallb for LoadBalancer


```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.5/config/manifests/metallb-native.yaml
```

### Set up address pool
```bash
docker network inspect -f '{{.IPAM.Config}}' kind
```

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: example
  namespace: metallb-system
spec:
  addresses:
  - 172.18.255.200-172.18.255.250
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: empty
  namespace: metallb-system
```

Or

```bash
cat <<EOF | kubectl apply -f -
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: example
  namespace: metallb-system
spec:
  addresses:
  - 172.18.255.200-172.18.255.250
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: empty
  namespace: metallb-system
EOF

```

## Ingress alternative for Metallb on macos
```bash

```
## get loadbalancer ip

```bash
LB_IP=$(kubectl get svc/nodeapp -o=jsonpath='{.status.loadBalancer.ingress[0].ip}')
```

## kind with Nignx ingress for MacOS
```bash

# create the cluster with port mapping


cat <<EOF | kind create cluster --name webhook --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  
  extraMounts:
    - hostPath: /tmp
      containerPath: /www

  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
EOF


```

```bash

# Deploy Nginx ingress controller

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: foo-app
  labels:
    app: foo
spec:
  containers:
  - name: foo-app
    image: hashicorp/http-echo:0.2.3
    args:
    - "-text=foo"
---
kind: Service
apiVersion: v1
metadata:
  name: foo-service
spec:
  selector:
    app: foo
  ports:
  # Default port used by the image
  - port: 5678
---
kind: Pod
apiVersion: v1
metadata:
  name: bar-app
  labels:
    app: bar
spec:
  containers:
  - name: bar-app
    image: hashicorp/http-echo:0.2.3
    args:
    - "-text=bar"
---
kind: Service
apiVersion: v1
metadata:
  name: bar-service
spec:
  selector:
    app: bar
  ports:
  # Default port used by the image
  - port: 5678
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
  - http:
      paths:
      - pathType: Prefix
        path: "/foo"
        backend:
          service:
            name: foo-service
            port:
              number: 5678
      - pathType: Prefix
        path: "/bar"
        backend:
          service:
            name: bar-service
            port:
              number: 5678
---

```

```bash
#!/usr/bin/env bash
number=$1
name=$2
region=$3
zone=$4
twodigits=$(printf "%02d\n" $number)
if [ -z "$3" ]; then
  region=us-east-1
fi
if [ -z "$4" ]; then
  zone=us-east-1a
fi
if hostname -I 2>/dev/null; then
  myip=$(hostname -I | awk '{ print $1 }')
else
  myip=$(ipconfig getifaddr en0)
fi
reg_name='kind-registry'
reg_port='5000'
running="$(docker inspect -f '{{.State.Running}}' "${reg_name}" 2>/dev/null || true)"
if [ "${running}" != 'true' ]; then
  docker run \
    -d --restart=always -p "0.0.0.0:${reg_port}:5000" --name "${reg_name}" \
    registry:2
fi
cache_port='5000'
cat > registries <<EOF
docker https://registry-1.docker.io
us-docker https://us-docker.pkg.dev
us-central1-docker https://us-central1-docker.pkg.dev
quay https://quay.io
gcr https://gcr.io
EOF
cat registries | while read cache_name cache_url; do
running="$(docker inspect -f '{{.State.Running}}' "${cache_name}" 2>/dev/null || true)"
if [ "${running}" != 'true' ]; then
  cat > ${HOME}/.${cache_name}-config.yml <<EOF
version: 0.1
proxy:
  remoteurl: ${cache_url}
log:
  fields:
    service: registry
storage:
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
http:
  addr: :5000
  headers:
    X-Content-Type-Options: [nosniff]
health:
  storagedriver:
    enabled: true
    interval: 10s
    threshold: 3
EOF
  docker run \
    -d --restart=always -v ${HOME}/.${cache_name}-config.yml:/etc/docker/registry/config.yml --name "${cache_name}" \
    registry:2
fi
done
cat << EOF > kind${number}.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
featureGates:
  EphemeralContainers: true
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 6443
    hostPort: 70${twodigits}
- role: worker
- role: worker
- role: worker
networking:
  serviceSubnet: "10.$(echo $twodigits | sed 's/^0*//').0.0/16"
  podSubnet: "10.1${twodigits}.0.0/16"
kubeadmConfigPatches:
- |
  apiVersion: kubeadm.k8s.io/v1beta2
  kind: ClusterConfiguration
  apiServer:
    extraArgs:
      service-account-signing-key-file: /etc/kubernetes/pki/sa.key
      service-account-key-file: /etc/kubernetes/pki/sa.pub
      service-account-issuer: api
      service-account-api-audiences: api,vault,factors
  metadata:
    name: config
- |
  kind: InitConfiguration
  nodeRegistration:
    kubeletExtraArgs:
      node-labels: "ingress-ready=true,topology.kubernetes.io/region=${region},topology.kubernetes.io/zone=${zone}"
containerdConfigPatches:
- |-
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors."localhost:${reg_port}"]
    endpoint = ["http://${reg_name}:${reg_port}"]
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
    endpoint = ["http://docker:${cache_port}"]
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors."us-docker.pkg.dev"]
    endpoint = ["http://us-docker:${cache_port}"]
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors."us-central1-docker.pkg.dev"]
    endpoint = ["http://us-central1-docker.pkg.dev:${cache_port}"]
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors."quay.io"]
    endpoint = ["http://quay:${cache_port}"]
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors."gcr.io"]
    endpoint = ["http://gcr:${cache_port}"]
EOF
kind create cluster --name kind${number} --config kind${number}.yaml
ipkind=$(docker inspect kind${number}-control-plane | jq -r '.[0].NetworkSettings.Networks[].IPAddress')
networkkind=$(echo ${ipkind} | awk -F. '{ print $1"."$2 }')
kubectl config set-cluster kind-kind${number} --server=https://${myip}:70${twodigits} --insecure-skip-tls-verify=true
kubectl --context=kind-kind${number} apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/namespace.yaml
kubectl --context=kind-kind${number} apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/metallb.yaml
kubectl --context=kind-kind${number} create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"
cat << EOF > metallb${number}.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - ${networkkind}.0${twodigits}.1-${networkkind}.0${twodigits}.254
EOF
kubectl --context=kind-kind${number} apply -f metallb${number}.yaml
docker network connect "kind" "${reg_name}" || true
docker network connect "kind" docker || true
docker network connect "kind" us-docker || true
docker network connect "kind" us-central1-docker || true
docker network connect "kind" quay || true
docker network connect "kind" gcr || true
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: local-registry-hosting
  namespace: kube-public
data:
  localRegistryHosting.v1: |
    host: "localhost:${reg_port}"
    help: "https://kind.sigs.k8s.io/docs/user/local-registry/"
EOF
kubectl config rename-context kind-kind${number} ${name}
```