
## **Self-Healing and Monitoring: A comprehensive guide to revolutionizing System Resilience Through Automation**

In today’s fast-paced digital world, maintaining system reliability and minimizing downtime are critical for business success. This comprehensive guide explores how to enhance system resilience through advanced monitoring and self-healing mechanisms. We will walk you through integrating Datadog for monitoring, setting up automated recovery scripts, and leveraging Node.js and webhooks to create a reliable self-healing system (with a focus on disk management). By the end of this guide, you'll have a fully automated setup that can proactively manage system issues, ensuring smooth and uninterrupted operations.


### **Prerequisites**

- **A Datadog Account**: Create an active Datadog account for monitoring and alerts.
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

	<img width="684" alt="image" src="https://github.com/user-attachments/assets/dbcdd80d-d48b-4339-86e9-204ba16a7836">

 	<img width="799" alt="image" src="https://github.com/user-attachments/assets/c26f2caa-9c90-424d-94b9-54cedb6c5557">


   - **Install the Agent**:
     - Run the following command on your Ubuntu server to install the Datadog Agent:
       ```bash
       DD_API_KEY=8axxxxxxxxxxxxxxxxxxxxxxxxxxxf43 DD_SITE="datadoghq.com" bash -c "$(curl -L https://install.datadoghq.com/scripts/install_script_agent7.sh)"
       ```

	<img width="804" alt="image" src="https://github.com/user-attachments/assets/43ed04b5-cda0-400b-8c6c-5044322ba2c4">

     - This command sets up the agent and automatically configures it with your API key.

1. **Verify Agent Installation**:
   - Check the agent status to ensure it is running:
     ```bash
     systemctl status datadog-agent
     sudo datadog-agent status
     ```
   - You should see output indicating that the agent is running and sending data.
  
     <img width="949" alt="image" src="https://github.com/user-attachments/assets/7589f5b1-0761-4ea3-b227-468337853cfe">

   
   - To check logs, use:
     ```bash
     tail -f /var/log/datadog/agent.log
     ```

#### **For Ubuntu Server Managing a Kubernetes Cluster with `kubectl` and `helm` already installed:**
1. **Create a Datadog API Key Secret**:
   - Execute the following command to create a Kubernetes secret containing your Datadog API key:
     ```bash
     kubectl create secret generic datadog-secret --from-literal api-key=8axxxxxxxxxxxxxxxxxxxxxxxxxxxf43
     ```

2. **Deploy the Datadog Agent in the Cluster**:
   - Add the Datadog Helm repository and update:
     ```bash
     curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
     chmod 700 get_helm.sh
     ./get_helm.sh

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

     <img width="721" alt="image" src="https://github.com/user-attachments/assets/dcda0e71-4f88-47b6-8efc-98ceebbf8c09">

   - Confirm agents are running:
     ```bash
     kubectl get all
     ```

     <img width="766" alt="image" src="https://github.com/user-attachments/assets/c051b027-075f-4815-9875-2e81bb8d93b1">
     
### **3. Prepare a Disk for Monitoring**
An alert will be triggered when the disk capacity reaches a determined threshold.

1. **Add a New Disk to the Server**:
   - In your VMware or AWS cloud instance, add a new 2GB disk.

#### **Adding a 2GB Disk in VMware Workstation**
1. **Open VMware Workstation**:
   - Launch VMware Workstation and select your virtual machine.

2. **Open VM Settings**:
   - Right-click on the virtual machine and select **Settings**.
  
     <img width="559" alt="image" src="https://github.com/user-attachments/assets/4742ad6b-9623-4909-a15c-34da72c78046">

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
  
     <img width="608" alt="image" src="https://github.com/user-attachments/assets/993071f4-9505-4357-8259-a74ca1c03c97">


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

     <img width="300" alt="image" src="https://github.com/user-attachments/assets/b0c66092-c256-40b4-b157-ee9ee0697f1f">

2. **Install LVM**:
   LVM (Logical Volume Management) is used to manage disk volumes because it offers flexibility, efficiency, and scalability. It allows dynamic resizing of partitions, easy addition and removal of disks, and improved storage utilization through aggregation and thin provisioning. LVM enhances performance with striping, supports snapshots for backups, simplifies administration, and can be combined with mirroring or RAID for high availability, making it an ideal choice for environments with dynamic storage needs.
   
   - Install LVM tools:
     ```bash
     sudo apt update
     sudo apt install -y lvm2
     ```

4. **Set Up LVM**:
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

5. **Format and Mount the Logical Volume**:
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

     <img width="348" alt="image" src="https://github.com/user-attachments/assets/d637cd74-f4ea-4446-8575-2e269778aa09">

6. **Configure Automatic Mounting**:
   - Add the following entry to `/etc/fstab` for automatic mounting at boot:
     ```bash
     echo '/dev/demoVG/demoLV /demo ext4 defaults 0 2' | sudo tee -a /etc/fstab
     ```
   - **Verify fstab Configuration**:
     ```bash
     cat /etc/fstab
     ```

### **4. Set Up the Webhook HTTPS Listener Using Node.js**
If you prefer not to use Node.js, you may explore python Flask service, or Datadog’s serverless functions (if using a cloud provider like AWS) to trigger the script directly via AWS Lambda or an equivalent service, but the below method gives direct control on the infrastructure.

**First create the script, that would be triggered by Datadog’s Webhook Integration that clears/move log files in `/demo` directory.

  **Example script (`purge_demo.sh`)**:
  
  ```
  nano /tmp/purge_demo.sh
  ```
  
  ```bash
  #!/bin/bash
  
  LOGFILE="/tmp/purge_demo.log"
  
  echo "Running purge script at $(date)" >> $LOGFILE
  
  # Directory to purge
  TARGET_DIR="/demo"
  
  # Check if the directory exists
  if [ -d "$TARGET_DIR" ]; then
      echo "Purging all files in $TARGET_DIR..." >> $LOGFILE
      rm -rf ${TARGET_DIR}/* >> $LOGFILE 2>&1
      echo "All files in $TARGET_DIR have been purged." >> $LOGFILE
  else
      echo "Directory $TARGET_DIR does not exist." >> $LOGFILE
  fi
  ```
  
  - **Make the Script Executable**:
    ```bash
    chmod +x /path/to/purge_demo.sh
    ```

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

     <img width="179" alt="image" src="https://github.com/user-attachments/assets/3cedf108-f727-4986-9c60-27decf46f12e">


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
  
     <img width="431" alt="image" src="https://github.com/user-attachments/assets/494ce355-c1a1-4a94-92a6-68fdf0f97a8c">

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

       <img width="450" alt="image" src="https://github.com/user-attachments/assets/a63ad15f-795c-4f7e-bb92-c6a001abd848">

   - The service will be actively listening for incoming HTTP POST requests on port 6060. When Datadog triggers the webhook, it will send an HTTP or HTTPS POST request to this specific URL. This request will prompt the execution of the purge script.

3. **Make the Listener Persistent with PM2** (the service will continuously run in background):
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

     <img width="478" alt="image" src="https://github.com/user-attachments/assets/1b862e50-8224-4f86-aeb8-3b23e65352e5">

	 
### **Step 4: Test the Webhook Listener**

1. **Simulate a Webhook Request:**

   You can use `curl` to simulate a POST request to your webhook:

   ```bash
   curl -X POST http://localhost:6060/purge
   ```

   <img width="390" alt="image" src="https://github.com/user-attachments/assets/6d7e4276-64b9-4d59-b893-3b091da55fe0">


   If everything is set up correctly, the Node.js script should execute `/tmp/purge_demo.sh` and return a confirmation message.

### **5. [Optional] Expose the Local Server with a VPN Tunnel**, Just in case it is not a Linux cloud instance, it will need a temporary internet access through a vpn tunnel.

1. **Install Localtunnel**:
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
     - **Name**: `Run_Purge_Script`
     - **URL**: `https://trigger-xxxxx.loca.lt/purge` #for tunnel URL
     - OR
     - **URL**: `https://server_domainIP/purge` #for cloud instance
     - **Additional Options**: Set as needed. [optional]
   - Click **Save**.
  
     <img width="768" alt="image" src="https://github.com/user-attachments/assets/b74dde66-c30d-435e-ad65-4fd78c23623c">


     <img width="730" alt="image" src="https://github.com/user-attachments/assets/24ea75c5-008c-4cc1-86fe-963d3c425d13">


2. **Test that datadog can send a POST test request and Set Up a Datadog Monitor to Trigger the Webhook**:

   - On the monitoring page, navigate to `synthetic monitoring and testing` > `New Test`.
   - Click `New API test`, `HTTP`, `URL: POST`, `https://server_domainIP/purge`, > `send`.
   - You should get a success response as the screenshot below
  
    <img width="633" alt="image" src="https://github.com/user-attachments/assets/fcfcb04d-96ba-4238-ae20-28d04774bc4c">

   - Create a Monitor:
     - Navigate to Infrastructure in Datadog page.
     - Hover your mouse on the host and click on `view host dashboard`
       <img width="519" alt="image" src="https://github.com/user-attachments/assets/dad3b193-3d29-4384-bba8-06645c2f8f48">
     - A graphic display of some metrics you can monitor will be shown here.
     - Click on the metrics you want to monitor (e.g. `disk usage by device`), click **Create Monitor**.
    
      <img width="529" alt="image" src="https://github.com/user-attachments/assets/69d31ae7-ff16-45de-9d18-8c1698580713">

   - Configure the Monitor:
     - Set the query to trigger an alert when disk usage exceeds 90%:
       ```bash
       max(last_5m):max:system.disk.in_use{device:/dev/mapper/demoVG-demoLV} by {device} > 0.9
       ```
  
       <img width="709" alt="image" src="https://github.com/user-attachments/assets/3147f7b7-ee00-4c77-8e42-630c31195322">

     - Set the alert message:
       ```vbnet
       Alert: Disk usage on /demo has exceeded 90%. Triggering purge script.
       ```
     - Add recipients:
       ```less
       @your_email@domain.com @webhook-Run_Purge_Script
       ```
    
       Here, your webhook will also be a recipient, by simply typing the `@` key, in the message tab, a list of recipients will pop up for you to select.

       <img width="707" alt="image" src="https://github.com/user-attachments/assets/fa0aa0e6-7fc8-497f-b656-e1409255b5a0">

   - **Save and Activate the Monitor**:
     - Click **Save** to activate the monitor.
    
   - Navigate to `monitor section` to see a list of your configured monitors.

       <img width="793" alt="image" src="https://github.com/user-attachments/assets/5b7d66fc-069a-4f23-8b94-11c2560e9893">


### **7. Verify the Self-Healing Process**

1. **Populate the `/demo` Directory**:
   - Open a separate session to the server.
   - Copy files to `/demo` directory until it reaches 95% capacity to simulate a full disk:
     ```bash
     dd if=/dev/zero of=/demo/testfile bs=1M count=1900
     ```

2. **Monitor Disk Usage**: (open a new session to the server)
   - Run a continuous loop on your server to monitor the /demo directory:
     ```bash
     while true; do date && ls -l && pwd && du -ms; sleep 2; done
     ```
     - This command shows me the real time date, list of files, size of files and current working directory of `/demo` partition.
    
       <img width="434" alt="image" src="https://github.com/user-attachments/assets/0a1ecfd5-8e53-44aa-b48d-2a54da009489">

   - Ensure that the disk usage reaches 90% and triggers the Datadog monitor.

3. **Check Webhook Execution**:
   - Verify that the webhook is called and the purge script executes as expected.
     
     <img width="799" alt="image" src="https://github.com/user-attachments/assets/eba3fa3f-e11a-4dcf-b373-38207e9e908c">

   - An email will be automatically sent about the filled-up disk, and the `/demo` partition will be cleaned up.

     ![Screenshot 2024-08-26 194702](https://github.com/user-attachments/assets/a324020f-d595-4aa7-9b36-83dfc0cbb061)

     You will notice that the time that the email was triggered and the time the disk was full, are the same.
     Within seconds, the script has been executed and any upcoming disruption would have been averted.

   - Check the `/demo` directory to ensure that files are deleted when the threshold is crossed.
  
     ![Screenshot 2024-08-26 194210](https://github.com/user-attachments/assets/2003c5ff-7a68-4e8f-8fb2-63b82a563ebf)

   - Another email will be received to inform that the alert has been treated and closed.
  
     ![Screenshot 2024-08-26 194746](https://github.com/user-attachments/assets/ff522e29-79e2-4f29-97dd-a823dc63f501)



By following these detailed steps, you'll establish a realistic self-healing system. While the script provided focuses on clearing logs, it can be adapted to perform other actions, such as restarting a service, scaling an instance, or any other task. With Datadog monitoring disk usage and triggering a Node.js webhook, the system will automatically execute the necessary script, ensuring responsive and efficient management of your infrastructure.
Please feel free to leave a like, comment, or ask a question if you need clarity on any of the steps. Happy Learning!
