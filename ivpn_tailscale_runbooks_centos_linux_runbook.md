# IVPN & Tailscale Compatibility Runbook: CentOS/RHEL Linux

## Overview
This runbook provides step-by-step instructions for installing and configuring IVPN and Tailscale on CentOS/RHEL-based systems to ensure they work together seamlessly. By default, IVPN's aggressive firewall (kill switch) blocks Tailscale's virtual network adapter. This guide resolves that conflict using split tunneling.

## Prerequisites
- CentOS/RHEL/Rocky/Alma Linux system (8 or 9)
- Sudo privileges
- Active IVPN account
- Active Tailscale account

## Step 1: Install IVPN
1. Add the IVPN repository:
   ```bash
   sudo curl -fsSL https://repo.ivpn.net/centos/ivpn.repo -o /etc/yum.repos.d/ivpn.repo
   ```
2. Install the IVPN client:
   ```bash
   sudo dnf install ivpn-ui
   ```
3. Enable and start the IVPN service:
   ```bash
   sudo systemctl enable --now ivpn-service
   ```
4. Open the IVPN GUI, log in to your account, and connect to a server.

## Step 2: Install Tailscale
1. Add the Tailscale repository:
   ```bash
   sudo dnf config-manager --add-repo https://pkgs.tailscale.com/stable/centos/9/tailscale.repo
   ```
   *(Note: Change `/9/` to `/8/` if using CentOS 8)*
2. Install Tailscale:
   ```bash
   sudo dnf install tailscale
   ```
3. Enable and start the Tailscale service:
   ```bash
   sudo systemctl enable --now tailscaled
   ```
4. Authenticate and connect Tailscale to your Tailnet:
   ```bash
   sudo tailscale up
   ```
   *(Note: At this point, your internet connection may drop or Tailscale may fail to route traffic due to IVPN's firewall.)*

## Step 3: Configure IVPN Split Tunneling
To allow Tailscale traffic to bypass the IVPN tunnel, you must add the Tailscale daemon to IVPN's split tunnel list.

1. Open the IVPN GUI.
2. Navigate to **Settings** (gear icon) -> **Split Tunnel**.
3. Check the box for **Split Tunnel**.
4. Ensure **Inverse mode (BETA)** is *unchecked*.
5. Uncheck **Block DNS servers not specified by the IVPN application**. *(Crucial for Tailscale MagicDNS)*
6. Click **Add application...**
7. Navigate to `/usr/sbin/` and select `tailscaled`.
   *(Alternatively, via CLI: `ivpn split-tunnel -add /usr/sbin/tailscaled`)*

## Step 4: Configure Tailscale DNS (Optional but Recommended)
If you experience DNS resolution issues, prevent Tailscale from hijacking the system DNS, allowing IVPN to handle public DNS queries while Tailscale handles internal MagicDNS.

1. Bring Tailscale down:
   ```bash
   sudo tailscale down
   ```
2. Bring Tailscale back up without accepting DNS:
   ```bash
   sudo tailscale up --accept-dns=false
   ```

## Step 5: Verification
1. Verify Tailscale is running and connected:
   ```bash
   tailscale status
   ```
2. Test connectivity to another node on your Tailnet:
   ```bash
   tailscale ping <TAILSCALE_IP_OF_OTHER_NODE>
   ```
3. Test general internet connectivity:
   ```bash
   ping 8.8.8.8
   ```

## Troubleshooting
- **No Internet:** Ensure IVPN's "Block DNS servers not specified by the IVPN application" is unchecked.
- **Cannot reach Tailnet nodes:** Verify `/usr/sbin/tailscaled` is exactly what was added to the IVPN split tunnel. Do not add the CLI tool `/usr/bin/tailscale`.
