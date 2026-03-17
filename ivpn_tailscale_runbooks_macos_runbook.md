# IVPN & Tailscale Compatibility Runbook: macOS

## Overview
This runbook provides step-by-step instructions for installing and configuring IVPN and Tailscale on macOS to ensure they work together seamlessly. By default, IVPN's aggressive firewall (kill switch) blocks Tailscale's virtual network adapter. This guide resolves that conflict using split tunneling.

## Prerequisites
- macOS system (Intel or Apple Silicon)
- Administrator privileges
- Active IVPN account
- Active Tailscale account

## Step 1: Install IVPN
1. Download the macOS installer from the [IVPN website](https://www.ivpn.net/apps-macos).
2. Open the downloaded `.dmg` file and drag the IVPN app into your **Applications** folder.
3. Launch IVPN from your Applications folder.
4. You may be prompted to install a helper tool or allow system extensions. Follow the on-screen prompts and authenticate with your Mac password or Touch ID.
5. Log in to your account and connect to a server.

## Step 2: Install Tailscale
*Note: There are two versions of Tailscale for macOS (App Store and Standalone). The Standalone version is generally preferred for advanced networking setups, but both work.*

1. Download the Standalone macOS installer from the [Tailscale website](https://tailscale.com/download/mac).
2. Open the downloaded `.pkg` file and follow the installation wizard.
3. Launch Tailscale from your Applications folder. It will appear in your menu bar at the top of the screen.
4. Click the Tailscale menu bar icon and select **Log in** to authenticate and connect to your Tailnet.
5. You will be prompted to allow Tailscale to add VPN configurations. Click **Allow**.
   *(Note: At this point, your internet connection may drop or Tailscale may fail to route traffic due to IVPN's firewall.)*

## Step 3: Configure IVPN Split Tunneling
To allow Tailscale traffic to bypass the IVPN tunnel, you must add the Tailscale application to IVPN's split tunnel list.

1. Open the IVPN application.
2. Navigate to **Settings** (gear icon) -> **Split Tunnel**.
3. Check the box for **Split Tunnel**.
4. Ensure **Inverse mode (BETA)** is *unchecked*.
5. Uncheck **Block DNS servers not specified by the IVPN application**. *(Crucial for Tailscale MagicDNS)*
6. Click **Add application...**
7. Navigate to your **Applications** folder and select **Tailscale.app**.

## Step 4: Configure Tailscale DNS (Optional but Recommended)
If you experience DNS resolution issues, prevent Tailscale from hijacking the system DNS, allowing IVPN to handle public DNS queries while Tailscale handles internal MagicDNS.

1. Open the Terminal app.
2. Bring Tailscale down:
   ```bash
   sudo tailscale down
   ```
3. Bring Tailscale back up without accepting DNS:
   ```bash
   sudo tailscale up --accept-dns=false
   ```

## Step 5: Verification
1. Open the Terminal app.
2. Verify Tailscale is running and connected:
   ```bash
   tailscale status
   ```
3. Test connectivity to another node on your Tailnet:
   ```bash
   tailscale ping <TAILSCALE_IP_OF_OTHER_NODE>
   ```
4. Test general internet connectivity:
   ```bash
   ping -c 4 8.8.8.8
   ```

## Troubleshooting
- **No Internet:** Ensure IVPN's "Block DNS servers not specified by the IVPN application" is unchecked.
- **Cannot reach Tailnet nodes:** Ensure you selected the correct `Tailscale.app` in the IVPN split tunnel settings. If you installed the App Store version, the path might be slightly different, but it should still be in your Applications folder.
- **System Extension Blocked:** If macOS blocks the IVPN or Tailscale system extensions, go to **System Settings > Privacy & Security** and look for a prompt to "Allow" the software from the respective developer.
