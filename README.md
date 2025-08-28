# kubernetes
WordPress + MySQL on Kubernetes (AWS)

This project demonstrates how to deploy a WordPress application with a MySQL backend on a custom Kubernetes cluster running on AWS EC2 instances. The setup leverages AWS storage solutions for persistent data and ensures scalability and high availability.

Project Architecture

Kubernetes Cluster: 3 EC2 instances (self-managed with kubeadm).

MySQL Deployment:

  Deployed as a Kubernetes Deployment with a headless Service.

  Credentials stored securely in Kubernetes Secrets.

  Persistent storage provisioned dynamically using Amazon EBS via ebs.csi.aws.com.

WordPress Deployment:

  Deployed with multiple replicas for scalability.

  Persistent storage mounted on Amazon EFS via efs.csi.aws.com to share data across replicas.

  Database credentials injected through Secrets + environment variables.

  Configured to disable SSL in DB connection (MYSQL_CLIENT_FLAGS=0) to avoid self-signed certificate issues.


Networking:

  Separate VPCs for MySQL and WordPress, connected via VPC Peering.

  WordPress exposed using a NodePort Service for external access.

Key Features

  Dynamic storage provisioning with EBS (for MySQL) and EFS (for WordPress shared files).

  Secure DB credentials management using Kubernetes Secrets.

  Scalable WordPress deployment with replicas sharing persistent data.

  AWS-native networking with VPC peering for cross-service communication.

  Troubleshooting and hardening for SSL/TLS issues with MySQL + WordPress.

How It Works

  MySQL stores structured data in an EBS-backed Persistent Volume Claim (PVC).

  WordPress pods mount an EFS-backed PVC, ensuring media uploads and WordPress core files are shared across replicas.

  WordPress connects to MySQL via a Kubernetes Service (mysql-service) using credentials stored in Secrets.

  External users access WordPress through a NodePort (31031) exposed by the WordPress Service.

Deployment Flow

  Create StorageClasses for EBS (MySQL) and EFS (WordPress).

  Apply PersistentVolumeClaims for both.

  Deploy MySQL + Service + Secret.

Deploy WordPress + Service + Secret, mounting the EFS PVC.

Access WordPress via http://<EC2_Node_IP>:31031.
