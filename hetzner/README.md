# MetalLB on Hetzner Cloud

Complete guide for deploying MetalLB on Kubernetes clusters hosted on Hetzner Cloud using Floating IPs.

## 📖 Overview

This setup uses MetalLB with Hetzner Cloud Floating IPs to provide LoadBalancer services for your Kubernetes cluster. The configuration uses Layer 2 mode for simple ARP-based load balancing.

## 📋 Prerequisites

- Kubernetes cluster running on Hetzner Cloud
- `kubectl` configured and connected to your cluster
- Helm 3.x installed
- Hetzner Floating IP (see setup guide below)
- Root or sudo access to the cluster

## 🚀 Installation

### 1. Install Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### 2. Add MetalLB Helm Repository

```bash
helm repo add metallb https://metallb.github.io/metallb
```

### 3. Create MetalLB Namespace

```bash
kubectl create ns metallb
```

### 4. Install MetalLB

```bash
helm install -n metallb metallb metallb/metallb
```

### 5. Verify Installation

```bash
kubectl get pods -n metallb
```

Wait until all pods (controller and speakers) are in `Running` status.

## ⚙️ Configuration

### 6. Setup Hetzner Floating IP

Before configuring MetalLB, you need a Floating IP from Hetzner Cloud:

1. Go to [Hetzner Cloud Console](https://console.hetzner.cloud/)
2. Navigate to your project → **Floating IPs**
3. Create a new Floating IP (IPv4)
4. Note the IP address

For detailed instructions, see the [Floating IP Setup Guide](docs/floating-ip-setup.md).

### 7. Configure IP Address Pool

Update `address-pool.yml` with your Floating IP:

```yaml
---
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: metallb-pool
  namespace: metallb
spec:
  addresses:
  - YOUR_FLOATING_IP/32  # e.g., 5.5.5.55/32
```

Apply the configuration:

```bash
kubectl apply -f address-pool.yml
```

### 8. Configure L2 Advertisement

Apply the L2 advertisement configuration:

```bash
kubectl apply -f l2-advertisement.yml
```

### 9. Verify Configuration

```bash
kubectl get ipaddresspools.metallb.io -n metallb
```

You should see your IP pool with the configured Floating IP address.

## 🧪 Testing

Deploy the example Nginx application:

```bash
kubectl apply -f ../examples/nginx-demo.yml
```

Check if the service received the Floating IP:

```bash
kubectl get svc nginx-demo
```

Test the endpoint:

```bash
curl http://YOUR_FLOATING_IP
```

You should see the Nginx welcome page.

## 📁 Configuration Files

### address-pool.yml

Defines the IP address pool that MetalLB will use:

```yaml
---
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: metallb-pool
  namespace: metallb
spec:
  addresses:
  - YOUR_FLOATING_IP/32
```

### l2-advertisement.yml

Configures Layer 2 advertisement for the IP pool:

```yaml
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: default-l2
  namespace: metallb
spec:
  ipAddressPools:
  - metallb-pool
```

## 🔧 How It Works

1. **MetalLB Controller**: Watches for LoadBalancer services and assigns IPs from the configured pool
2. **MetalLB Speakers**: Run as DaemonSet on all nodes, handle ARP/NDP announcements
3. **Floating IP**: Acts as the external IP address for your services
4. **Layer 2 Mode**: Uses ARP to announce the IP on the local network

## 📝 Important Notes

- **Floating IP Location**: Must be in the same Hetzner location as your cluster
- **Firewall Rules**: Ensure Hetzner firewall allows traffic to the Floating IP
- **Network Configuration**: All cluster nodes must be able to reach the Floating IP
- **IP Availability**: One Floating IP can be used for multiple services (shared IP)

## 🔍 Troubleshooting

### Service Stuck in Pending

Check MetalLB controller logs:
```bash
kubectl logs -n metallb deployment/metallb-controller
```

### IP Not Assigned

Verify IP pool configuration:
```bash
kubectl describe ipaddresspool metallb-pool -n metallb
```

### Service Not Accessible

1. Check Hetzner firewall rules
2. Verify Floating IP is assigned in Hetzner Console
3. Check speaker pods are running:
   ```bash
   kubectl get pods -n metallb -l component=speaker
   ```

For more troubleshooting, see the [Troubleshooting Guide](docs/troubleshooting.md).

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
helm uninstall metallb -n metallb
kubectl delete namespace metallb
```

## 📚 Additional Resources

- [Floating IP Setup Guide](docs/floating-ip-setup.md) - Detailed Hetzner Floating IP configuration
- [Troubleshooting Guide](docs/troubleshooting.md) - Common issues and solutions
- [MetalLB Documentation](https://metallb.universe.tf/)
- [Hetzner Cloud Docs](https://docs.hetzner.com/cloud/)

## 🆚 Differences from Bare-Metal Setup

- Uses Hetzner Floating IPs instead of physical network IPs
- Requires Hetzner Cloud Console configuration
- Floating IPs can be reassigned between servers instantly
- Includes Hetzner-specific firewall considerations

---

**Next Steps**: After setup, consider configuring Ingress controllers (nginx-ingress, Traefik) to route traffic from the LoadBalancer to your applications.
