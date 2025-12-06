# MetalLB on Bare-Metal Kubernetes

Complete guide for deploying MetalLB on traditional bare-metal Kubernetes clusters.

## 📖 Overview

This setup uses MetalLB with your own network IP address range to provide LoadBalancer services for bare-metal Kubernetes clusters. The configuration uses Layer 2 mode for ARP-based load balancing.

## 📋 Prerequisites

- Kubernetes cluster running on bare-metal servers
- `kubectl` configured and connected to your cluster
- Available IP address range from your network
- Layer 2 network connectivity between nodes
- Network administrator access (if needed for IP allocation)

## 🚀 Installation

### 1. Create MetalLB Namespace

```bash
kubectl create namespace metallb-system
```

### 2. Deploy MetalLB Components

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.9/config/manifests/metallb-native.yaml
```

### 3. Verify Installation

```bash
kubectl get pods -n metallb-system
```

Wait until all pods (controller and speakers) are in `Running` status.

## ⚙️ Configuration

### 4. Configure IP Address Pool

Update `address-pool.yml` with your available IP range:

```yaml
---
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.200.220-192.168.200.230  # Replace with your IP range
```

Apply the configuration:

```bash
kubectl apply -f address-pool.yml
```

### 5. Configure L2 Advertisement

Apply the L2 advertisement configuration:

```bash
kubectl apply -f l2-advertisement.yml
```

### 6. Verify Configuration

```bash
kubectl get ipaddresspools.metallb.io -n metallb-system
kubectl get l2advertisements.metallb.io -n metallb-system
```

## 🧪 Testing

Deploy the example Nginx application:

```bash
kubectl apply -f ../examples/nginx-demo.yml
```

Check if the service received an IP:

```bash
kubectl get svc nginx-demo
```

Test the endpoint:

```bash
curl http://<EXTERNAL-IP>
```

## 📁 Configuration Files

### address-pool.yml

Defines the IP address pool for MetalLB:

```yaml
---
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default-pool
  namespace: metallb-system
spec:
  addresses:
  # Single IP
  - 192.168.200.220/32
  # Or IP range
  - 192.168.200.221-192.168.200.230
  # Or CIDR notation
  - 192.168.201.0/24
```

### l2-advertisement.yml

Configures Layer 2 advertisement:

```yaml
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: default-l2
  namespace: metallb-system
spec:
  ipAddressPools:
  - default-pool
```

## 🔧 Advanced Configuration

### Multiple IP Pools

Create multiple pools for different purposes:

```yaml
---
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: production-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.200.100-192.168.200.150
---
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: development-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.200.200-192.168.200.250
```

### Service Annotations

Assign specific pool to a service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  annotations:
    metallb.universe.tf/address-pool: production-pool
spec:
  type: LoadBalancer
  # ...
```

### Sharing IPs

Allow multiple services to share the same IP:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: service1
  annotations:
    metallb.universe.tf/allow-shared-ip: "shared-key"
spec:
  type: LoadBalancer
  # ...
---
apiVersion: v1
kind: Service
metadata:
  name: service2
  annotations:
    metallb.universe.tf/allow-shared-ip: "shared-key"
spec:
  type: LoadBalancer
  # ...
```

## 📝 Important Notes

- **IP Range**: Ensure the IP range doesn't conflict with DHCP or other assignments
- **Network Access**: All nodes must be on the same Layer 2 network
- **ARP**: MetalLB uses ARP to announce IPs to the network
- **Failover**: Automatic failover when nodes fail

## 🔍 Troubleshooting

### Service Stuck in Pending

Check MetalLB controller logs:
```bash
kubectl logs -n metallb-system deployment/controller
```

### IP Not Reachable

Verify network connectivity:
```bash
# From a node
arping -I <interface> <assigned-ip>

# Check speaker logs
kubectl logs -n metallb-system -l component=speaker
```

### ARP Not Working

Check L2 advertisement:
```bash
kubectl describe l2advertisement default-l2 -n metallb-system
```

## 🧹 Cleanup

Remove test application:
```bash
kubectl delete -f ../examples/nginx-demo.yml
```

Remove MetalLB configuration:
```bash
kubectl delete -f l2-advertisement.yml
kubectl delete -f address-pool.yml
```

Uninstall MetalLB (optional):
```bash
kubectl delete -f https://raw.githubusercontent.com/metallb/metallb/v0.14.9/config/manifests/metallb-native.yaml
kubectl delete namespace metallb-system
```

## 📚 Additional Resources

- [MetalLB Official Documentation](https://metallb.universe.tf/)
- [Layer 2 Configuration](https://metallb.universe.tf/configuration/_advanced_l2_configuration/)
- [MetalLB Concepts](https://metallb.universe.tf/concepts/)

---

**Related Setups:**
- [Hetzner Cloud Setup](../hetzner/README.md) - For cloud deployments with Floating IPs
