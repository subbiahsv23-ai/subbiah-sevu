# Kubernetes Persistent Volume Configuration

This directory contains Kubernetes configuration files for setting up persistent storage with Apache deployment.

## Files

### pv.yaml
- **PersistentVolume (PV)** configuration
- Storage: 2Gi
- Access Mode: ReadWriteOnce
- Storage Class: manual
- Host Path: /mnt/data

### pvc.yaml
- **PersistentVolumeClaim (PVC)** configuration
- Claims 2Gi of storage from the PersistentVolume
- Matches the PV storage class (manual)

### deployment.yaml
- **Deployment** configuration for Apache HTTP Server
- Replicas: 3
- Mounts persistent storage at: /usr/local/apache2/htdocs
- Resource limits and requests configured

## Usage

Apply these configurations in order:

```bash
kubectl apply -f pv.yaml
kubectl apply -f pvc.yaml
kubectl apply -f deployment.yaml
```

## Reference

https://kubernetes.io/docs/tutorials/configuration/configure-persistent-volume-storage/
