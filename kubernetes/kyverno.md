```bash
# Source: https://gist.github.com/9774cf9d4d39b02bb01db928d9f3a0d2

####################################################
# Kubernetes Native Policy Management With Kyverno #
# https://youtu.be/DREjzfTzNpA                     #
####################################################

# Referenced videos:
# - How to apply policies in Kubernetes using Open Policy Agent (OPA) and Gatekeeper: https://youtu.be/14lGc7xMAe4
# - K3d - How to run Kubernetes cluster locally using Rancher k3s: https://youtu.be/mCesuGk-Fks

#########
# Setup #
#########

git clone https://github.com/vfarcic/kyverno-demo

cd kyverno-demo

# Please watch https://youtu.be/mCesuGk-Fks if you are not familiar with k3d.
# It could be any other Kubernetes cluster. It does not have to be k3d.
k3d cluster create --config k3d.yaml

kubectl create \
    --filename https://raw.githubusercontent.com/kyverno/kyverno/main/config/install.yaml

kubectl --namespace kyverno \
    rollout status \
    deployment kyverno

cp app/orig.yaml app/app.yaml

kubectl create namespace production

kubectl apply --filename policies/

#####################
# Disallow NodePort #
#####################

cat app/app.yaml

kubectl --namespace production \
    apply --filename app/app.yaml

# Open `app/app.yaml` and change Service `spec.type` to `ClusterIP`

cat policies/block-node-port.yaml

###########################
# Require resource limits #
###########################

kubectl --namespace production \
    apply --filename app/app.yaml

# Open `app/app.yaml` and add `spec.template.spec.containers[].resources` with `requests` and `limits`.

cat policies/container-must-have-limits.yaml

#######################
# Disallow latest tag #
#######################

kubectl --namespace production \
    apply --filename app/app.yaml

# Open `app/app.yaml` and change `spec.template.spec.containers[].image` to `vfarcic/devops-toolkit-series:3.0.0`

cat policies/image-not-latest.yaml

#############
# Reporting #
#############

kubectl get policyreport -A

kubectl --namespace production \
    describe \
    policyreport polr-ns-production

kubectl --namespace production \
    apply --filename app/app.yaml

kubectl get policyreport -A

kubectl --namespace production \
    describe \
    policyreport polr-ns-production

#####################################
# Policy catalog and other features #
#####################################

# Open https://kyverno.io/policies/

################
# Random stuff #
################

# Open https://kyverno.io/policies/pod-security/restricted/deny-privilege-escalation/deny-privilege-escalation/

# Open https://kyverno.io/docs/writing-policies/generate/

###########
# Destroy #
###########

k3d cluster delete devops-toolkit
```