1. Tips and tricks
Ref: https://faun.pub/how-to-pass-the-certified-kubernetes-administrator-cka-exam-without-any-stress-57f67e7b6862

Cncf curriculum: https://github.com/cncf/curriculum


- Utils
use `yq` to filter yaml and json output
```bash 
cat sphere.yaml| yq -r .spec.template.spec.containers.0.name
```

`jq` example
```bash
k get pod busybox -ojson|jq '.spec.containers[0].image'
```


2. Enable auto-completion
```bash
source <(kubectl completion bash)  
echo "source <(kubectl completion bash)" >> ~/.bashrc
```

3. Create aliases
```bash
alias k=kubectl  
alias kgp="k get pod"  
alias kgd="k get deploy"  
alias kgs="k get svc" 
alias kgn="k get nodes"  
alias kd="k describe"  
alias kge="k get events --sort-by='.metadata.creationTimestamp' |tail -8"

export do="--dry-run=client -o yaml"
export now="--force --grace-period 0"
```

Dry-run and output to yaml
```bash
k run nginx --image=nginx $do > pod.yaml
```

Force deletion
```bash
export now="--force --grace-period 0"
k delete pod test $now
```

## Kubernetes certs and keys

Kubernetes ca.certs and keys are stored at
```bash
# kubernetes certs
ls /etc/kubernetes/pki/

# etcd certs and keys
ls /etc/kubernetes/pki/etcd/
```