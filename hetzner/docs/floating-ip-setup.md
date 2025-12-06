# Hetzner Floating IP Setup Guide

This guide explains how to create and configure a Floating IP in Hetzner Cloud for use with MetalLB.

## What is a Floating IP?

A Floating IP is a static IP address that can be instantly remapped from one server to another. In the context of MetalLB, it serves as the external IP address for your LoadBalancer services.

## Creating a Floating IP

### Step 1: Access Hetzner Cloud Console

1. Log in to your [Hetzner Cloud Console](https://console.hetzner.cloud/)
2. Select your project

### Step 2: Navigate to Floating IPs

1. In the left sidebar, click on **"Floating IPs"**
2. Click the **"Add Floating IP"** button

### Step 3: Configure Floating IP

1. **Type**: Select **IPv4**
2. **Location**: Choose the same location as your Kubernetes cluster
3. **Name**: Give it a descriptive name (e.g., `k8s-metallb-ip`)
4. **Description** (optional): Add a description for reference

### Step 4: Create and Assign

1. Click **"Create & Buy now"**
2. After creation, you'll see your new Floating IP in the list
3. Note down the IP address (e.g., `5.5.5.55`)

### Step 5: Assign to Server (Optional)

You can optionally assign the Floating IP to one of your servers:

1. Click on the Floating IP
2. Click **"Actions"** → **"Assign to server"**
3. Select your master or worker node
4. Click **"Assign Floating IP"**

## Configuring MetalLB with Floating IP

Once you have your Floating IP, update the MetalLB configuration:

1. Edit `address-pool.yml`
2. Replace `YOUR_FLOATING_IP` with your actual Floating IP
3. Apply the configuration:

```bash
kubectl apply -f address-pool.yml
```

## Important Notes

### Network Configuration

- Floating IPs work at the network level and don't require specific server configuration
- MetalLB will handle the ARP/NDP announcements automatically
- All nodes in your cluster must be able to reach the Floating IP

### Firewall Rules

Ensure your Hetzner firewall allows traffic to the Floating IP:

1. Go to **"Firewalls"** in Hetzner Cloud Console
2. Create or edit firewall rules to allow:
   - Inbound traffic on required ports (e.g., 80, 443)
   - From any source (`0.0.0.0/0` for public access)

### DNS Configuration

To use a domain name:

1. Point your domain's A record to the Floating IP
2. Example DNS record:
   ```
   Type: A
   Name: example.com
   Value: 5.5.5.55
   TTL: 3600
   ```

## Pricing

- Floating IPs are billed monthly (check current Hetzner pricing)
- The IP remains yours even if not assigned to a server
- You can reassign it between servers instantly at no additional cost

## Troubleshooting

### Floating IP Not Responding

1. Verify the IP is assigned in Hetzner Console
2. Check firewall rules allow traffic
3. Verify MetalLB pods are running:
   ```bash
   kubectl get pods -n metallb
   ```

### Service Not Getting IP

1. Check IP Address Pool configuration:
   ```bash
   kubectl get ipaddresspools.metallb.io -n metallb
   ```
2. Verify the Floating IP matches the pool configuration
3. Check MetalLB controller logs:
   ```bash
   kubectl logs -n metallb deployment/metallb-controller
   ```

## Best Practices

1. **Naming Convention**: Use descriptive names for Floating IPs (e.g., `prod-k8s-ingress`)
2. **Documentation**: Keep a record of which Floating IP is used for which service
3. **Backup**: Note down Floating IPs before making cluster changes
4. **Monitoring**: Set up monitoring for services using Floating IPs

## Additional Resources

- [Hetzner Floating IP Documentation](https://docs.hetzner.com/cloud/floating-ips/overview/)
- [Hetzner Cloud Pricing](https://www.hetzner.com/cloud#pricing)
- [MetalLB Layer 2 Configuration](https://metallb.universe.tf/configuration/_advanced_l2_configuration/)

---

**Next Steps**: After setting up your Floating IP, return to the [main README](../README.md) to continue with MetalLB configuration.
