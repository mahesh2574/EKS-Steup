# Amazon EKS Cluster Setup Guide

This guide explains how to create an Amazon EKS Cluster using `eksctl`.

---

# 1. Create IAM Role

Create an IAM Role with **Programmatic Access**.

Attach the following AWS Managed Policies:

- AmazonEC2FullAccess
- AmazonEKSClusterPolicy
- AWSCloudFormationFullAccess
- IAMFullAccess

(Optional for Labs)

- AmazonECRFullAccess

Create a custom policy with the following JSON:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "eks:*",
            "Resource": "*"
        }
    ]
}
```

Attach this custom policy to the IAM user.

---

# 2. Install AWS CLI

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip

sudo apt update

sudo apt install unzip -y

unzip awscliv2.zip

sudo ./aws/install

aws --version

aws configure

aws sts get-caller-identity
```

---

# 3. Install kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

chmod +x kubectl

sudo mv kubectl /usr/local/bin/

kubectl version --client
```

---

# 4. Install eksctl

```bash
curl --silent --location \
"https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" \
| tar xz -C /tmp

sudo mv /tmp/eksctl /usr/local/bin

eksctl version
```

---

# 5. Create EKS Cluster

```bash
eksctl create cluster \
  --name my-eks22 \
  --region ap-south-1 \
  --zones ap-south-1a,ap-south-1b \
  --version 1.30 \
  --without-nodegroup
```

---

# 6. Associate IAM OIDC Provider

```bash
eksctl utils associate-iam-oidc-provider \
  --cluster my-eks22 \
  --region ap-south-1 \
  --approve
```

---

# 7. Create Managed Node Group

```bash
eksctl create nodegroup \
  --cluster my-eks22 \
  --region ap-south-1 \
  --name node2 \
  --node-type t3.medium \
  --nodes 3 \
  --nodes-min 2 \
  --nodes-max 4 \
  --node-volume-size 20 \
  --managed \
  --ssh-access \
  --ssh-public-key Key \
  --asg-access \
  --full-ecr-access \
  --alb-ingress-access
```

---

# 8. Configure kubectl

```bash
aws eks update-kubeconfig \
  --region ap-south-1 \
  --name my-eks22
```

---

# 9. Verify Cluster

```bash
kubectl get nodes

kubectl get pods -A

kubectl get ns
```

---

# 10. Create Namespace

```bash
kubectl create namespace webapps
```

---

# 11. Jenkins Service Account

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: webapps
```

---

# 12. Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: webapps
  name: app-role
rules:
- apiGroups:
  - ""
  - apps
  - autoscaling
  - batch
  - extensions
  - policy
  - rbac.authorization.k8s.io
  resources:
  - pods
  - secrets
  - configmaps
  - daemonsets
  - deployments
  - events
  - endpoints
  - horizontalpodautoscalers
  - ingresses
  - jobs
  - limitranges
  - namespaces
  - nodes
  - persistentvolumes
  - persistentvolumeclaims
  - replicasets
  - replicationcontrollers
  - resourcequotas
  - serviceaccounts
  - services
  verbs:
  - get
  - list
  - watch
  - create
  - update
  - patch
  - delete
```

---

# 13. RoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-rolebinding
  namespace: webapps

roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: app-role

subjects:
- kind: ServiceAccount
  name: jenkins
  namespace: webapps
```

---

# 14. Generate Service Account Token

```bash
kubectl create token jenkins -n webapps
```

---

# 15. Verify Resources

```bash
kubectl get sa -n webapps

kubectl get role -n webapps

kubectl get rolebinding -n webapps

kubectl get nodes

kubectl get pods -A
```

---

# 16. Delete EKS Cluster

```bash
eksctl delete cluster \
  --name my-eks22 \
  --region ap-south-1
```
