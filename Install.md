# Install AWX

## Kuerbnetes

Update packages
```bash
sudo apt update && sudo apt upgrade -y
```

Install kubernetes
```bash
curl -sfL https://get.k3s.io | sh -
```

Give user rights
```bash
sudo chown <user>:<user> /etc/rancher/k3s/k3s.yaml
```

Verification
```bash
kubectl version

Client Version: v1.31.4+k3s1
Kustomize Version: v5.4.2
Server Version: v1.31.4+k3s1
```

## Kustomize

Install Kustomize
```bash
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash
```

Move binairie
```bash
sudo mv kustomize /usr/local/bin
```

Create config files
```bash
vi awx.yaml

---
    apiVersion: awx.ansible.com/v1beta1
    kind: AWX
    metadata:
      name: awx
    spec:
      service_type:  nodeport
      nodeport_port: 30080
```

```bash
vi kustomization.yaml

apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - github.com/ansible/awx-operator/config/default?ref=0.28.0
  - awx.yaml

images:
  - name: quay.io/ansible/awx-operator
    newTag: 0.28.0

namespace: awx
```

Build pods
```bash
kustomize build . | kubectl apply -f -
```

Get pods
```bash
kubectl get pods --namespace awx

NAME                                              READY   STATUS    RESTARTS       AGE
awx-d47c55f4f-djfkm                               4/4     Running   0              10m
awx-operator-controller-manager-86b58ffc5-rtjzr   2/2     Running   0              10m
awx-postgres-13-0                                 1/1     Running   0              10m
```

Get admin password
```bash
kubectl get secret awx-admin-password -o jsonpath="{.data.password}" --namespace awx | base64 --decode
```

You can connect to AWX on http://localhost:30080 with admin user and password.
