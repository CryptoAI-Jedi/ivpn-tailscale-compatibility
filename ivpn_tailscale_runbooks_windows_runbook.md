# IVPN & Tailscale Compatibility Runbook: Windows 10/11

## Overview
This runbook provides step-by-step instructions for installing and configuring IVPN and Tailscale on Windows 10 and 11 to ensure they work together seamlessly. By default, IVPN's aggressive firewall (kill switch) blocks Tailscale's virtual network adapter. This guide resolves that conflict using split tunneling and proper network profile configuration.

## Prerequisites
- Windows 10 or 11 system
- Administrator privileges
- Active IVPN account
- Active Tailscale account

## Step 1: Install IVPN
1. Download the Windows installer from the [IVPN website](https://www.ivpn.net/apps-windows).
2. Run the installer and follow the on-screen prompts.
3. Open the IVPN application, log in to your account, and connect to a server.

## Step 2: Install Tailscale
1. Download the Windows installer from the [Tailscale website](https://tailscale.com/download/windows).
2. Run the installer and follow the on-screen prompts.
3. Launch Tailscale from the Start menu. It will appear in your system tray.
4. Right-click the Tailscale tray icon and select **Log in** to authenticate and connect to your Tailnet.
   *(Note: At this point, your internet connection may drop or Tailscale may fail to route traffic due to IVPN's firewall.)*

## Step 3: Configure IVPN Split Tunneling
To allow Tailscale traffic to bypass the IVPN tunnel, you must add the Tailscale background daemon and GUI application to IVPN's split tunnel list.

1. Open the IVPN application.
2. Navigate to **Settings** (gear icon) -> **Split Tunnel**.
3. Check the box for **Split Tunnel**.
4. Ensure **Inverse mode (BETA)** is *unchecked*.
5. Uncheck **Block DNS servers not specified by the IVPN application**. *(Crucial for Tailscale MagicDNS)*
6. Click **Add application...**
7. Navigate to `C:\Program Files\Tailscale\` and select **`tailscaled.exe`**. *(This is the background daemon)*
8. Click **Add application...** again.
9. Navigate to `C:\Program Files\Tailscale\` and select **`tailscale-ipn.exe`**. *(This is the GUI application)*
   *(CRITICAL: Do NOT select `tailscale.exe`, as that is only the CLI tool and will not fix the routing issue.)*

## Step 4: Configure Windows Network Profile (Crucial for Corporate/Public Networks)
Windows defaults new network connections to "Public", which applies strict firewall rules that block Tailscale traffic. You must set your active network connection to "Private".

1. Open Windows **Settings**.
2. Navigate to **Network & internet**.
3. Click on your active connection (e.g., **Wi-Fi** or **Ethernet**).
4. Under **Network profile type**, select **Private network**.
5. Disconnect and reconnect IVPN to apply the new firewall rules.

## Step 5: Verification
1. Open PowerShell or Command Prompt.
2. Verify Tailscale is running and connected:
   ```powershell
   tailscale status
   ```
3. Test connectivity to another node on your Tailnet:
   ```powershell
   tailscale ping <TAILSCALE_IP_OF_OTHER_NODE>
   ```
4. Test general internet connectivity:
   ```powershell
   ping 8.8.8.8
   ```

## Troubleshooting
- **No Internet:** Ensure IVPN's "Block DNS servers not specified by the IVPN application" is unchecked.
- **Cannot reach Tailnet nodes:** Verify you added `tailscaled.exe` and `tailscale-ipn.exe` to the IVPN split tunnel, NOT `tailscale.exe`.
- **Works at home but not at work:** Ensure your Windows Network Profile is set to "Private" on the work network (Step 4).
