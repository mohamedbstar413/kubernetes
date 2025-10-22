# 🚀 EKS Multi-Service Application

A production-ready Kubernetes application deployed on Amazon EKS, featuring MongoDB, Mongo Express, and Nginx services with ingress routing and persistent storage.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Components](#components)
- [Deployment](#deployment)
- [Configuration](#configuration)
- [Access](#access)
- [Troubleshooting](#troubleshooting)

---

## 🎯 Overview

This project demonstrates a complete multi-tier application deployment on Amazon EKS with the following features:

- **MongoDB Database** with persistent EBS storage
- **Mongo Express** web interface for database management
- **Nginx** web server for serving static content
- **NGINX Ingress Controller** for intelligent routing
- **AWS EBS CSI Driver** for dynamic volume provisioning
- **RBAC** security configuration

---

## 🏗️ Architecture

```
Internet → Load Balancer → NGINX Ingress Controller
                              ↓
                    ┌─────────┴─────────┐
                    ↓                   ↓
              /web route           / route
                    ↓                   ↓
            Nginx Service      Express Service
              (Port 80)          (Port 8081)
                                       ↓
                                 Mongo Service
                                  (Port 27017)
                                       ↓
                                  EBS Volume
                                   (10Gi)
```

### Traffic Flow

| Path | Service | Backend | Description |
|------|---------|---------|-------------|
| `/` | express-service | Mongo Express | Database management UI |
| `/web` | nginx-service | Nginx | Static web content |

---

## ✅ Prerequisites

- AWS Account with EKS cluster provisioned
- `kubectl` configured with cluster access
- AWS CLI installed and configured
- Helm 3.x (for ingress controller installation)
- EBS CSI Driver installed in the cluster

### Required AWS Resources

- **EKS Cluster** running Kubernetes 1.24+
- **EBS Volume**: `vol-05eb80ad0600b4d97` (pre-created)
- **OIDC Provider**: Configured for the cluster
- **IAM Role**: `ebs-controller-role` with EBS permissions

---

## 🔧 Components

### 1. **MongoDB Database**
- **Image**: `mongo:6.0`
- **Replicas**: 1
- **Storage**: 5Gi persistent volume (EBS)
- **Credentials**: Stored in Kubernetes Secret
- **Port**: 27017

### 2. **Mongo Express**
- **Image**: `mongo-express`
- **Replicas**: 3
- **Type**: NodePort (31031)
- **Purpose**: Web-based MongoDB admin interface

### 3. **Nginx Web Server**
- **Image**: `nginx`
- **Replicas**: 1
- **Port**: 80
- **Purpose**: Serve static content

### 4. **NGINX Ingress Controller**
- **Type**: LoadBalancer (AWS ELB)
- **SSL**: Disabled (can be enabled)
- **Scheme**: Internet-facing

### 5. **Storage**
- **StorageClass**: `mongo-sc`
- **Provisioner**: AWS EBS CSI Driver
- **Volume Mode**: Immediate binding
- **Access Mode**: ReadWriteOnce

---

## 🚀 Deployment

### Step 1: Deploy Storage Components

```bash
# Create StorageClass
kubectl apply -f storage-class.yaml

# Create PersistentVolume
kubectl apply -f persistent-volume.yaml

# Create PersistentVolumeClaim
kubectl apply -f persistent-volume-claim.yaml
```

### Step 2: Deploy Secrets

```bash
# Create MongoDB credentials
kubectl apply -f mongo-secret.yaml
```

> **Note**: Default credentials are `admin/Pa55w0rd`. Change these in production!

### Step 3: Deploy Database

```bash
# Deploy MongoDB
kubectl apply -f mongo-deployment.yaml
kubectl apply -f mongo-service.yaml

# Wait for MongoDB to be ready
kubectl wait --for=condition=ready pod -l app=mongo --timeout=300s
```

### Step 4: Deploy Applications

```bash
# Deploy Mongo Express
kubectl apply -f express-deployment.yaml
kubectl apply -f express-service.yaml

# Deploy Nginx
kubectl apply -f nginx-deployment.yaml
kubectl apply -f nginx-service.yaml
```

### Step 5: Configure RBAC and Ingress

```bash
# Create ServiceAccount and RBAC
kubectl apply -f nginx-ingress-rbac.yaml

# Deploy Ingress
kubectl apply -f ingress.yaml
```

### Step 6: Verify Deployment

```bash
# Check all pods
kubectl get pods

# Check services
kubectl get svc

# Check ingress
kubectl get ingress

# Get LoadBalancer URL
kubectl get svc -n ingress-nginx
```

---

## ⚙️ Configuration

### MongoDB Credentials

Edit `mongo-secret.yaml`:

```yaml
data:
  username: <base64-encoded-username>
  password: <base64-encoded-password>
```

Generate base64 values:
```bash
echo -n "your-username" | base64
echo -n "your-password" | base64
```

### Scaling Applications

```bash
# Scale Mongo Express
kubectl scale deployment express-deploy --replicas=5

# Scale Nginx
kubectl scale deployment nginx-deploy --replicas=3
```

### Storage Configuration

Modify the PVC in `persistent-volume-claim.yaml` to request more storage:

```yaml
resources:
  requests:
    storage: 10Gi  # Increase as needed
```

---

## 🌐 Access

### Get LoadBalancer URL

```bash
kubectl get ingress multi-service-ingress -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

### Access Points

- **Mongo Express**: `http://<LOAD_BALANCER_URL>/`
- **Nginx**: `http://<LOAD_BALANCER_URL>/web`

### NodePort Access (Optional)

If not using ingress:
```bash
kubectl get nodes -o wide
# Access Mongo Express at: http://<NODE_IP>:31031
```

---

## 🔍 Troubleshooting

### Pods Not Starting

```bash
# Check pod status
kubectl describe pod <pod-name>

# Check logs
kubectl logs <pod-name>

# Check events
kubectl get events --sort-by='.lastTimestamp'
```

### Storage Issues

```bash
# Check PV/PVC binding
kubectl get pv,pvc

# Describe PVC
kubectl describe pvc mongo-pvc2

# Verify EBS CSI driver
kubectl get pods -n kube-system -l app=ebs-csi-controller
```

### Ingress Not Working

```bash
# Check ingress status
kubectl describe ingress multi-service-ingress

# Check ingress controller logs
kubectl logs -n ingress-nginx -l app.kubernetes.io/component=controller

# Verify service endpoints
kubectl get endpoints
```

### Database Connection Issues

```bash
# Test MongoDB connectivity
kubectl run -it --rm debug --image=mongo:6.0 --restart=Never -- mongosh mongodb://admin:Pa55w0rd@mongo-service:27017

# Check MongoDB logs
kubectl logs -l app=mongo,tier=db
```

---

## 📊 Monitoring

### Check Resource Usage

```bash
# Pod resources
kubectl top pods

# Node resources
kubectl top nodes
```

### View Application Logs

```bash
# MongoDB logs
kubectl logs -l app=mongo,tier=db --tail=100

# Mongo Express logs
kubectl logs -l app=mongo,tier=server --tail=100

# Nginx logs
kubectl logs -l app=nginx --tail=100
```

---

## 🔒 Security Considerations

- [ ] Change default MongoDB credentials
- [ ] Enable SSL/TLS on ingress
- [ ] Implement network policies
- [ ] Use secrets management (AWS Secrets Manager)
- [ ] Enable pod security policies
- [ ] Restrict service account permissions
- [ ] Implement backup strategy for MongoDB

---

## 🛠️ Maintenance

### Backup MongoDB

```bash
# Exec into MongoDB pod
kubectl exec -it <mongo-pod-name> -- mongodump --out /backup

# Copy backup to local
kubectl cp <mongo-pod-name>:/backup ./mongodb-backup
```

### Update Application

```bash
# Update image
kubectl set image deployment/express-deploy express=mongo-express:latest

# Rollout status
kubectl rollout status deployment/express-deploy

# Rollback if needed
kubectl rollout undo deployment/express-deploy
```

---

## 📝 Notes

- **EBS Volume ID**: Update `vol-05eb80ad0600b4d97` with your volume ID
- **OIDC Provider**: Update ARN in `ebs-csi-controller-sa.yaml`
- **IAM Role**: Ensure `ebs-controller-role` has proper permissions
- **Region**: Configured for `us-east-1`

---

## 📚 Additional Resources

- [EKS Documentation](https://docs.aws.amazon.com/eks/)
- [NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/)
- [AWS EBS CSI Driver](https://github.com/kubernetes-sigs/aws-ebs-csi-driver)
- [MongoDB on Kubernetes](https://www.mongodb.com/kubernetes)

---

## 📄 License

This project is open source and available under the MIT License.

---

## 👥 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

---

**Built with ❤️ using Kubernetes & AWS EKS**