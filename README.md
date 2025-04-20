# Advanced Kubernetes Storage Solutions

This project demonstrates advanced storage patterns and solutions in Kubernetes, including various storage classes, CSI implementations, data protection strategies, and dynamic provisioning.


## Project Contents

This project covers the following advanced storage topics:

1. **Storage Class Implementation**: Multiple storage options for different workloads
2. **CSI Drivers**: Custom Storage Interface integration
3. **StatefulSet Patterns**: Advanced data persistence for stateful applications
4. **Backup and Recovery**: Implementing Velero for Kubernetes backup
5. **Storage Operators**: Rook-Ceph for software-defined storage
6. **Data Encryption**: Securing persistent data at rest

## Storage Class Implementation

This section demonstrates implementing and using different storage classes for various workload requirements:

- Performance-optimized SSD storage
- Cost-optimized HDD storage
- In-memory storage for high-speed temporary data
- Local storage for node-specific workloads
- Replicated storage for high availability

### Example Storage Classes:

* `premium-ssd.yaml`: High-performance SSD storage
* `standard-hdd.yaml`: Cost-effective HDD storage
* `in-memory.yaml`: RAM-based ephemeral storage
* `local-storage.yaml`: Node-local storage
* `replicated-storage.yaml`: Highly available storage with replication

## CSI Driver Implementation

This section demonstrates the implementation and configuration of Container Storage Interface (CSI) drivers:

- Custom CSI driver installation
- Volume snapshots and cloning
- Volume expansion
- Raw block volumes
- Inline ephemeral volumes

### Example CSI Configurations:

* `aws-ebs-csi-driver.yaml`: AWS EBS CSI driver installation
* `azure-disk-csi-driver.yaml`: Azure Disk CSI driver installation 
* `gcp-pd-csi-driver.yaml`: GCP Persistent Disk CSI driver installation
* `ceph-csi-rbd.yaml`: Ceph RBD CSI driver installation
* `snapshot-controller.yaml`: Volume snapshot controller installation
* `volumesnapshot-class.yaml`: Volume snapshot class definition

## StatefulSet Patterns

Advanced StatefulSet implementations for data-intensive applications:

- Database clusters (MySQL, PostgreSQL, MongoDB)
- Messaging systems (Kafka, RabbitMQ)
- Distributed caches (Redis, Memcached)
- Customized volume claim templates
- Anti-affinity rules for high availability

### Example StatefulSet Implementations:

* `mysql-cluster.yaml`: Multi-node MySQL cluster with primary-replica replication
* `postgresql-ha.yaml`: High-availability PostgreSQL cluster
* `mongodb-sharded.yaml`: Sharded MongoDB cluster
* `kafka-cluster.yaml`: Multi-broker Kafka cluster with ZooKeeper
* `elasticsearch-cluster.yaml`: Elasticsearch cluster with dedicated data, master, and client nodes

## Backup and Recovery

Implementation of Velero for Kubernetes backup and disaster recovery:

- Full cluster backup
- Namespace-scoped backups
- Schedule-based backup plans
- PV/PVC backup and restore
- Cross-cluster migration

### Example Backup Configurations:

* `velero-installation.yaml`: Velero installation with provider plugins
* `backup-storage-location.yaml`: Defining backup storage locations
* `volume-snapshot-location.yaml`: Defining volume snapshot locations
* `scheduled-backup.yaml`: Recurring backup schedule
* `backup-hooks.yaml`: Pre/post backup execution hooks
* `selective-restore.yaml`: Granular restore options

## Storage Operators

Implementing Rook-Ceph for software-defined storage within Kubernetes:

- Ceph storage cluster
- Block, file, and object storage
- Dynamic provisioning
- Storage monitoring
- Cluster expansion

### Example Rook-Ceph Configurations:

* `rook-operator.yaml`: Rook operator installation
* `ceph-cluster.yaml`: Ceph cluster configuration
* `ceph-block-pool.yaml`: RBD pool configuration
* `ceph-filesystem.yaml`: CephFS configuration
* `ceph-object-store.yaml`: Ceph Object Gateway (S3-compatible)
* `storage-dashboard.yaml`: Monitoring dashboard setup

## Data Encryption

Securing persistent data through encryption:

- Volume-level encryption
- Kubernetes Secrets integration
- Key management with Vault
- LUKS encryption
- CSI encryption support

### Example Encryption Implementations:

* `encryption-provider-config.yaml`: Kubernetes API server encryption configuration
* `vault-integration.yaml`: HashiCorp Vault integration
* `encrypted-pv-class.yaml`: Storage class with encryption enabled
* `vault-csi-provider.yaml`: Vault CSI provider for secrets
* `kmip-integration.yaml`: Key management service integration

## Prerequisites

- Kubernetes cluster v1.22+
- kubectl installed and configured
- Helm v3+
- Cloud provider access (for some examples)
- Sufficient cluster resources (recommend at least 6 nodes)

## Setup Instructions

### 1. Create Namespaces

```bash
kubectl apply -f namespaces.yaml
```

### 2. Deploy Storage Classes

```bash
kubectl apply -f storage-classes/
```

### 3. Install CSI Drivers

Choose the appropriate CSI driver based on your environment:

```bash
# For AWS
kubectl apply -f csi-drivers/aws-ebs-csi-driver.yaml

# For Azure
kubectl apply -f csi-drivers/azure-disk-csi-driver.yaml

# For GCP
kubectl apply -f csi-drivers/gcp-pd-csi-driver.yaml

# For on-premises Ceph
kubectl apply -f csi-drivers/ceph-csi-rbd.yaml
```

Install the snapshot controller:

```bash
kubectl apply -f csi-drivers/snapshot-controller.yaml
kubectl apply -f csi-drivers/volumesnapshot-class.yaml
```

### 4. Deploy Stateful Applications

```bash
kubectl apply -f statefulsets/
```

### 5. Set Up Backup and Recovery

Install Velero with the appropriate provider plugin:

```bash
# Using Helm
helm repo add vmware-tanzu https://vmware-tanzu.github.io/helm-charts
helm repo update
helm install velero vmware-tanzu/velero \
  --namespace velero \
  --create-namespace \
  --set-file credentials.secretContents.cloud=./credentials-velero \
  --set configuration.provider=aws \
  --set configuration.backupStorageLocation.bucket=<YOUR_BUCKET> \
  --set configuration.backupStorageLocation.config.region=<YOUR_REGION>
```

Apply backup configurations:

```bash
kubectl apply -f backup/
```

### 6. Deploy Rook-Ceph

```bash
kubectl apply -f rook-ceph/operator.yaml
kubectl apply -f rook-ceph/cluster.yaml
kubectl apply -f rook-ceph/storage-classes.yaml
```

### 7. Configure Data Encryption

```bash
kubectl apply -f encryption/
```

## Testing and Verification

### Testing Storage Classes

```bash
# Create test PVCs with different storage classes
kubectl apply -f examples/test-pvcs.yaml

# Check the provisioned PVs
kubectl get pv,pvc
```

### Testing CSI Features

```bash
# Create a volume snapshot
kubectl apply -f examples/volume-snapshot.yaml

# Restore from snapshot
kubectl apply -f examples/restore-from-snapshot.yaml

# Expand a volume
kubectl apply -f examples/volume-expansion.yaml
```

### Testing StatefulSets

```bash
# Deploy a test database
kubectl apply -f examples/mysql-test.yaml

# Check pod creation and volume attachment
kubectl get pods,pvc

# Test database replication
kubectl exec -it mysql-0 -- mysql -u root -p -e "CREATE DATABASE test;"
kubectl exec -it mysql-1 -- mysql -u root -p -e "SHOW DATABASES;"
```

### Testing Backup and Restore

```bash
# Create a backup
velero backup create test-backup --include-namespaces=default

# Simulate a disaster
kubectl delete namespace default

# Restore from backup
velero restore create --from-backup test-backup
```

### Testing Rook-Ceph

```bash
# Check Ceph status
kubectl -n rook-ceph exec -it $(kubectl -n rook-ceph get pod -l app=rook-ceph-tools -o jsonpath='{.items[0].metadata.name}') -- ceph status

# Test block storage
kubectl apply -f examples/rook-ceph-block-test.yaml

# Test file storage
kubectl apply -f examples/rook-ceph-filesystem-test.yaml

# Test object storage
kubectl apply -f examples/rook-ceph-object-test.yaml
```

## Performance Benchmarking

This project includes tools for benchmarking storage performance:

- IOPing for latency measurements
- FIO for throughput testing
- Database benchmarking
- Comparative analysis scripts

Run performance tests:

```bash
kubectl apply -f benchmarks/
kubectl logs -f storage-benchmark
```

## Monitoring Storage

Monitor storage resources using:

- Prometheus metrics
- Grafana dashboards for storage
- Ceph dashboard for Rook-Ceph
- Custom storage alerts

Access the dashboards:

```bash
# Grafana dashboard
kubectl port-forward svc/grafana 3000:3000 -n monitoring

# Ceph dashboard
kubectl port-forward svc/rook-ceph-mgr-dashboard 7000:7000 -n rook-ceph
```

## Troubleshooting Guide

### Common PV/PVC Issues

```bash
# Check PVC status
kubectl describe pvc <pvc-name>

# Check related PV
kubectl describe pv <pv-name>

# Check storage provisioner pods
kubectl get pods -n kube-system | grep csi
```

### StatefulSet Troubleshooting

```bash
# Check StatefulSet events
kubectl describe statefulset <statefulset-name>

# Check pod mount points
kubectl exec -it <pod-name> -- df -h

# Check filesystem on volumes
kubectl exec -it <pod-name> -- ls -la /data
```

### CSI Driver Issues

```bash
# Check CSI driver pods
kubectl get pods -n <csi-namespace>

# View CSI driver logs
kubectl logs -n <csi-namespace> <csi-controller-pod>

# Check CSI node plugin logs
kubectl logs -n <csi-namespace> <csi-node-pod>
```

### Rook-Ceph Troubleshooting

```bash
# Check Ceph health
kubectl -n rook-ceph exec -it <tools-pod> -- ceph health detail

# Check OSD status
kubectl -n rook-ceph exec -it <tools-pod> -- ceph osd status

# View Ceph logs
kubectl -n rook-ceph logs <osd-pod>
```

## Best Practices

- Use the right storage class for each workload
- Implement proper backup strategies
- Use node anti-affinity for StatefulSets
- Monitor storage usage and performance
- Plan for disaster recovery
- Test restore procedures regularly

See `docs/best-practices.md` for detailed recommendations.

## Contributing

1. Fork this repo
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push your branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request
