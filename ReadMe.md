<div align="center">
  <img src="https://metallb.universe.tf/images/logo/metallb-blue.png" alt="MetalLB Logo" width="200"/>
  <h1>MetalLB Load Balancer for Kubernetes</h1>
  
  <p>
    <img src="https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white" alt="Kubernetes"/>
    <img src="https://img.shields.io/badge/MetalLB-Latest-blue?style=for-the-badge" alt="MetalLB"/>
    <img src="https://img.shields.io/badge/License-MIT-green?style=for-the-badge" alt="License"/>
  </p>
  
  <p><strong>Production-ready MetalLB configurations for bare-metal and cloud Kubernetes clusters</strong></p>
</div>

---

## 📖 Overview

MetalLB is a load-balancer implementation for bare-metal Kubernetes clusters, using standard routing protocols. This repository provides production-ready configurations for different deployment scenarios.

### ✨ Features

- 🚀 **Multiple Deployment Options** - Bare-metal and cloud-specific configurations
- 🔧 **Layer 2 Mode** - Simple ARP/NDP-based load balancing
- 📦 **Production Ready** - Tested configurations
- 📚 **Comprehensive Documentation** - Detailed guides and examples
- 🎯 **Flexible IP Management** - Multiple IP pool configurations

## 📂 Repository Structure

```
metallb-k8s/
├── README.md                    # This file
├── bare-metal/                  # Traditional bare-metal setup
│   ├── README.md
│   ├── address-pool.yml
│   └── l2-advertisement.yml
├── hetzner/                     # Hetzner Cloud specific setup
│   ├── README.md
│   ├── address-pool.yml
│   ├── l2-advertisement.yml
│   └── docs/
│       ├── floating-ip-setup.md
└── examples/                    # Example applications
    └── nginx-demo.yml
```

## 🚀 Quick Start

Choose your deployment scenario:

### 🖥️ Bare-Metal Deployment

For traditional bare-metal Kubernetes clusters with physical network infrastructure.

**[→ Go to Bare-Metal Setup](bare-metal/README.md)**

**Use case:**
- On-premise data centers
- Physical servers with direct network access
- Static IP ranges from your network

### ☁️ Hetzner Cloud Deployment

For Kubernetes clusters hosted on Hetzner Cloud using Floating IPs.

**[→ Go to Hetzner Setup](hetzner/README.md)**

**Use case:**
- Hetzner Cloud infrastructure
- Using Hetzner Floating IPs
- Cloud-native deployments

## 📋 Prerequisites

### Common Requirements

- ☸️ Kubernetes cluster (v1.19+)
- 🛠️ `kubectl` CLI configured
- 🔧 Helm 3.x (recommended)

### Bare-Metal Specific

- 🌐 Available IP address range from your network
- 📡 Layer 2 network connectivity between nodes

### Hetzner Cloud Specific

- 🌐 Hetzner Cloud account
- 💰 Available Floating IP
- 🔑 Access to Hetzner Cloud Console

## 🧪 Testing

After deployment, test with the example Nginx application:

```bash
kubectl apply -f examples/nginx-demo.yml
```

Verify the LoadBalancer service:

```bash
kubectl get svc nginx-demo
```

Test the endpoint:

```bash
curl http://<EXTERNAL-IP>
```

## 📚 Documentation

### Setup Guides

- **[Bare-Metal Setup](bare-metal/README.md)** - Complete guide for bare-metal deployments
- **[Hetzner Setup](hetzner/README.md)** - Complete guide for Hetzner Cloud deployments

### Additional Resources

- **[Hetzner Floating IP Setup](hetzner/docs/floating-ip-setup.md)** - Detailed Floating IP configuration
- **[Troubleshooting Guide](hetzner/docs/troubleshooting.md)** - Common issues and solutions

### External Links

- [MetalLB Official Documentation](https://metallb.universe.tf/)
- [MetalLB Configuration Reference](https://metallb.universe.tf/configuration/)
- [Layer 2 Mode Concepts](https://metallb.universe.tf/concepts/layer2/)
- [Kubernetes Services](https://kubernetes.io/docs/concepts/services-networking/service/)

## 🔧 Configuration Overview

### IP Address Pool

Defines which IP addresses MetalLB can assign:

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: metallb-pool
  namespace: metallb
spec:
  addresses:
  - 192.168.1.100-192.168.1.150  # Bare-metal
  # OR
  - 5.5.5.55/32               # Hetzner Floating IP
```

### L2 Advertisement

Configures how MetalLB announces IPs:

```yaml
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: default-l2
  namespace: metallb
spec:
  ipAddressPools:
  - metallb-pool
```

## 🆚 Deployment Comparison

| Feature | Bare-Metal | Hetzner Cloud |
|---------|-----------|---------------|
| **IP Source** | Network IP range | Floating IP |
| **Setup Complexity** | Medium | Low |
| **Cost** | Included | Additional cost |
| **Flexibility** | High | Medium |
| **Failover** | Manual | Automatic |
| **Best For** | On-premise | Cloud deployments |

## 🔍 Common Issues

### Service Stuck in Pending

```bash
# Check MetalLB pods
kubectl get pods -n metallb

# Check controller logs
kubectl logs -n metallb deployment/metallb-controller
```

### IP Not Assigned

```bash
# Verify IP pool
kubectl get ipaddresspools.metallb.io -n metallb

# Check configuration
kubectl describe ipaddresspool <pool-name> -n metallb
```

### Service Not Accessible

```bash
# Check service status
kubectl get svc

# Verify endpoints
kubectl get endpoints <service-name>

# Test from within cluster
kubectl run -it --rm debug --image=busybox -- wget -O- http://<cluster-ip>
```

For detailed troubleshooting, see:
- [Hetzner Troubleshooting Guide](hetzner/docs/troubleshooting.md)
- [MetalLB Troubleshooting](https://metallb.universe.tf/troubleshooting/)

## 🧹 Cleanup

Remove example application:

```bash
kubectl delete -f examples/nginx-demo.yml
```

Remove MetalLB:

```bash
# Delete configurations
kubectl delete -f <your-setup>/l2-advertisement.yml
kubectl delete -f <your-setup>/address-pool.yml

# Uninstall MetalLB (if using Helm)
helm uninstall metallb -n metallb
kubectl delete namespace metallb

# OR (if using manifest)
kubectl delete -f https://raw.githubusercontent.com/metallb/metallb/v0.14.9/config/manifests/metallb-native.yaml
```

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📄 License

MIT License

##  Acknowledgments

- [MetalLB Project](https://metallb.universe.tf/)
- [Kubernetes Community](https://kubernetes.io/)

---

<div align="center">

  <p>
    <a href="https://metallb.universe.tf/">MetalLB</a> •
    <a href="https://kubernetes.io/">Kubernetes</a> •
  </p>
</div>
