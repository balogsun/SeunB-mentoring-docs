
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

   * ![dhcp6](https://github.com/user-attachments/assets/21fc0d25-2aff-41f0-9b97-adad6a0c69d3)
   * ![dhcp8](https://github.com/user-attachments/assets/793e27a0-7ee8-447c-9755-7fd8ba631d2f)


2. **Scope Wizard**:

   * **Name**: e.g. “Building1 Scope”
   * **IP Range**: `192.168.1.1` to `192.168.1.254`, mask `255.255.255.0`

     * ![dhcp9](https://github.com/user-attachments/assets/052e2938-3293-4599-8862-4bd09aeb5c4f)

   * **Exclusions**: reserve `192.168.1.1 – 192.168.1.10` (for routers, servers)

     * ![dhcp10](https://github.com/user-attachments/assets/2ad798f9-7e73-4d49-83ab-233cced62ef4)

   * **Lease Duration**: 1 day

     * ![dhcp11](https://github.com/user-attachments/assets/40675141-ff60-4192-be26-b9796e07f4d2)

   * **Configure Options** now: set gateway (router IP), DNS server, domain name.
3. **Result**: VPN clients will receive an IP address from this range when they connect.

---

### 3. Enable and Configure RRAS (VPN + NAT)

1. In **Server Manager → Tools → Routing and Remote Access**, right-click your server → **Configure and Enable Routing and Remote Access**.

   * ![route](https://github.com/user-attachments/assets/202d4e24-e38f-4468-a5d7-412cdbedcbb3)

2. In the wizard select **“Virtual private network (VPN) access and NAT”**.

   * ![route2](https://github.com/user-attachments/assets/f31e9034-5758-4044-9fe5-355f210eaf7d)

3. After installation, under **IPv4 → NAT**, right-click and choose **New Interface…**.

   * ![route6](https://github.com/user-attachments/assets/2c93c854-bb14-4bb1-a911-8bb40be2904b)
   * ![route4](https://github.com/user-attachments/assets/97e17fdb-d94c-466e-b74b-b3f2f0449da9)


4. Mark your **public** adapter and check **Enable NAT on this interface**.
5. **Result**: Remote clients will build a VPN tunnel and can share the server’s public IP to reach the Internet (if desired).

---

### 4. Set Up Network Policy Server (NPS)

1. In **Server Manager → Tools → Network Policy Server**.

   * ![route15](https://github.com/user-attachments/assets/90cb6839-d896-407f-86ea-4482f2274159)

2. Under **Policies**, select **Network Policies** and open the policy **“Connections to Microsoft Routing and Remote Access server”**.

   * ![route5](https://github.com/user-attachments/assets/ba019ef3-cf47-4a48-8ad2-7acfb4468a4c)

3. **Overview** tab:

   * Access Permission → **Grant Access**
   * Type of network access server → **Remote Access Server (VPN-Dial up)**
4. **Conditions** tab: add **MS-RAS Vendor ID = “311\$”** so only this RRAS is allowed.

   * ![route6](https://github.com/user-attachments/assets/e4a2a956-65a1-4741-b927-2644c4fc5c4e)

5. **Constraints** tab: set

   * **Authentication Methods** (e.g. MS-CHAP v2 or EAP)
   * **Idle Timeout** (disconnect after 1 minute of no traffic)

     * ![route8](https://github.com/user-attachments/assets/73481f89-13bb-4620-bb58-c66cd108e237)

   * **Session Timeout** (max time per session)

     * ![route14](https://github.com/user-attachments/assets/914d52f8-24c6-42c5-8763-531057abc30f)

   * **Day/time restrictions** (here, allowed 24×7)

     * ![route7](https://github.com/user-attachments/assets/7c33faf6-1b03-4793-bade-de8e53cd7b2a)

6. **Result**: Only authorized users can connect over VPN, under the conditions you choose.

---

### 5. (Optional) Enable Accounting & Reporting

1. In **Remote Access Management Console → Reporting → Configure Accounting**.

   * ![route12](https://github.com/user-attachments/assets/b2bbc345-a8ba-4144-8901-93160f5b0990)
   * ![route13](https://github.com/user-attachments/assets/9a6bd684-6fcc-497a-b3de-67208c9de846)


2. Enable **Use RADIUS accounting** and/or **Use inbox accounting**.
3. **Result**: The server logs who connected, when, and how much data they used.

---

## Part 2: Client-Side Setup (Windows 10)

### 1. Add a New VPN Profile

1. Open **Settings → Network & Internet → VPN** and click **Add a VPN connection**.

   * ![win-vpn1](https://github.com/user-attachments/assets/2cfab204-730e-4390-90e5-6cea3b0d51a8)
   * ![win-vpn2](https://github.com/user-attachments/assets/76c502ca-4bb2-4453-9914-61ad14907e76)
   * ![win-vpn3](https://github.com/user-attachments/assets/43a48c1e-f394-4cbd-b6ec-ff8375b8eaba)

2. Fill in:

   * **VPN provider**: Windows (built-in)
   * **Connection name**: e.g. `adatum.ca`
   * **Server name or address**: the VPN server’s public name or IP (e.g. `dc01.adatum.ca` or `192.168.1.200`)
   * **VPN type**: **Automatic** (or explicitly choose **SSTP**/**L2TP/IPsec** if you know the server’s protocol)
   * **Sign-in info**: **User name and password**, then type your username (e.g. `Jess`) and password.
   * ![win-vpn5](https://github.com/user-attachments/assets/2f0c5de4-d344-4fff-9fdb-50664a23c32a)
   * ![win-vpn6](https://github.com/user-attachments/assets/635c42a1-c282-49cb-aadd-0340002338fd)

3. Click **Save**. The profile appears in your VPN list.

   * ![win-vpn4](https://github.com/user-attachments/assets/3e714381-4877-47bf-a520-11ce213fbe39)


---

### 2. Edit or Test the Connection

1. Select your new VPN entry and click **Advanced options → Edit** to tweak any field.

   * ![win-vpn7](https://github.com/user-attachments/assets/a65676b0-d75b-4996-ba12-4665a39ad9ce)

2. Finally, hit **Connect**.

   * ![win-vpn8](https://github.com/user-attachments/assets/40763f17-3677-4373-a7c6-560bc832fb54)

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
