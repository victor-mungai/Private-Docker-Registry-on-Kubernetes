# Private-Docker-Registry-on-Kubernetes
A production-oriented setup for running a private Docker Registry inside Kubernetes, secured with TLS and integrated with Kubernetes image pulling mechanisms.

This project focuses on registry deployment, authentication, storage, and Kubernetes integration, following patterns commonly used in real-world clusters.
#Project Structure
```
.
├── application
│   └── pod.yaml
├── auth
│   └── htpasswd
├── certs
│   ├── csr.yaml
│   ├── openssl.conf
│   ├── registry.crt
│   ├── registry.csr
│   └── registry.key
├── deployment
│   ├── deployment.yaml
│   └── svc.yaml
└── storage
    └── storage.yaml

```
#Architecture Overview
```
Docker Client / Kubernetes Nodes
        |
        |  HTTPS (TLS)
        v
Kubernetes Service
        |
        v
Docker Registry (registry:2)
        |
        v
Persistent Storage

```
#Objectives
Run a self-hosted Docker registry inside Kubernetes

Secure registry traffic over HTTPS

Enable authenticated image push and pull

Allow Kubernetes workloads to pull private images

Persist images using Kubernetes storage primitives

#Components Breakdown
*deployment/*

Contains the core Kubernetes resources for running the registry.

Deployment

Runs registry:2

Mounts TLS certificates

Uses persistent storage

Enables basic authentication

Service

Exposes the registry to clients and cluster nodes

*storage/*

Defines persistent storage for registry image layers.

Ensures images survive pod restarts

Designed to be backed by any Kubernetes-supported storage class

*auth/*

Handles registry authentication.

Uses htpasswd for basic auth

Integrated directly into the registry container

Compatible with docker login

*certs/*

Contains TLS artifacts used by the registry.

In production, certificates are expected to be issued and managed by a trusted Certificate Authority such as:

Let’s Encrypt (via cert-manager)

Cloud provider managed CAs

This directory represents how certificates are consumed by the registry, not how production TLS should be manually managed.

*application/*

Example Kubernetes workload that pulls an image from the private registry.

Validates registry accessibility

Verifies authentication and TLS configuration

Demonstrates imagePullSecrets usage

#Security Model

Registry served over HTTPS

TLS certificates signed by a trusted CA in production

Registry protected with authentication

Kubernetes workloads authenticate using imagePullSecrets

No insecure registry configuration required on nodes

#Deployment Flow

Create required resources
```
kubectl apply -f storage/
kubectl apply -f auth/
kubectl apply -f certs/
```

Deploy the registry
```
kubectl apply -f deployment/
```

Authenticate
```
docker login my-registry.example.com
```

Push an image
```
docker tag busybox my-registry.example.com/my-busybox:v1
docker push my-registry.example.com/my-busybox:v1
```

Deploy application
```
kubectl apply -f application/
```
#Validation Checklist

Registry reachable over HTTPS

docker login succeeds

Images push and pull correctly

Pods start without ErrImagePull

Images persist after pod restarts

#Tech Stack

Kubernetes

Docker Registry v2

TLS (CA-signed)

Basic Authentication

Persistent Volumes

# Production Considerations

Use Ingress + cert-manager for automated TLS

Protect registry access with RBAC and network policies

Monitor storage utilization

Consider HA registry setups for scale

Back up registry data regularly

#Why This Project Matters

Private registries are foundational infrastructure in Kubernetes environments.
This project demonstrates practical knowledge of:

Secure container distribution

Kubernetes storage and networking

Authentication flows

Production-grade deployment patterns
