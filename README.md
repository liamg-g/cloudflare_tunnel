# cloudflare_tunnel
LXC running Ubuntu to anchor a cloudflare tunnel for remote access to Proxmox server

# Cloudflare Tunnel Setup on Ubuntu LXC

A step-by-step guide to deploying a lightweight, secure Cloudflare Tunnel inside an Ubuntu LXC container (e.g., on Proxmox VE) to safely expose local services without open inbound ports.

---

## System Requirements & Resources

Cloudflare's `cloudflared` daemon is minimal and requires low compute resources.

| Resource | Minimum / Recommended | Notes |
| :--- | :--- | :--- |
| **Container Type** | Unprivileged LXC | No root privileges required |
| **OS Template** | Ubuntu 22.04 / 24.04 LTS | Standard minimal template |
| **CPU** | 1 vCPU | Typically idles under 1% usage |
| **RAM** | 256 MB – 512 MB | Uses under 50 MB active RAM |
| **Disk** | 4 GB | Sufficient for OS & packages |
| **Network** | DHCP or Static IP | Needs outbound internet connectivity |

---

## Step-by-Step Installation

### Step 1: Prepare the LXC Container
1. Download an **Ubuntu LTS** container template in your hypervisor (e.g., Proxmox VE).
2. Create an **Unprivileged LXC Container** with the resource specifications listed above.
3. Start the container and open the terminal console.

---

### Step 2: Retrieve your Cloudflare Tunnel Token
1. Log into your **Cloudflare Dashboard** and navigate to **Zero Trust**.
2. Go to **Networks** $\rightarrow$ **Tunnels** and click **Create a Tunnel**.
3. Choose **Cloudflare Tunnel (Connector)** and name your tunnel.
4. Copy the installation token provided in the dashboard setup screen.

---

### Step 3: Install `cloudflared` on Ubuntu

Run the following commands in your Ubuntu LXC terminal:

```bash
# 1. Update system packages and install prerequisites
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl lsb-release gnupg

# 2. Add Cloudflare's official GPG key and APT repository
sudo mkdir -p /usr/share/keyrings
curl -fsSL [https://pkg.cloudflare.com/cloudflare-main.gpg](https://pkg.cloudflare.com/cloudflare-main.gpg) | sudo tee /usr/share/keyrings/cloudflare-main.gpg >/dev/null
echo "deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] [https://pkg.cloudflare.com/cloudflared](https://pkg.cloudflare.com/cloudflared) $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/cloudflared.list

# 3. Install cloudflared
sudo apt update
sudo apt install -y cloudflared

# 4. Install the service using your secret token
sudo cloudflared service install <YOUR_SECRET_TOKEN>

# 5. Verify the service status
sudo systemctl status cloudflared
