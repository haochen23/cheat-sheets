```bash
# Source: https://gist.github.com/0a322f969368bec74b75677da217291c

#################################################################################

# Manage Container (Docker) Images, Helm, CNAB, and Other Artifacts With Harbor #

# https://youtu.be/f931M4-my1k #

#################################################################################

# Additional Info:

# - Harbor: https://goharbor.io

# - Signing And Verifying Container Images With Sigstore Cosign And Kyverno: https://youtu.be/HLb1Q086u6M

#########

# Setup #

#########

# Create a Kubernetes cluster with Ingress controller set as default.

# Replace `127.0.0.1` with the Ingress Service IP

export INGRESS_HOST=127.0.0.1

git clone https://github.com/vfarcic/harbor-demo

cd harbor-demo

helm repo add harbor https://helm.goharbor.io

helm repo update

helm upgrade --install harbor harbor/harbor \

--namespace harbor \

--create-namespace \

--set expose.ingress.hosts.core=harbor.$INGRESS_HOST.nip.io \

--set expose.ingress.hosts.notary=notary.$INGRESS_HOST.nip.io \

--set externalURL=http://harbor.$INGRESS_HOST.nip.io \

--values values.yaml \

--wait

echo "http://harbor.$INGRESS_HOST.nip.io"

# Open it in a browser

# User: admin

# Password: Harbor12345

# `Administration` > `Registries` > `+ NEW ENDPOINT` > Add Docker Hub registry

# `Projects` > `NEW PROJECT`

# - Project Name: dot

# - Press the `OK` button

# `Projects` > `dot` > `Configuration`

# - Check `Cosign` in `Deployment Security`

# - Check `Prevent vulnerable images from running` in `Deployment Security` and set the severity to `High`.

# - Set `Automatically scan images on push` in `Vulnerability scanning`

cosign generate-key-pair

cp go.mod.orig go.mod

# Install `yq` from https://github.com/mikefarah/yq if you do not have it already

yq --inplace \

".image.repository = \"harbor.$INGRESS_HOST.nip.io/dot/silly-demo\"" \

helm/values.yaml

yq --inplace \

".ingress.host = \"silly-demo.$INGRESS_HOST.nip.io\"" \

helm/values.yaml

############################################

# Build And Push Container (Docker) Images #

############################################

echo harbor.$INGRESS_HOST.nip.io

# If using Docker:

#  Configure the daemon to use insecure registry.

# More info: https://docs.docker.com/registry/insecure/#deploy-a-plain-http-registry.

# If using nerdctl:

# Create alias `docker` to `nerdctl`.

#  Add `--insecure-registry` to the all the `docker login` and `docker image push`, and `docker image pull` commands.

docker login --username admin harbor.$INGRESS_HOST.nip.io

# Password: Harbor12345

docker image build \

--tag harbor.$INGRESS_HOST.nip.io/dot/silly-demo:v0.0.1 .

docker image push \

harbor.$INGRESS_HOST.nip.io/dot/silly-demo:v0.0.1

##################################

# Sign Container (Docker) Images #

##################################

docker image pull \

harbor.$INGRESS_HOST.nip.io/dot/silly-demo:v0.0.1

cosign sign --key cosign.key \

harbor.$INGRESS_HOST.nip.io/dot/silly-demo:v0.0.1

################################################

# Vulnerability Scanning With Harbor And Trivy #

################################################

docker image pull \

harbor.$INGRESS_HOST.nip.io/dot/silly-demo:v0.0.1

# Fix the `High` issues

docker image build \

--tag harbor.$INGRESS_HOST.nip.io/dot/silly-demo:v0.0.2 .

docker image push \

harbor.$INGRESS_HOST.nip.io/dot/silly-demo:v0.0.2

cosign sign --key cosign.key \

harbor.$INGRESS_HOST.nip.io/dot/silly-demo:v0.0.2

docker image pull \

harbor.$INGRESS_HOST.nip.io/dot/silly-demo:v0.0.2

###################################################

# Store Helm Charts And Other Artifacts In Harbor #

###################################################

cat helm/values.yaml

yq --inplace ".image.tag = \"v0.0.2\"" helm/values.yaml

yq --inplace ".version = \"0.0.2\"" helm/Chart.yaml

helm package helm

helm registry login harbor.$INGRESS_HOST.nip.io --insecure

# User: admin

# Password: Harbor12345

helm push silly-demo-0.0.2.tgz \

oci://harbor.$INGRESS_HOST.nip.io/dot

###########

# Destroy #

###########

yq --inplace ".image.tag = \"latest\"" helm/values.yaml

yq --inplace ".version = \"0.0.1\"" helm/Chart.yaml

# Destroy or reset the Kubernetes cluster

[![@ericchen-lab3nz](https://avatars.githubusercontent.com/u/104544129?s=80&v=4)](https://gist.github.com/ericchen-lab3nz)

Write
```