## Kubernetes Architecture

### Control Plane Components

- Controller Manager
- Cloud Controller Manager
- Etcd Database
- Scheduler

### Node Components

- Runtime
- Kube-proxy
- Kubelet

## AWS Concepts

### **IAM Role**
- Assumable by **anyone** who needs it
- **Short-term** credentials (token 1hr)

### **IAM User**
- Uniquely associated with **one person**
- **Long term** credentials (password / access key)

## AWS Commands

### Check the AWS User

```bash
aws sts get-caller-identity
```

### Connect to AWS using a profile

```bash
aws configure --profile developer
aws sts get-caller-identity --profile developer
```

### Connect to the EKS

```bash
aws eks update-kubeconfig --region us-east-1 --name staging-demo
```

### Assume Role

```bash
aws sts assume-role --role-arn arn:aws:iam::424432388155:role/staging-demo-eks-admin --role-session-name manager-session --profile manager
```

### To use the role

- Edit the ~/.aws/config
- Add a new profile:

```bash
[profile eks-admin]
role_arn = arn:aws:iam::424432388155:role/staging-demo-eks-admin
source_profile = manager
```

- Update the kubeconfig with the new profile:

```bash
aws eks update-kubeconfig --region us-east-1 --name staging-demo --profile eks-admin
```

## Kubernetes Commands

### Check local Kubernetes config

```bash
kubectl config view --minify
```

### Check if the user has admin privileges

```bash
kubectl auth can-i "*" "*"
```

