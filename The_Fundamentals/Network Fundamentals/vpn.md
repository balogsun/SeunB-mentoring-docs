
# Setting Up a VPN on Windows Server and Windows 10 Client

---

## Part 1: Server-Side Setup (Windows Server 2022)

### 1. Install Required Roles

1. Open **Server Manager** and click **Add roles and features**.

   * ![add roles](https://github.com/user-attachments/assets/c6d24ca1-b87e-4c2d-8656-67ec3ca0d6c8)

2. Choose **Role-based or feature-based installation**, then pick your server (DC01.adatum.ca).
   
   *  ![dhcp1](https://github.com/user-attachments/assets/84fb92f2-722e-47cb-ab03-b009546f6fe8)
   *  ![dhcp2](https://github.com/user-attachments/assets/21ea50a0-0f52-4d72-9502-ffc39a0b337f)

4. On **Server Roles**, check:

   * **DHCP Server**
   * **Network Policy and Access Services** → **DirectAccess and VPN (RAS)** and **Routing**
   *  ![add roles2](https://github.com/user-attachments/assets/a7c5692c-9867-41a4-b7e3-bba5fabcacf3)
   *  ![dhcp3](https://github.com/user-attachments/assets/965db896-c1f6-4dfa-85bf-fb12a2597065)
   *  ![add roles3](https://github.com/user-attachments/assets/cb7725d1-242f-4703-ab0a-6bc1df3f1672)

5. Finish the wizard. This installs:

   * **DHCP** (hands out IPs)
   * **RRAS** (provides VPN & NAT)
   * **NPS** (controls who can connect)
   * ![dhcp4](https://github.com/user-attachments/assets/0071b73d-79a0-4a54-8e2d-732c44e71fb6)

---

### 2. Configure DHCP for VPN Clients

1. In **Server Manager → Tools → DHCP**, expand your server and right-click **IPv4 → New Scope**.

   * ![dhcp5](https://github.com/user-attachments/assets/1f1d7a94-cf34-42a0-a365-90dec50b236a)

   * ![dhcp6](https://github.com/user-attachments/assets/c0cefba9-9df6-4ee3-82ef-7a5fc791e665)


2. **Scope Wizard**:

   * **Name**: e.g. “Building1 Scope”
   * ![dhcp7](https://github.com/user-attachments/assets/4bee857c-f467-4ffe-b942-9c7e4c66d451)

   * **IP Range**: `192.168.1.1` to `192.168.1.254`, mask `255.255.255.0`

     * !![dhcp8](https://github.com/user-attachments/assets/31c02819-fe19-4997-9c8a-3bfcf94dce0a)

   * **Exclusions**: reserve `192.168.1.1 – 192.168.1.10` (for routers, servers)

     * ![dhcp9](https://github.com/user-attachments/assets/24caaae8-71e9-4134-b546-d15071b0fbc3)

   * **Lease Duration**: 1 day

     * ![dhcp10](https://github.com/user-attachments/assets/f9e93ab1-e458-4f34-a811-53ef4a2f7c1b)

   * **Configure Options** now: set gateway (router IP), DNS server, domain name.
   * ![dhcp11](https://github.com/user-attachments/assets/08806103-0c52-408f-892a-d4b051658dc9)

3. **Result**: VPN clients will receive an IP address from this range when they connect.

---

### 3. Enable and Configure RRAS (VPN + NAT)

1. In **Server Manager → Tools → Routing and Remote Access**, right-click your server → **Configure and Enable Routing and Remote Access**.

2. In the wizard select **“Virtual private network (VPN) access and NAT”**.

   * ![route1](https://github.com/user-attachments/assets/92348b56-9ee1-4e3e-a354-58a627655199)

3. After installation, under **IPv4 → NAT**, right-click and choose **New Interface…**.

   * ![route](https://github.com/user-attachments/assets/ceae8517-8667-415b-80ca-194cf50a0dad)

4. Mark your **public** adapter and check **Enable NAT on this interface**.
   * ![route2](https://github.com/user-attachments/assets/3cee780d-eaa5-46c4-8970-825cce4e36d6)

6. **Result**: Remote clients will build a VPN tunnel and can share the server’s public IP to reach the Internet (if desired).

---

### 4. Set Up Network Policy Server (NPS)

1. In **Server Manager → Tools → Network Policy Server**.

   * ![route4](https://github.com/user-attachments/assets/41a520ea-9616-4745-bc98-5c0cbc0e65b4)

2. Under **Policies**, select **Network Policies** and open the policy **“Connections to Microsoft Routing and Remote Access server”**.

3. **Overview** tab:

   * Access Permission → **Grant Access**
   * Type of network access server → **Remote Access Server (VPN-Dial up)**
   *  ![route9](https://github.com/user-attachments/assets/85335c7f-2ce7-4785-8165-e5bb4ae790a0)

   *  ![route5](https://github.com/user-attachments/assets/ba019ef3-cf47-4a48-8ad2-7acfb4468a4c)
     
4. **Conditions** tab: add **MS-RAS Vendor ID = “311\$”** so only this RRAS is allowed.

   * ![route6](https://github.com/user-attachments/assets/e4a2a956-65a1-4741-b927-2644c4fc5c4e)
  
   * ![route7](https://github.com/user-attachments/assets/6c9b80e2-9167-4616-9cab-6f2f285b7249)

5. **Constraints** tab: set

   * **Authentication Methods** (e.g. MS-CHAP v2 or EAP)
   * **Idle Timeout** (disconnect after 1 minute of no traffic)

     * ![route8](https://github.com/user-attachments/assets/c34a8f39-3981-4da9-91af-573687fcf82d)

   * **Session Timeout** (max time per session)
   * ![route15](https://github.com/user-attachments/assets/a6625bd4-738f-428b-97e1-c6485c502a44)

   * **Day/time restrictions** (here, allowed 24×7)

6. **Result**: Only authorized users can connect over VPN, under the conditions you choose.

---

### 5. (Optional) Enable Accounting & Reporting

1. In **Remote Access Management Console → Reporting → Configure Accounting**.

   * ![route10](https://github.com/user-attachments/assets/479ae4d6-fd5c-4e4a-879f-fec8558dbdb3)

   * ![route11](https://github.com/user-attachments/assets/aab8f03e-492b-4b3c-8a18-cc3709ea8f35)
   * ![route12](https://github.com/user-attachments/assets/82db0ecf-9ba2-4e44-859b-c7884a7ba954)

2. Enable **Use RADIUS accounting** and/or **Use inbox accounting**.
   * ![route13](https://github.com/user-attachments/assets/a3ac254d-3567-496f-8f6c-db3b928eaeb6)
     
4. **Result**: The server logs who connected, when, and how much data they used.

---

## Part 2: Client-Side Setup (Windows 10)

### 1. Add a New VPN Profile

1. Open **Settings → Network & Internet → VPN** and click **Add a VPN connection**.

   * ![win-vpn1](https://github.com/user-attachments/assets/2cfab204-730e-4390-90e5-6cea3b0d51a8)
   * ![win-vpn2](https://github.com/user-attachments/assets/76c502ca-4bb2-4453-9914-61ad14907e76)
   

2. Fill in:

   * **VPN provider**: Windows (built-in)
   * **Connection name**: e.g. `adatum.ca`
   * **Server name or address**: the VPN server’s public name or IP (e.g. `dc01.adatum.ca` or `192.168.1.200`)
   * **VPN type**: **Automatic** (or explicitly choose **SSTP**/**L2TP/IPsec** if you know the server’s protocol)
   * **Sign-in info**: **User name and password**, then type your username (e.g. `Jess`) and password.
   * ![win-vpn3](https://github.com/user-attachments/assets/43a48c1e-f394-4cbd-b6ec-ff8375b8eaba)
   * ![win-vpn5](https://github.com/user-attachments/assets/2f0c5de4-d344-4fff-9fdb-50664a23c32a)
   
3. Click **Save**. The profile appears in your VPN list.

   * ![win-vpn4](https://github.com/user-attachments/assets/3e714381-4877-47bf-a520-11ce213fbe39)
  
---

### 2. Edit or Test the Connection

1. Select your new VPN entry and click **Advanced options → Edit** to tweak any field.

   * ![win-vpn6](https://github.com/user-attachments/assets/635c42a1-c282-49cb-aadd-0340002338fd)
   * ![win-vpn7](https://github.com/user-attachments/assets/a65676b0-d75b-4996-ba12-4665a39ad9ce)
   * ![win-vpn8](https://github.com/user-attachments/assets/15b531af-7ad4-430e-abac-726e912e2e8c)


2. Finally, hit **Connect**.

  * ![win-vpn4](https://github.com/user-attachments/assets/3e714381-4877-47bf-a520-11ce213fbe39)

3. If it fails: go back to **Edit** and choose the exact **VPN type** that matches your server (PPTP, L2TP, SSTP, or IKEv2).

---

## What Each Step Achieves

| Step                         | Goal                                                                                                                                                                                                            |
| ---------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Install Roles & Features     | add roles to server such as **DHCP Server** **Network Policy and Access Services** role, **Remote Access** roles, along side roles services such as **DirectAccess and VPN (RAS)** and **Routing**). |
| Configure DHCP Scope         | Provide IP addresses to both VPN and LAN clients by configuring IP ranges, reserving specific addresses, and setting an appropriate lease duration.                                                             |
| Enable RRAS (VPN + NAT)      | Create the VPN endpoint and allow traffic forwarding by enabling Routing and Remote Access and activating NAT on the server’s public network interface.                                                         |
| Set Up NPS Policies          | Control who and how clients authenticate by selecting the RRAS service in Network Policy Server, then applying authentication methods plus connection day schedules and idle/session timeout settings.          |
| Enable Accounting (Optional) | Log and report connection activity by using the Remote Access Management tool to configure RADIUS or inbox accounting for auditing and usage tracking.                                                          |
| Add VPN Profile on Client    | Tell Windows 10 how to reach and authenticate with the VPN by entering the server’s IP or domain, choosing the correct protocol, and supplying user credentials.                                                |
| Test & Edit Client Profile   | Ensure the VPN type, server address, and credentials match the server settings; edit the profile and retry until the client successfully connects.  
---

**Summary:**

* The **server** side is all about installing and configuring the services (DHCP, RRAS, NPS) that make VPN possible and secure.
* The **client** side is simply adding a VPN profile in Windows 10, pointing it at your server, and testing the connection.
