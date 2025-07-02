
# Setting Up a VPN on Windows Server and Windows 10 Client

This report walks through every step—on the **server** and on the **client**—to build a working VPN.
---

## Part 1: Server-Side Setup (Windows Server 2022)

### 1. Install Required Roles

1. Open **Server Manager** and click **Add roles and features**.

   * ![add roles2](https://github.com/user-attachments/assets/3e768b66-10ba-4e1f-9c91-6bc763357633)


2. Choose **Role-based or feature-based installation**, then pick your server (DC01.adatum.ca).

   * ![add roles3](https://github.com/user-attachments/assets/cb7725d1-242f-4703-ab0a-6bc1df3f1672)

3. On **Server Roles**, check:

   * **DHCP Server**
   * **Network Policy and Access Services** → **DirectAccess and VPN (RAS)** and **Routing**
   * ![add roles](https://github.com/user-attachments/assets/127a1409-0956-404d-abab-f6cd8cb9b23a)
   * ![add roles2](https://github.com/user-attachments/assets/163b2089-f484-4f7b-82f7-12b768a37482)

4. Finish the wizard. This installs:

   * **DHCP** (hands out IPs)
   * **RRAS** (provides VPN & NAT)
   * **NPS** (controls who can connect)

---

### 2. Configure DHCP for VPN Clients

1. In **Server Manager → Tools → DHCP**, expand your server and right-click **IPv4 → New Scope**.

   * *(screenshot: `dhcp6.jpg` and `dhcp8.jpg`)*
2. **Scope Wizard**:

   * **Name**: e.g. “Building1 Scope”
   * **IP Range**: `192.168.1.1` to `192.168.1.254`, mask `255.255.255.0`

     * *(screenshot: `dhcp9.jpg`)*
   * **Exclusions**: reserve `192.168.1.1 – 192.168.1.10` (for routers, servers)

     * *(screenshot: `dhcp10.jpg`)*
   * **Lease Duration**: 1 day

     * *(screenshot: `dhcp11.jpg`)*
   * **Configure Options** now: set gateway (router IP), DNS server, domain name.
3. **Result**: VPN clients will receive an IP address from this range when they connect.

---

### 3. Enable and Configure RRAS (VPN + NAT)

1. In **Server Manager → Tools → Routing and Remote Access**, right-click your server → **Configure and Enable Routing and Remote Access**.

   * *(screenshot: `route.jpg`)*
2. In the wizard select **“Virtual private network (VPN) access and NAT”**.

   * *(screenshot: `route2.jpg`)*
3. After installation, under **IPv4 → NAT**, right-click and choose **New Interface…**.

   * *(screenshot: `route6.jpg` and `route4.jpg`)*
4. Mark your **public** adapter and check **Enable NAT on this interface**.
5. **Result**: Remote clients will build a VPN tunnel and can share the server’s public IP to reach the Internet (if desired).

---

### 4. Set Up Network Policy Server (NPS)

1. In **Server Manager → Tools → Network Policy Server**.

   * *(screenshot: `route15.jpg`)*
2. Under **Policies**, select **Network Policies** and open the policy **“Connections to Microsoft Routing and Remote Access server”**.

   * *(screenshot: `route5.jpg`)*
3. **Overview** tab:

   * Access Permission → **Grant Access**
   * Type of network access server → **Remote Access Server (VPN-Dial up)**
4. **Conditions** tab: add **MS-RAS Vendor ID = “311\$”** so only this RRAS is allowed.

   * *(screenshot: `route6.jpg`)*
5. **Constraints** tab: set

   * **Authentication Methods** (e.g. MS-CHAP v2 or EAP)
   * **Idle Timeout** (disconnect after 1 minute of no traffic)

     * *(screenshot: `route8.jpg`)*
   * **Session Timeout** (max time per session)

     * *(screenshot: `route14.jpg`)*
   * **Day/time restrictions** (here, allowed 24×7)

     * *(screenshot: `route7.jpg`)*
6. **Result**: Only authorized users can connect over VPN, under the conditions you choose.

---

### 5. (Optional) Enable Accounting & Reporting

1. In **Remote Access Management Console → Reporting → Configure Accounting**.

   * *(screenshot: `route12.jpg` and `route13.jpg`)*
2. Enable **Use RADIUS accounting** and/or **Use inbox accounting**.
3. **Result**: The server logs who connected, when, and how much data they used.

---

## Part 2: Client-Side Setup (Windows 10)

### 1. Add a New VPN Profile

1. Open **Settings → Network & Internet → VPN** and click **Add a VPN connection**.

   * *(screenshot: `win-vpn2.jpg` and `win-vpn3.jpg`)*
2. Fill in:

   * **VPN provider**: Windows (built-in)
   * **Connection name**: e.g. `adatum.ca`
   * **Server name or address**: the VPN server’s public name or IP (e.g. `dc01.adatum.ca` or `192.168.1.200`)
   * **VPN type**: **Automatic** (or explicitly choose **SSTP**/**L2TP/IPsec** if you know the server’s protocol)
   * **Sign-in info**: **User name and password**, then type your username (e.g. `Jess`) and password.
   * *(screenshot: `win-vpn4.jpg` and `win-vpn6.jpg`)*
3. Click **Save**. The profile appears in your VPN list.

   * *(screenshot: `win-vpn5.jpg`)*

---

### 2. Edit or Test the Connection

1. Select your new VPN entry and click **Advanced options → Edit** to tweak any field.

   * *(screenshot: `win-vpn7.jpg`)*
2. Finally, hit **Connect**.

   * *(screenshot: `win-vpn8.jpg` and `win-vpn1.jpg`)*
3. If it fails: go back to **Edit** and choose the exact **VPN type** that matches your server (PPTP, L2TP, SSTP, or IKEv2).

---

## What Each Step Achieves

| Step                         | Goal                                                       |
| ---------------------------- | ---------------------------------------------------------- |
| Install Roles & Features     | Give the server DHCP, VPN, routing, and RADIUS services    |
| Configure DHCP Scope         | Provide IP addresses to VPN and LAN clients                |
| Enable RRAS (VPN + NAT)      | Create the VPN endpoint and allow traffic forwarding       |
| Set Up NPS Policies          | Control who and how clients authenticate                   |
| Enable Accounting (Optional) | Log and report connection activity                         |
| Add VPN Profile on Client    | Tell Windows 10 how to reach and authenticate with the VPN |
| Test & Edit Client Profile   | Ensure protocol and credentials match the server settings  |

---

**Summary:**

* The **server** side is all about installing and configuring the services (DHCP, RRAS, NPS) that make VPN possible and secure.
* The **client** side is simply adding a VPN profile in Windows 10, pointing it at your server, and testing the connection.
