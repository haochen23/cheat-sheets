```bash
# Source: https://gist.github.com/d1bd7ab00d2288c663e436cd513efe85

###########################################################################
# Signing And Verifying Container Images With Sigstore Cosign And Kyverno #
# https://youtu.be/HLb1Q086u6M                                            #
###########################################################################

# Additional Info:
# - Sigstore (Cosign): https://sigstore.dev
# - Kubernetes-Native Policy Management With Kyverno: https://youtu.be/DREjzfTzNpA
# - How To Replace Docker With nerdctl And Rancher Desktop: https://youtu.be/evWPib0iNgY
# - Bitnami Sealed Secrets - How To Store Kubernetes Secrets In Git Repositories: https://youtu.be/xd2QoV6GJlc

#########
# Setup #
#########

# Create a Kubernetes cluster

# Install the `cosign` CLI from https://docs.sigstore.dev/cosign/installation

git clone https://github.com/vfarcic/silly-demo

cd silly-demo

# Replace `[...]` with your container registry user (e.g., Docker Hub user).
export REGISTRY_USER=[...]

# Replace `docker.io` with the registry address is not using Docker Hub.
export REGISTRY_ADDR=docker.io

cosign generate-key-pair

mv cosign.pub cosign.pub cosign/.

helm repo add kyverno \
    https://kyverno.github.io/kyverno

helm repo update

helm upgrade --install \
    kyverno kyverno/kyverno \
    --namespace kyverno \
    --create-namespace \
    --wait

kubectl create namespace production

# Install `yq` from https://github.com/mikefarah/yq if you do not have it already

yq --inplace \
    ".spec.template.spec.containers[0].image = \"$REGISTRY_ADDR/$REGISTRY_USER/silly-demo:1.0.10\"" \
    k8s/deployment.yaml

# Replace the value of `spec.rules[0].verifyImages[0].attestors[0].entries[0].keys.publicKeys` in `cosign/kyverno.yaml` with the contents of `cosign/cosign.pub`

docker image build \
    --tag $REGISTRY_ADDR/$REGISTRY_USER/silly-demo:1.0.10 \
    .

# Not using `docker` but alias to `nerdctl` installed as part of Rancher Desktop
# Watch evWPib0iNgY if you're not familiar with `nerdctl` and Rancher Desktop

docker image push \
    $REGISTRY_ADDR/$REGISTRY_USER/silly-demo:1.0.10

######################################################
# Client-Side Container Image Validation With Cosign #
######################################################

cosign verify --key cosign/cosign.pub \
    $REGISTRY_ADDR/$REGISTRY_USER/silly-demo:1.0.10

#########################################################
# Enforce Usage Of Signed Container Images With Kyverno #
#########################################################

cat cosign/kyverno.yaml

kubectl --namespace production apply \
    --filename cosign/kyverno.yaml

cat k8s/deployment.yaml

kubectl --namespace production apply \
    --filename k8s/

##############################################
# Sign Container Images With Sigstore Cosign #
##############################################

cosign sign --key cosign/cosign.key \
    $REGISTRY_ADDR/$REGISTRY_USER/silly-demo:1.0.10

cosign verify --key cosign/cosign.pub \
    $REGISTRY_ADDR/$REGISTRY_USER/silly-demo:1.0.10 \
    | jq .

kubectl --namespace production apply \
    --filename k8s/

###########
# Destroy #
###########

kubectl --namespace production delete \
    --filename k8s/

# Destroy or reset the cluster
```