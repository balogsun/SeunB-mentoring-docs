
## **Boosting System Resilience with Monitoring and Self-Healing: A Comprehensive Guide**

In today’s fast-paced digital world, maintaining system reliability and minimizing downtime are critical for business success. This comprehensive guide explores how to enhance system resilience through advanced monitoring and self-healing mechanisms. We will walk you through integrating Datadog for monitoring, setting up automated recovery scripts, and leveraging Node.js and webhooks to create a robust self-healing system. By the end of this guide, you'll have a fully automated setup that can proactively manage system issues, ensuring smooth and uninterrupted operations.


### **Prerequisites**

- **A Datadog Account**: Ensure you have an active Datadog account for monitoring and alerts.
- **A Linux Server**: This can be either on-premises or a cloud-based instance.
- **Internet Access**: Required for installing software, setting up integrations, and accessing Datadog.


### **1. Create a Datadog Account**
1. **Sign Up for Datadog**:
   - Visit [Datadog's sign-up page](https://www.datadoghq.com/) and create a `free trial` account by entering your email and setting a password.
   - Complete the registration process, create a name for your `organization`, and verify your email if required.
   - Log in to your Datadog dashboard.

### **2. Deploy Your First Datadog Agent**
Guides to install on your preferred operating system are listed in the `integration` --> `agents` section.

#### **For a Local Ubuntu Server:**
1. **Install the Datadog Agent**:
   - **Obtain Your Datadog API Key**:
     - Navigate to **Organization settings** > **API Keys** to find your API key.

   - **Install the Agent**:
     - Run the following command on your Ubuntu server to install the Datadog Agent:
       ```bash
       DD_API_KEY=8axxxxxxxxxxxxxxxxxxxxxxxxxxxf43 DD_SITE="datadoghq.com" bash -c "$(curl -L https://install.datadoghq.com/scripts/install_script_agent7.sh)"
       ```
     - This command sets up the agent and automatically configures it with your API key.

2. **Verify Agent Installation**:
   - Check the agent status to ensure it is running:
     ```bash
     sudo datadog-agent status
     ```
   - You should see output indicating that the agent is running and sending data.
   
   - To check logs, use:
     ```bash
     tail -f /var/log/datadog/agent.log
     ```

#### **For Ubuntu Server Managing a Kubernetes Cluster:**
1. **Create a Datadog API Key Secret**:
   - Execute the following command to create a Kubernetes secret containing your Datadog API key:
     ```bash
     kubectl create secret generic datadog-secret --from-literal api-key=8axxxxxxxxxxxxxxxxxxxxxxxxxxxf43
     ```

2. **Deploy the Datadog Agent in the Cluster**:
   - Add the Datadog Helm repository and update:
     ```bash
     helm repo add datadog https://helm.datadoghq.com
     helm repo update
     ```

   - Create and configure `datadog-values.yaml`:
     ```bash
     nano datadog-values.yaml
     ```
     Add the following content:
     ```yaml
     datadog:
       apiKeyExistingSecret: datadog-secret
     ```

   - Deploy the Datadog Agent:
     ```bash
     helm install datadog-agent -f datadog-values.yaml datadog/datadog
     ```

   - Confirm agents are running:
     ```bash
     kubectl get all
     ```

### **3. Prepare a Disk for Monitoring**
An alert will be triggered when the disk capacity reaches a determined threshold.

1. **Add a New Disk to the Server**:
   - In your VMware or AWS cloud instance, add a new 2GB disk named `/dev/sdb`.

#### **Adding a 2GB Disk in VMware Workstation**
1. **Open VMware Workstation**:
   - Launch VMware Workstation and select your virtual machine.

2. **Open VM Settings**:
   - Right-click on the virtual machine and select **Settings**.

3. **Add a New Disk**:
   - Click **Add** to open the **Add Hardware Wizard**.
   - Choose **Hard Disk** and click **Next**.
   - Select **SCSI** (recommended) or **IDE** and click **Next**.
   - Choose **Create a new virtual disk** and click **Next**.
   - Specify the disk size as **2 GB**.
   - Choose the location to store the virtual disk file and click **Next**.
   - Click **Finish** to create the disk.

#### **Adding a 2GB Disk in AWS**
1. **Log in to AWS Management Console**:
   - Navigate to [AWS Management Console](https://aws.amazon.com/console/).
   - Log in with your credentials.

2. **Navigate to EC2 Dashboard**:
   - Go to **Services** > **EC2**.

3. **Create a New EBS Volume**:
   - In the left sidebar, click on **Volumes** under **Elastic Block Store**.
   - Click **Create Volume**.
   - Configure the volume:
     - **Volume Type**: Choose `General Purpose SSD (gp3)`, `Provisioned IOPS SSD (io1)`, etc.
     - **Size**: Enter **2 GiB**.
     - **Availability Zone**: Select the same availability zone as your EC2 instance.
   - Click **Create Volume**.

4. **Attach the EBS Volume to an EC2 Instance**:
   - Go back to **Volumes**.
   - Select the volume you created.
   - Click **Actions** > **Attach Volume**.
   - Choose the instance you want to attach the volume to from the drop-down list.
   - Click **Attach**.

5. **Log in to Your EC2 or VMware Instance**.

6. **Prepare the Volume for Use**:
   - **Verify Disk**:
     ```bash
     lsblk
     ```
     The new disk should be listed as `/dev/xvdf` or similar.

2. **Install LVM**:
   - Install LVM tools:
     ```bash
     sudo apt update
     sudo apt install -y lvm2
     ```

3. **Set Up LVM**:
   - Create a Physical Volume (PV):
     ```bash
     sudo pvcreate /dev/sdb
     ```
   - Create a Volume Group (VG):
     ```bash
     sudo vgcreate demoVG /dev/sdb
     ```
   - Create a Logical Volume (LV) using all available space:
     ```bash
     sudo lvcreate -n demoLV -l 100%FREE demoVG
     ```

4. **Format and Mount the Logical Volume**:
   - Format the LV with ext4 filesystem:
     ```bash
     sudo mkfs.ext4 /dev/demoVG/demoLV
     ```
   - Create a mount point and mount the LV:
     ```bash
     sudo mkdir /demo
     sudo mount /dev/demoVG/demoLV /demo
     ```
   - **Verify the Mount**:
     ```bash
     df -h /demo
     ```

5. **Configure Automatic Mounting**:
   - Add the following entry to `/etc/fstab` for automatic mounting at boot:
     ```bash
     echo '/dev/demoVG/demoLV /demo ext4 defaults 0 2' | sudo tee -a /etc/fstab
     ```
   - **Verify fstab Configuration**:
     ```bash
     cat /etc/fstab
     ```

### **4. Set Up the Webhook HTTPS Listener Using Node.js**

1. **Install Node.js**:
   - Install the Node.js package repository and Node.js:
     ```bash
     curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
     sudo apt install -y nodejs
     ```
   - **Verify Installation**:
     ```bash
     node -v
     npm -v
     ```

2. **Create a Simple Node.js Webhook Listener**:
   - Create a directory for the webhook listener and navigate to it:
     ```bash
     mkdir ~/webhook_listener
     cd ~/webhook_listener
     ```
   - Initialize a new Node.js project:
     ```bash
     npm init -y
     ```
   - Install Express:
     ```bash
     npm install express
     ```
   - Create the `webhook_listener.js` file:
     ```bash
     nano webhook_listener.js
     ```
     - Add the following code to `webhook_listener.js`:
       ```javascript
       const express = require('express');
       const { exec } = require('child_process');
       
       const app = express();
       const port = 6060;
       
       app.post('/purge', (req, res) => {
           // Execute the purge script
           exec('/tmp/purge_demo.sh', (error, stdout, stderr) => {
               if (error) {
                   console.error(`Error executing script: ${error.message}`);
                   res.status(500).send('Internal Server Error');
                   return;
               }
               if (stderr) {
                   console.error(`Script stderr: ${stderr}`);
               }
               console.log(`Script output: ${stdout}`);
               res.send('Purge script executed');
           });
       });
       
       app.listen(port, () => {
           console.log(`Webhook listener running at http://localhost:${port}`);
       });
       ```
       
   - The service will be actively listening for incoming HTTP POST requests on port 6060. When Datadog triggers the webhook, it will send an HTTP or HTTPS POST request to this specific URL. This request will prompt the execution of the purge script.

3. **Make the Listener Persistent with PM2**:
   - Install PM2:
     ```bash
     sudo npm install -g pm2
     ```
   - Start the webhook listener with PM2:
     ```bash
     pm2 start webhook_listener.js
     ```
   - **Verify PM2 Process**:
     ```bash
     pm2 list
     pm2 stop webhook_listener.js
     pm2 restart webhook_listener.js
     ```
	 
### **Step 4: Test the Webhook Listener**

1. **Simulate a Webhook Request:**

   You can use `curl` to simulate a POST request to your webhook:

   ```bash
   curl -X POST http://localhost:8080/purge
   ```

   If everything is set up correctly, the Node.js script should execute `/tmp/purge_demo.sh` and return a confirmation message.

### **5. [Optional] Expose the Local Server with a VPN Tunnel**

1.

 **Install Localtunnel**:
   - Install Localtunnel:
     ```bash
     sudo npm install -g localtunnel
     ```

2. **Create a Tunnel**:
   - Start a Localtunnel to expose your local webhook listener:
     ```bash
     lt --port 6060 --subdomain trigger-xxxx
     ```
   - This screen output URL will look like `https://trigger-xxxx.loca.lt`.

3. **Get Tunnel Password**:
   - Access the tunnel password (if needed) for first-time access:
   - Open the URL in a browser and you may be requested to enter the tunnel password.
   
     ```bash
     wget -q -O - https://loca.lt/mytunnelpassword
     ```
   - The private IP displayed will be your password.

### **6. Configure Datadog Webhook**

1. **Create a Webhook in Datadog**:
   - Log in to Datadog and navigate to **Integrations** > Search for **Webhooks**.
   - Click **New Webhook** and configure it:
     - **Name**: `Purge Webhook`
     - **URL**: `https://trigger-xxxxx.loca.lt/purge`
     - **Additional Options**: Set as needed. [optional]
   - Click **Save**.

2. **Set Up a Datadog Monitor to Trigger the Webhook**:
   - Create a Monitor:
     - Navigate to Infrastructure in Datadog.
     - Click on the host where the disk is mounted and select **Create Monitor**.

   - Configure the Monitor:
     - Set the query to trigger an alert when disk usage exceeds 90%:
       ```bash
       max(last_5m):max:system.disk.in_use{device:/dev/mapper/demoVG-demoLV} by {device} > 0.9
       ```
     - Set the alert message:
       ```vbnet
       Alert: Disk usage on /demo has exceeded 90%. Triggering purge script.
       ```
     - Add recipients:
       ```less
       @your_email@domain.com @webhook-Run_Purge_Script
       ```

   - **Save and Activate the Monitor**:
     - Click **Save** to activate the monitor.

### **7. Verify the Self-Healing Process**

1. **Populate the `/demo` Directory**:
   - Open a separate session to the server.
   - Copy files to `/demo` directory until it reaches 95% capacity to simulate a full disk:
     ```bash
     dd if=/dev/zero of=/demo/testfile bs=1M count=1900
     ```

2. **Monitor Disk Usage**:
   - Run a continuous loop on your server to monitor the /demo directory:
     ```bash
     while true; do date && ls -l && pwd && du -ms; sleep 2; done
     ```
   - Ensure that the disk usage reaches 90% and triggers the Datadog monitor.

3. **Check Webhook Execution**:
   - Verify that the webhook is called and the purge script executes as expected.
   - An email will be automatically sent about the filled-up disk, and the `/demo` path will be cleaned up.
   - Check the `/demo` directory to ensure that files are deleted when the threshold is crossed.
   - Another email will be received to inform that the alert has been treated and closed.

---

By following these detailed steps, you'll have a robust self-healing system in place, with Datadog monitoring disk usage and triggering a Node.js webhook to automatically purge files when necessary.
