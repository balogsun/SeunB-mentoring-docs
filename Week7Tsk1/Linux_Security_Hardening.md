# **Securing your Cloud Infrastructure: A comprehensive guide to hardening, scaling, automating and monitoring your servers**

## **Introduction:**

In today's fast-paced tech environment, launching a new product often requires setting up a secure and scalable web server cluster quickly and efficiently. As a DevOps engineer at a growing startup, your role is critical in ensuring that these servers are not only operational but also secure against potential threats. This guide takes you through a real-world scenario of securing a new web server cluster from scratch, covering everything from initial server hardening to advanced automation and monitoring techniques. By the end, you'll have a robust, repeatable process for securing and managing web servers, ready to be scaled across your infrastructure.

Let's walk through creating a repeatable process that can be applied to multiple servers, ensuring they are all configured securely and consistently.
I will be hardening the system, setting up file integrity monitoring, system audit, central logging solution and finally automating the process with ansible.

Below is a bash script for basic system hardening of an Ubuntu server. It can definitely be applied to some other linux flavous as needed. The script covers various security measures to ensure the server is well-protected. After the script, I'll explain each stage in detail.

### Bash Script: `nano system_hardening.sh`

```bash
#!/bin/bash

# Ensure the script is run as root
if [ "$(id -u)" -ne 0 ]; then
    echo "This script must be run as root. Exiting."
    exit 1
fi

# Update and upgrade the system
echo "Updating and upgrading the system..."
apt update && apt upgrade -y

# 1. Disable root login via SSH
echo "Disabling root login via SSH..."
sed -i 's/^PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config


# 2. Enable logging of sudo commands
echo "Enabling logging of sudo commands..."
echo "Defaults logfile=/var/log/sudo.log" >> /etc/sudoers

# 3. Install and configure UFW (Uncomplicated Firewall)
echo "Installing and configuring UFW..."
apt install -y ufw
ufw default deny incoming
ufw default allow outgoing
ufw allow OpenSSH
ufw enable

# 4. Disable unused filesystems
echo "Disabling unused filesystems..."
echo "install cramfs /bin/true" >> /etc/modprobe.d/disable-filesystems.conf
echo "install freevxfs /bin/true" >> /etc/modprobe.d/disable-filesystems.conf
echo "install jffs2 /bin/true" >> /etc/modprobe.d/disable-filesystems.conf
echo "install hfs /bin/true" >> /etc/modprobe.d/disable-filesystems.conf
echo "install hfsplus /bin/true" >> /etc/modprobe.d/disable-filesystems.conf
echo "install udf /bin/true" >> /etc/modprobe.d/disable-filesystems.conf

# 5. Set strong password policies
echo "Setting strong password policies..."
apt install -y libpam-pwquality
sed -i 's/^# minlen.*/minlen = 12/' /etc/security/pwquality.conf
sed -i 's/^# dcredit.*/dcredit = -1/' /etc/security/pwquality.conf
sed -i 's/^# ucredit.*/ucredit = -1/' /etc/security/pwquality.conf
sed -i 's/^# lcredit.*/lcredit = -1/' /etc/security/pwquality.conf
sed -i 's/^# ocredit.*/ocredit = -1/' /etc/security/pwquality.conf

# 6. Disable IPv6 if not needed
echo "Disabling IPv6..."
sed -i 's/^#net.ipv6.conf.all.disable_ipv6 = 1/net.ipv6.conf.all.disable_ipv6 = 1/' /etc/sysctl.conf
sysctl -p

# 7. Secure shared memory
echo "Securing shared memory..."
echo "tmpfs /run/shm tmpfs defaults,noexec,nosuid 0 0" >> /etc/fstab

# 8. Remove unnecessary packages
echo "Removing unnecessary packages..."
apt-get autoremove -y --purge

# 9. Install and configure rkhunter
echo "Installing and configuring rkhunter..."
apt install -y rkhunter
rkhunter --update
rkhunter --propupd
rkhunter --check --sk

# 10. Set up log file permissions
echo "Setting up log file permissions..."
chmod -R go-rwx /var/log/*

# 11. Configure login banner
echo "Configuring login banner..."
echo "Authorized access only. All activity may be monitored and reported." > /etc/issue.net
sed -i 's/^#Banner.*/Banner \/etc\/issue.net/' /etc/ssh/sshd_config

# 12. Set up limits on user processes
echo "Setting up limits on user processes..."
echo "* hard nproc 100" >> /etc/security/limits.conf

# 13. Restrict cron jobs to authorized users
echo "Restricting cron jobs to authorized users..."
touch /etc/cron.allow
chmod 600 /etc/cron.allow

# 14. Restrict access to su command
echo "Restricting access to su command..."

# Ensure required package is installed.
apt install -y libpam-modules

# Configure PAM to restrict access to su command.
if ! grep -q "auth required pam_wheel.so use_uid" /etc/pam.d/su; then
    echo "auth required pam_wheel.so use_uid" >> /etc/pam.d/su
fi

# Create the 'wheel' group if it does not exist
if ! getent group wheel > /dev/null; then
    echo "Creating 'wheel' group..."
    groupadd wheel
fi

# Add the 'ubuntu' user to the 'wheel' group
echo "Adding 'ubuntu' user to 'wheel' group..."
usermod -aG wheel ubuntu

echo "su command access restricted to 'wheel' group members, including 'ubuntu' user."

# 15. Disable core dumps
echo "Disabling core dumps..."
echo "* hard core 0" >> /etc/security/limits.conf
echo "fs.suid_dumpable = 0" >> /etc/sysctl.conf
sysctl -p

# 16. Enable ASLR (Address Space Layout Randomization)
echo "Enabling ASLR..."
echo "kernel.randomize_va_space = 2" >> /etc/sysctl.conf
sysctl -p

# Restart SSH to apply changes
echo "Restarting SSH service..."
systemctl restart ssh

echo "System hardening completed."
```

Make the script executable and run it.

```
chmod 764 system_hardening.sh
```

```
./chmod 764 system_hardening.sh
```
Snippet of the script running

![Screenshot 2024-08-09 100220](https://github.com/user-attachments/assets/0ceddca4-dd50-4845-acac-d59a352e5d4a)


### Summary of Hardening Steps

Update the server packages.

1. **Disable root login via SSH**: Prevents direct root login, reducing the risk of brute force attacks on the root account.
2. **Enable logging of sudo commands**: Tracks all commands executed with `sudo`, enhancing accountability and auditability.
3. **Install and configure UFW (Uncomplicated Firewall)**: Sets up a simple firewall to manage incoming and outgoing traffic.
4. **Disable unused filesystems**: Prevents the loading of unnecessary and potentially vulnerable filesystems.
5. **Set strong password policies**: Enforces strong password requirements to reduce the risk of password-based attacks. These settings enforce a minimum password length of 12 characters with at least one digit, uppercase letter, lowercase letter, and special character.
6. **Disable IPv6 if not needed**: Disables IPv6 to reduce the attack surface if it's not in use.
7. **Secure shared memory**: Protects shared memory by mounting it with restrictive options.
8. **Remove unnecessary packages**: Reduces the attack surface by removing unused packages.
9. **Install and configure rkhunter**: Checks for rootkits and other security issues on the server.
10. **Set up log file permissions**: Ensures that log files are protected from unauthorized access.
11. **Configure login banner**: Displays a warning message to users before login, indicating monitoring and restrictions.
12. **Set up limits on user processes**: Limits the number of processes a user can run, preventing fork bomb attacks.
13. **Restrict cron jobs to authorized users**: Only allows authorized users to create and edit cron jobs, reducing the risk of malicious cron tasks.
14. **Restrict access to su command**: Limits access to the `su` command to authorized users, preventing unauthorized privilege escalation.
15. **Disable core dumps**: Prevents the creation of core dumps, which could contain sensitive information.
16. **Enable ASLR (Address Space Layout Randomization)**: Randomizes memory addresses, making it more difficult for attackers to exploit vulnerabilities.

This script now focuses on securing a Linux server by implementing a set of essential hardening measures without adding new tools or services beyond those already included.

This script is designed to cover a wide range of basic security measures that are generally applicable to many environments. For your specific production environment, additional hardening steps might be needed depending on the specific use case.

#### Now lets test some of the functionalities that the script has implemented

- vagrant user is not a member of wheel and was not able to perform su functions, unlike the ubuntu user.

  ![Screenshot 2024-08-09 100426](https://github.com/user-attachments/assets/3ff910f6-31bf-4076-8830-dfadd04cc8e5)

- vagrant user tried to change password that did not meet up with strong password policies and that attempt failed.

  ![Screenshot 2024-08-09 100829](https://github.com/user-attachments/assets/025ef59d-faa5-4edd-8104-dd5206aa020a)

- display banner is shown whenever a user logs in.
  ![Screenshot 2024-08-09 105721](https://github.com/user-attachments/assets/f0404282-ee6b-40c2-9937-0cbf654ee78f)

- root user could no longer make a direct ssh login

## Next we can install and setup `Fail2ban`

A security tool designed to protect servers from brute-force attacks, protecting services such as SSH, HTTP, and FTP  by monitoring log files for suspicious activity and automatically banning offending IP addresses.
 It works by defining "jails" for specific services like SSH, HTTP, and FTP, where it monitors for failed login attempts or other malicious behavior. When a threshold of failures is reached, Fail2ban modifies firewall rules to block the IP for a set period. The tool is highly configurable, allowing custom filters and flexible ban durations, and it automatically unbans IPs after the ban period expires.

 Below is a script that will install and setup a basic configuration that provides a strong level of security, but you can further tailor it to meet specific needs by creating custom jails and filters.

```bash
#!/bin/bash

# Fail2ban installation and configuration script

echo "Updating package list..."
sudo apt update

echo "Installing Fail2ban..."
sudo apt install -y fail2ban

# Create a local copy of the jail configuration file for custom settings if not already present
echo "Creating local configuration file..."
if [ ! -f /etc/fail2ban/jail.local ]; then
    sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
fi

# Backup existing jail.local before modifying
echo "Backing up existing jail.local..."
sudo cp /etc/fail2ban/jail.local /etc/fail2ban/jail.local.bak

# Append new configuration settings to jail.local
echo "Configuring Fail2ban..."
sudo bash -c 'echo -e "[DEFAULT]\n# Whitelist your own IPs\nignoreip = 127.0.0.1/8 ::1 192.168.5.0/24 192.168.79.0/24\n\n# Ban time in seconds (e.g., 10 minutes)\nbantime = 600\n\n# Time frame for counting failures in seconds\nfindtime = 600\n\n# Max retry attempts before banning\nmaxretry = 5\n\n[sshd]\nenabled = true\nport = ssh\nlogpath = %(sshd_log)s\nmaxretry = 5" | sudo tee /etc/fail2ban/jail.local > /dev/null'

# Restart and enable Fail2ban service
echo "Starting and enabling Fail2ban service..."
sudo systemctl restart fail2ban
sudo systemctl enable fail2ban

# Display the status of the Fail2ban service
echo "Checking Fail2ban status..."
sudo systemctl status fail2ban

# Optional: Monitor Fail2ban logs
echo "You can monitor Fail2ban logs with the following command:"
echo "sudo tail -f /var/log/fail2ban.log"

echo "Fail2ban installation and configuration completed."
```

### How to Use the Script

2. **Make the Script Executable:**
   Run the following command to make the script executable:

   ```bash
   chmod +x install_fail2ban.sh
   ```

3. **Run the Script:**
   Execute the script with `sudo` to install and configure Fail2ban:

   ```bash
   sudo ./install_fail2ban.sh
   ```

### What This Script Does

- **Installs Fail2ban** using the package manager.
- **Creates a local configuration file** (`jail.local`) to customize settings without affecting the default `jail.conf`.
- **Configures Fail2ban** with some basic settings:
  - Whitelisting local IPs.
  - Setting ban time, find time, and maximum retries.
  - Enabling and configuring the SSH jail.
- **Starts and enables** the Fail2ban service to run at boot.
- **Provides the status** of the Fail2ban service for verification.
- **Offers a command** to monitor Fail2ban logs for real-time information.

### Other commands to try

List Banned IPs

```bash
sudo fail2ban-client status ssh
```

<img width="373" alt="image" src="https://github.com/user-attachments/assets/97bf746d-0335-4a22-858a-009b7e2f91a4">

This will show the number of currently banned IPs and the list of banned IPs.

If you need to unban an IP address manually:

```
sudo fail2ban-client set sshd unbanip <IP_ADDRESS>
```

- Replace <IP_ADDRESS> with the actual IP address you want to unban.

This script automates the entire process of installing and configuring Fail2ban, making it easy to secure your server against brute-force attacks.

Now this same server has an alternate IP, i attempted to make several failed attempts to it and here are the logs showing the IP being banned, and future connections from that IP were no longer permitted.

<img width="609" alt="image" src="https://github.com/user-attachments/assets/bdcac231-8d1a-409f-a17b-1b373e8eec1f">


## Next, Lets install and configure a basic web server `apache`

To install and configure a web server on Ubuntu, you can use Apache or nginx. I have created a custom html document (a simple pizza booking website) for this purposea and here's a step-by-step guide to install and configure Apache: 

### Bash Script for Installing and Configuring Apache

```
nano apache.sh
```

```bash
#!/bin/bash

# Web server installation and configuration script

echo "Installing Apache..."
sudo apt install -y apache2

# Ensure Apache is running and enabled on boot
echo "Starting and enabling Apache service..."
sudo systemctl start apache2
sudo systemctl enable apache2

# Configure firewall to allow HTTP and HTTPS traffic
echo "Configuring firewall..."
sudo ufw allow 'Apache Full'

# Create a simple HTML file for the custom site

echo "Creating a pizza ordering web page..."
echo '<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Order Your Pizza</title>
    <style>
        body {
            background-color: #f4f4f9;
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            color: #333;
            text-align: center;
        }
        header {
            background-color: #808080; /* Grey */
            color: white;
            padding: 20px;
        }
        h1 {
            margin: 0;
            font-size: 2.5em;
        }
        h2 {
            font-size: 1.5em;
            margin-top: 0.5em;
            color: #555;
        }
        h3 {
            font-size: 1.2em;
            margin-top: 1em;
            color: #333;
        }
        .container {
            max-width: 800px;
            margin: 20px auto;
            padding: 20px;
            background-color: white;
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        }
        .form-group {
            margin-bottom: 15px;
            text-align: left;
        }
        .form-group input {
            width: 100%;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 4px;
        }
        .toppings {
            display: flex;
            justify-content: center;
            flex-wrap: wrap;
        }
        .toppings div {
            margin: 10px;
            font-size: 1.1em;
        }
        .button {
            background-color: #4CAF50; /* Green */
            border: none;
            color: white;
            padding: 15px 25px;
            text-align: center;
            text-decoration: none;
            display: inline-block;
            font-size: 16px;
            margin: 10px;
            border-radius: 5px;
            cursor: pointer;
            transition: background-color 0.3s ease;
        }
        .button:hover {
            background-color: #45a049;
        }
        .button.reset {
            background-color: #e7e7e7; /* Gray */
            color: black;
        }
        .button.reset:hover {
            background-color: #d5d5d5;
        }
        img {
            max-width: 100%;
            height: auto;
            margin-top: 20px;
            border-radius: 8px;
        }
        footer {
            margin-top: 20px;
            font-size: 14px;
            color: #777;
        }
    </style>
</head>
<body>
    <header>
        <h1>Your One Stop Shop for Great Pizzas</h1>
    </header>
    <div class="container">
        <h2>Kindly Make Your Order by Filling the Details Below</h2>
        <form>
            <div class="form-group">
                <label for="name">Name:</label>
                <input type="text" id="name" placeholder="Your Name" required>
            </div>
            <div class="form-group">
                <label for="email">Email:</label>
                <input type="email" id="email" placeholder="Your Email" required>
            </div>
            <div class="form-group">
                <label for="phone">Phone No:</label>
                <input type="tel" id="phone" placeholder="Your Phone Number" required>
            </div>
            <h3>Select Pizza Toppings</h3>
            <div class="toppings">
                <div><input type="checkbox" id="chicken"><label for="chicken"> Chicken</label></div>
                <div><input type="checkbox" id="pepperoni"><label for="pepperoni"> Pepperoni</label></div>
                <div><input type="checkbox" id="sausage"><label for="sausage"> Sausage</label></div>
                <div><input type="checkbox" id="mushrooms"><label for="mushrooms"> Mushrooms</label></div>
            </div>
            <button type="submit" class="button">Submit</button>
            <button type="reset" class="button reset">Reset</button>
        </form>
    </div>
    <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/9/91/Pizza-3007395.jpg/1280px-Pizza-3007395.jpg" alt="Pizza">
    <footer>
        <h4>All Rights Reserved</h4>
    </footer>
</body>
</html>' | sudo tee /var/www/html/index.html > /dev/null

# Remove the default site configuration

echo "Removing the default site configuration..."
sudo a2dissite 000-default.conf

# Create and enable custom site configuration

echo "Creating custom site configuration..."
sudo tee /etc/apache2/sites-available/custom-site.conf > /dev/null <<EOF
<VirtualHost *:80>
    DocumentRoot /var/www/html
    <Directory /var/www/html>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
EOF

echo "Enabling the custom site configuration..."
sudo a2ensite custom-site.conf

# Restart Apache to apply any configuration changes

echo "Restarting Apache..."
sudo systemctl restart apache2

# Display Apache status

echo "Checking Apache status..."
sudo systemctl status apache2

echo "Apache installation and configuration completed. You can access your web server at http://<your-server-ip>."

```

 **Save the Script**: Save the script as `install_configure_apache.sh`.
 **Make it Executable**:
   ```bash
   chmod +x apache.sh
   ```

 **Run the Script**:

   ```bash
   sudo ./apache.sh
   ```
**A snippet of the script when run**
![Screenshot 2024-08-09 113613](https://github.com/user-attachments/assets/509e8834-6c3b-473e-86d8-914d846080cc)

### Accessing the Web Server

Once the script completes, you can access the web server by navigating to `http://<your-server-ip>` in your web browser. You should see the simple HTML page you created.

![Screenshot 2024-08-09 113842](https://github.com/user-attachments/assets/982eba30-81a7-4f71-8a12-d3c1572ec461)


### Next lets setup File Integrity Monitoring with AIDE (Advanced Intrusion Detection Environment),  to help you monitor file integrity effectively and receive daily reports

To implement File Integrity Monitoring with AIDE (Advanced Intrusion Detection Environment), follow these steps:
We will also be setting up mail services using gmail.

One Needs to Have 2-Step Verification Enabled for Google mail Account as Less Secure Apps has been deprecated.

1. Navigate to `https://myaccount.google.com/apppasswords`.
2. Name and Create your app, then click the `Create` button.
3. A new app password will be generated for you. Copy this password.

### 1. **Install AIDE**

```
# !/bin/bash

# Install AIDE

echo "Installing AIDE..."
sudo apt-get install -y aide

# Backup the existing AIDE configuration file

echo "Backing up AIDE configuration..."
sudo cp /etc/aide/aide.conf /etc/aide/aide.conf.bkup

# Add exclusions to AIDE configuration file

echo "Configuring AIDE exclusions..."
echo "!/tmp" | sudo tee -a /etc/aide/aide.conf > /dev/null
echo "!/var/spool" | sudo tee -a /etc/aide/aide.conf > /dev/null

# Initialize AIDE database

echo "Initializing AIDE database. This may take a while..."
sudo aide -c /etc/aide/aide.conf --init

# Move the new AIDE database to the correct location

echo "Moving AIDE database..."
sudo mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db

# Run an AIDE check against the initialized database

echo "Running AIDE check. This may take a while..."
sudo aide -c /etc/aide/aide.conf --check

# Install Postfix

echo "Installing Postfix..."
sudo apt-get install -y postfix

# Configure Postfix for Gmail relay

echo "Configuring Postfix..."
sudo sed -i '/^relayhost =/d' /etc/postfix/main.cf
echo "relayhost = [smtp.gmail.com]:587" | sudo tee -a /etc/postfix/main.cf > /dev/null
echo "smtp_use_tls = yes" | sudo tee -a /etc/postfix/main.cf > /dev/null
echo "smtp_sasl_auth_enable = yes" | sudo tee -a /etc/postfix/main.cf > /dev/null
echo "smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd" | sudo tee -a /etc/postfix/main.cf > /dev/null
echo "smtp_sasl_security_options = noanonymous" | sudo tee -a /etc/postfix/main.cf > /dev/null

# Create sasl_passwd file for Postfix

echo "Creating SASL password file..."
sudo touch /etc/postfix/sasl_passwd
echo "[smtp.gmail.com]:587 <balogsun@gmail.com>:hbstkssqemmjfqyi" | sudo tee -a /etc/postfix/sasl_passwd > /dev/null

# Secure the SASL password file and update Postfix lookup table

echo "Securing SASL password file..."
sudo chmod 600 /etc/postfix/sasl_passwd
sudo postmap /etc/postfix/sasl_passwd

# Allow ports 25 and 587 on firewall

echo "Allowing ports 25 and 587 on firewall..."
sudo ufw allow out 25
sudo ufw allow out 587

# Restart Postfix to apply changes

echo "Restarting Postfix..."
sudo systemctl restart postfix

# Test the Postfix configuration

echo "Testing Postfix configuration by sending a test email..."
echo "If you get this mail, it means the mail application is working" | mail -s "Postfix mail from <ServerA@seunb.com>" <balogsun@gmail.com>

# Print mail logs

echo "Printing mail logs..."
tail /var/log/mail.log

# Set up a cron job for daily AIDE checks and email reports

echo "Setting up daily AIDE checks cron job..."
(crontab -l 2>/dev/null; echo "0 2 ** * /usr/bin/aide -c /etc/aide/aide.conf --check | mail -s 'AIDE Daily Report' <user@gmail.com>") | crontab -

echo "Setup complete. Check your email for the test message and AIDE reports."

```

#### Screenshot showing Email sent successfully from the analysis of `AIDE Daily Report`

<img width="564" alt="image" src="https://github.com/user-attachments/assets/a4458d67-7088-45cf-a979-c3a44d7e47ec">


### Steps for Conducting a Security Audit with Lynis

1. **Install Lynis**
   - Open your terminal and update the package list:

     ```bash
     sudo apt-get update
     ```

   - Install Lynis:

     ```bash
     sudo apt-get install -y lynis
     ```

2. **Run a System Audit**
   - Execute the Lynis system audit: The audit process will take some time and will generate output directly in the terminal. To save the report to a file for easier review, you can redirect the output:

     ```bash
     sudo lynis audit system > /var/log/lynis-audit.log
     ```

3. **Review the Lynis Report**
   - Open and review the Lynis report, from the saved log file:

     ```bash
     sudo tail /var/log/lynis-audit.log
     ```

     ![Screenshot 2024-08-09 231213](https://github.com/user-attachments/assets/bf284bc0-d13f-4ec2-9106-df609fbd5518)

     ![Screenshot 2024-08-09 231106](https://github.com/user-attachments/assets/ca6a427b-60e6-427a-b3fc-00c43f2f5f54)

     ![Screenshot 2024-08-09 231129](https://github.com/user-attachments/assets/671169e6-1d50-4d73-8311-79731a16fed9)



### Next, we will be reviewing the Lynis report, and address at least 5 medium or high-risk findings, and then document the changes made to fix the findings with justifications.

### 1. **Weak File Permissions on Critical Directories**

- **Finding**: Directories such as `/etc/cron.d`, `/etc/cron.daily`, `/etc/cron.hourly`, `/etc/cron.weekly`, and `/etc/cron.monthly` have permissive permissions.
- **Impact**: Loose file permissions on these directories could allow unauthorized users to modify scheduled tasks, potentially leading to privilege escalation or system compromise.
- **Action**: Tighten permissions on these directories.
- **Steps**:
     1. **Check Current Permissions**:

        ```bash
        ls -ld /etc/cron.*
        ```

        - Review the permissions to ensure they are not overly permissive.
     2. **Set Secure Permissions**:

        ```bash
        sudo chmod 600 /etc/cron.d
        sudo chmod 600 /etc/cron.daily
        sudo chmod 600 /etc/cron.hourly
        sudo chmod 600 /etc/cron.weekly
        sudo chmod 600 /etc/cron.monthly

  sudo chmod 600 /etc/cron.yearly
        ```
     3. **Verify the Change**:
        ```bash
        ls -ld /etc/cron.*
        ```
        - Ensure the permissions are set to `600` (i.e., `-rw-------`), meaning only the root user can read and write.

- **Justification**: Restricting access to cron directories prevents unauthorized users from tampering with scheduled tasks, which is critical for maintaining system integrity.

<img width="379" alt="Screenshot 2024-08-09 233208" src="https://github.com/user-attachments/assets/c6550271-1c60-4c5e-a246-f9727c43f117">

### 2. **Weak File Permissions on Sensitive Files**

- **Finding**: Some files, such as `/etc/ssh/sshd_config`, and directories like `/etc/cron.d`, have weak permissions.
- **Impact**: Inadequate file permissions can allow unauthorized users to modify critical configuration files or execute malicious cron jobs.
- **Action**: Adjust file and directory permissions to ensure they are secure.
- **Steps**:
     1. **Check Current Permissions**:

        ```bash
        ls -l /etc/ssh/sshd_config
        ```

        - Review the permissions to ensure they are not overly permissive.
     2. **Set Secure Permissions**:

        ```bash
        sudo chmod 600 /etc/ssh/sshd_config
        ```

     3. **Verify the Change**:

        ```bash
        ls -l /etc/ssh/sshd_config
        ```

        - Ensure the permissions are set to `600` (i.e., `-rw-------`), meaning only the root user can read and write.

- **Justification**: Proper permissions help protect sensitive files and configurations from unauthorized access and modification.

### 3. **Missing Password Aging Configurations**

- **Finding**: User password aging settings are disabled (`/etc/login.defs`).
- **Impact**: Lack of password aging increases the risk of password-related vulnerabilities as passwords may not be changed regularly.
- **Action**: Enable and configure password aging settings in `/etc/login.defs` to enforce regular password changes.
- **Steps**:
     1. **Edit the Login Definitions File**:

        ```bash
        sudo nano /etc/login.defs
        ```

     2. **Configure Password Aging Settings**:
        - Set the minimum number of days between password changes:

          ```bash
          PASS_MIN_DAYS 7
          ```

        - Set the maximum number of days a password is valid:

          ```bash
          PASS_MAX_DAYS 90
          ```

        - Set the number of days before password expiration that the user is warned:

          ```bash
          PASS_WARN_AGE 10
          ```

     3. **Apply the Changes**:
        - Apply these settings to all user accounts:

          ```bash
          sudo chage --mindays 7 --maxdays 90 --warndays 14 ubuntu
          ```

     4. **Verify the Settings**:

        ```bash
        sudo chage -l ubuntu
        ```

        - Ensure the correct password aging policies are applied.

- **Justification**: Enforcing password aging helps ensure that passwords are updated regularly, reducing the risk of compromised accounts.

![Screenshot 2024-08-09 233815](https://github.com/user-attachments/assets/c6dba8c7-1eca-4009-8823-07633742198f)

<img width="469" alt="image" src="https://github.com/user-attachments/assets/a83dcc70-4d36-4526-aa82-539ae2d591b9">

### 4. **Lack of Sticky Bit on /tmp Directory**

- **Finding**: The sticky bit is not set on the `/tmp` directory.
- **Impact**: Without the sticky bit, users can delete or modify files in `/tmp` that are owned by other users, which could lead to privilege escalation or data tampering.
- **Action**: Set the sticky bit on the `/tmp` directory.
- **Steps**:
     1. **Check Current Permissions**:

        ```bash
        ls -ld /tmp
        ```

        - This command will show the current permissions of the `/tmp` directory. Look for a `t` at the end of the permission string (e.g., `drwxrwxrwt`), which indicates the sticky bit is set.
     2. **Set the Sticky Bit**:

        ```bash
        sudo chmod +t /tmp
        ```

     3. **Verify the Change**:

        ```bash
        ls -ld /tmp
        ```

        - Ensure the sticky bit is now set (`drwxrwxrwt`).

- **Justification**: Applying the sticky bit restricts file deletions in `/tmp` to the file's owner, thereby improving security and preventing unauthorized file manipulations.

![Screenshot 2024-08-09 234033](https://github.com/user-attachments/assets/c39f1d74-48c4-45c6-a88d-98600a40aa72)

### 5. **Unrestricted NTP Service Access**

- **Finding**: NTP (Network Time Protocol) service is running without any access control.
- **Impact**: An unrestricted NTP service can be abused for DDoS amplification attacks or for manipulating the system clock.
- **Action**: Configure the NTP service to restrict access.
- **Steps**:
     1. **Edit the NTP Configuration File**:

        ```bash
        sudo nano /etc/ntp.conf
        ```

     2. **Add Access Control Restrictions**:
        - Add the following line to restrict default access:

          ```bash
          restrict default kod nomodify notrap nopeer noquery
          ```

     3. **Restart the NTP Service**:

        ```bash
        sudo systemctl restart ntp
        ```

     4. **Verify the Configuration**:

        ```bash
        ntpq -p
        ```

        - This command will show the status of the NTP peers and the current configuration.

- **Justification**: Restricting NTP access helps prevent abuse of the service and ensures the system clock's integrity.

These steps will help address the findings identified in the security audit, ensuring the system is better protected against various threats and vulnerabilities.

## Understanding CIS Benchmarks

**CIS (Center for Internet Security) Benchmarks** are consensus-based, best-practice security configuration guides. They are designed to help organizations secure their IT systems and data against cyber threats. These benchmarks are widely recognized as industry standards for hardening systems and improving their security posture.

### Using Tools Like oscap (OpenSCAP) or InSpec to Apply and Check CIS Benchmarks

To run InSpec against a Linux Ubuntu host, follow these steps to install InSpec, run a pre-built CIS benchmark profile, and generate a report.

### 1. **Install InSpec on the Ubuntu Host**

First, you'll need to install InSpec on your Ubuntu system.

#### A. **Install via Omnitruck Script (Recommended)**

This is the easiest method to install InSpec.

```bash
curl https://omnitruck.chef.io/install.sh | sudo bash -s -- -P inspec
```

![Screenshot 2024-08-12 110005](https://github.com/user-attachments/assets/f70310f7-4aab-40a1-9fa8-0a881032c55b)

![Screenshot 2024-08-12 110139](https://github.com/user-attachments/assets/27e87272-7599-47d7-bab5-eb098ed7c5c0)

#### B. **Install via APT Repository (Alternative Method)**

1. **Add the Chef APT repository:**

   ```bash
   echo "deb [arch=amd64] https://packages.chef.io/repos/apt/stable $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/chef-stable.list
   ```

2. **Add the GPG key:**

   ```bash
   curl https://packages.chef.io/chef.asc | sudo apt-key add -
   ```

3. **Update the package list and install InSpec:**

   ```bash
   sudo apt-get update
   sudo apt-get install inspec
   ```

### 2. **Obtain a CIS Benchmark Profile**

You can use a pre-built CIS benchmark profile. One such profile is provided by the DevSec Project.

#### A. **Download the CIS Benchmark Profile**

For example, to download and execute the CIS benchmark for Linux:

```bash
inspec supermarket exec dev-sec/linux-baseline
```

This command downloads the profile and executes it on your local machine.

### 3. **Run the InSpec Profile Against the Ubuntu Host**

Once you have the InSpec profile, you can run it on the host machine.

#### A. **Run the Profile Locally on the Ubuntu Host**

```bash
inspec exec https://github.com/dev-sec/linux-baseline
git clone https://github.com/dev-sec/linux-baseline.git
cd linux-baseline
inspec exec .

```

This command will execute the CIS benchmark checks against your Ubuntu system.

#### B. **Run the Profile and Generate a Report**

You can generate a report in various formats, such as JSON or HTML, for further analysis.

- **Generate an HTML Report:**

  ```bash
  inspec exec https://github.com/dev-sec/linux-baseline --reporter html:report.html
  inspec exec . --reporter html:report.html
  ```

 <img width="316" alt="image" src="https://github.com/user-attachments/assets/c9ef9c2b-72b5-41ef-b8c7-142c0eae9063">


- **Generate a JSON Report:**

  ```bash
  inspec exec dev-sec/linux-baseline --reporter json:report.json
  ```

### 4. **Review the Report**

- If you generated an HTML report, you can ship out the report to your local machine and then open it in a web browser:

- If you generated a JSON report, you can parse it or use it as input for other tools or automation processes.

  ![Screenshot 2024-08-12 120733](https://github.com/user-attachments/assets/fd2c7695-c388-4501-b84a-44327328fa0f)

  ![Screenshot 2024-08-12 120852](https://github.com/user-attachments/assets/99af79df-25b8-405a-8cff-1e5a0d3dc2be)


### 5. **(Optional) Schedule Regular Scans**

You can automate InSpec scans by scheduling them with cron.

#### A. **Edit the Crontab**

```bash
crontab -e
```

#### B. **Add a Cron Job for Regular Scans**

For example, to run the scan every day at midnight and generate an HTML report:

```bash
0 0 * * * /usr/bin/inspec exec /path/to/linux-baseline --reporter html:/path/to/report-$(date +\%Y-\%m-\%d).html
```

/usr/bin/inspec exec /root/linux-baseline --reporter html:/root/linux-baseline/report-$(date +\%Y-\%m-\%d).html

This cron job will create a new report file each day, named with the current date.

#### Steps to Apply and Check CIS Benchmarks Using OpenSCAP (Alternative to Inspec)

1. **Install OpenSCAP**:
sudo apt-get install openscap-scanner openscap-utils

### Check OpenSCAP version

```
oscap -V
```

### Install SCAP security guide

```
sudo apt install ssg-debderived ssg-base
```

The SSG policies we just installed are located in the /usr/share/xml/scap/ssg/content directory.

# Download the latest Scap Security Guide from

```
sudo wget https://github.com/ComplianceAsCode/content/releases/download/v0.1.74/scap-security-guide-0.1.74.zip
```

### Install unzip if not already pre-installed

```
sudo apt install unzip
```

### Unzip Scap Security Guide

```
sudo unzip scap-security-guide-0.1.74.zip
cd scap-security-guide-0.1.74
```

You should now see the SSG security policies for various linux flavors, but we will work with ssg-ubuntu2204-xccdf.xml in our scenario.

# Display a list of available Profiles in the selected security policy

```
oscap info /usr/share/xml/scap/ssg/content/ssg-ubuntu2204-xccdf.xml
```

<img width="516" alt="image" src="https://github.com/user-attachments/assets/517b01e9-a43f-4d98-8d6b-0f9b48bd8c80">

Lets work with `xccdf_org.ssgproject.content_profile_cis_level1_server` which is tailored for CIS benchmarks Level 1 server hardening.

### Run the evaluation scan that will output the report in html format

```
sudo oscap xccdf eval --profile xccdf_org.ssgproject.content_profile_cis_level1_server --report report.html /usr/share/xml/scap/ssg/content/ssg-ubuntu2204-xccdf.xml
```

After the evaluation has completed, you will see a long list of rules and whether your system passed or failed those rules. Copy the `result.html` file to your local machine and open with web browser.

By following these steps, you'll be able to use InSpec to run CIS benchmark checks against your Ubuntu system, generate reports, and even automate regular compliance checks.
By integrating CIS Benchmarks and tools like OpenSCAP into your security practices, you can significantly enhance your system's security posture, ensure compliance with industry standards, and automate the process of securing your IT infrastructure.

## Next, let us put evrything we have previously done into automation. On your chosen primary server.

Here is a bash script containing the steps to install and configure Ansible on an Ubuntu system.
I have 1 primary server and 2 other hosts, and ansible will be configured with ssh keys to authenticate to the host servers.
Run this script on only the primary server where you want ansible to be installed.

```
sudo nano install_ansible.sh
```

```bash
#!/bin/bash

# Install Ansible
echo "Installing Ansible..."
sudo apt install ansible -y

# Verify Ansible installation
echo "Verifying Ansible installation..."
ansible --version

# Create and configure the inventory file
echo "Configuring Ansible inventory file..."
sudo mkdir /etc/ansible/
sudo touch /etc/ansible/hosts
sudo tee /etc/ansible/hosts > /dev/null <<EOL
[ServerA]
192.168.x.x

[ServerB]
192.168.x.x

[ServerC]
192.168.x.x
EOL

# Edit the Ansible configuration file
echo "Editing Ansible configuration file..."
sudo tee /etc/ansible/ansible.cfg > /dev/null <<EOL
[defaults]
inventory = /etc/ansible/hosts
remote_user = ubuntu

[privilege_escalation]
become = True
become_method = sudo
become_user = root
ask_sudo_pass = False
become_ask_pass = True
EOL

# Test Ansible configuration
#echo "Testing Ansible configuration..."
#ansible all -m ping
#
echo "Ansible installation and configuration complete!"
```

1. **Save the Script:** name it `install_ansible.sh`.

2. **Make the Script Executable:**

   ```bash
   chmod +x install_ansible.sh
   ```

3. **Run the Script:**

   ```bash
   ./install_ansible.sh
   ```

### Then we run the ansbile script to configure ansible hosts and authentication keys

```
# !/bin/bash
# Define variables

SSH_KEY_PATH="$HOME/.ssh/id_rsa"
ANSIBLE_HOSTS="/etc/ansible/hosts"
REMOTE_USER="ubuntu"

# Generate SSH key pair if it doesn't exist

if [ ! -f "$SSH_KEY_PATH" ]; then
  echo "Generating SSH key pair..."
  ssh-keygen -t rsa -b 4096 -f "$SSH_KEY_PATH" -N "" -C "$REMOTE_USER"
else
  echo "SSH key pair already exists at $SSH_KEY_PATH."
fi

# Extract host IPs from Ansible inventory file

HOSTS=$(grep -E '^\[Server[A-C]\]' "$ANSIBLE_HOSTS" -A 10 | grep -v '^\[' | awk '{print $1}')

# Copy the SSH key to the target hosts

for HOST in $HOSTS; do
  echo "Copying SSH key to $REMOTE_USER@$HOST..."
  
  ssh-copy-id -i "$SSH_KEY_PATH.pub" "$REMOTE_USER@$HOST"

  if [ $? -eq 0 ]; then
    echo "Successfully copied SSH key to $HOST."
  else
    echo "Failed to copy SSH key to $HOST."
  fi
done

# Test SSH access

for HOST in $HOSTS; do
  echo "Testing SSH access to $REMOTE_USER@$HOST..."
  
  ssh -o PasswordAuthentication=no "$REMOTE_USER@$HOST" "echo 'SSH to $HOST successful.'"

  if [ $? -eq 0 ]; then
    echo "SSH access to $HOST confirmed."
  else
    echo "SSH access to $HOST failed."
  fi
done

echo "SSH key setup complete."
```

<img width="417" alt="image" src="https://github.com/user-attachments/assets/f2293700-7b6f-45ec-b700-e9a9866337e6">


### Below hardening yaml script includes both package installation and configuration, you may choose to separate installation and configuration commands into separate tasks, or separate yaml files  

#### run `hardening_script.yml` to harden the rest of the ansible hosts

```yaml
---
- name: Run System Hardening Script
  hosts: all
  become: true
  tasks:

    - name: Ensure the hardening script is present on the target host
      copy:
        dest: /tmp/system_hardening.sh
        content: |
          #!/bin/bash
          echo "Updating the system..."
          apt update
          echo "Disabling root login via SSH..."
          sed -i 's/^PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config
          echo "Enabling logging of sudo commands..."
          echo "Defaults logfile=/var/log/sudo.log" >> /etc/sudoers
          echo "Installing and configuring UFW..."
          apt install -y ufw
          ufw default deny incoming
          ufw default allow outgoing
          ufw allow OpenSSH
          ufw enable
          echo "Disabling unused filesystems..."
          echo "install cramfs /bin/true" >> /etc/modprobe.d/disable-filesystems.conf
          echo "install freevxfs /bin/true" >> /etc/modprobe.d/disable-filesystems.conf
          echo "install jffs2 /bin/true" >> /etc/modprobe.d/disable-filesystems.conf
          echo "install hfs /bin/true" >> /etc/modprobe.d/disable-filesystems.conf
          echo "install hfsplus /bin/true" >> /etc/modprobe.d/disable-filesystems.conf
          echo "install udf /bin/true" >> /etc/modprobe.d/disable-filesystems.conf
          echo "Setting strong password policies..."
          apt install -y libpam-pwquality
          sed -i 's/^# minlen.*/minlen = 12/' /etc/security/pwquality.conf
          sed -i 's/^# dcredit.*/dcredit = -1/' /etc/security/pwquality.conf
          sed -i 's/^# ucredit.*/ucredit = -1/' /etc/security/pwquality.conf
          sed -i 's/^# lcredit.*/lcredit = -1/' /etc/security/pwquality.conf
          sed -i 's/^# ocredit.*/ocredit = -1/' /etc/security/pwquality.conf
          echo "Disabling IPv6..."
          sed -i 's/^#net.ipv6.conf.all.disable_ipv6 = 1/net.ipv6.conf.all.disable_ipv6 = 1/' /etc/sysctl.conf
          sysctl -p
          echo "Securing shared memory..."
          echo "tmpfs /run/shm tmpfs defaults,noexec,nosuid 0 0" >> /etc/fstab
          echo "Removing unnecessary packages..."
          apt-get autoremove -y --purge
          echo "Installing and configuring rkhunter..."
          apt install -y rkhunter
          rkhunter --update
          rkhunter --propupd
          rkhunter --check --sk
          echo "Setting up log file permissions..."
          chmod -R go-rwx /var/log/*
          echo "Configuring login banner..."
          echo "Authorized access only. All activity may be monitored and reported." > /etc/issue.net
          sed -i 's/^#Banner.*/Banner \/etc\/issue.net/' /etc/ssh/sshd_config
          echo "Setting up limits on user processes..."
          echo "* hard nproc 100" >> /etc/security/limits.conf
          echo "Restricting cron jobs to authorized users..."
          touch /etc/cron.allow
          chmod 600 /etc/cron.allow
          echo "Restricting access to su command..."
          apt install -y libpam-modules
          if ! grep -q "auth required pam_wheel.so use_uid" /etc/pam.d/su; then
              echo "auth required pam_wheel.so use_uid" >> /etc/pam.d/su
          fi
          if ! getent group wheel > /dev/null; then
              echo "Creating 'wheel' group..."
              groupadd wheel
          fi
          echo "Adding 'ubuntu' user to 'wheel' group..."
          usermod -aG wheel ubuntu
          echo "Disabling core dumps..."
          echo "* hard core 0" >> /etc/security/limits.conf
          echo "fs.suid_dumpable = 0" >> /etc/sysctl.conf
          sysctl -p
          echo "Enabling ASLR..."
          echo "kernel.randomize_va_space = 2" >> /etc/sysctl.conf
          sysctl -p
          echo "Restarting SSH service..."
          systemctl restart ssh
          echo "System hardening completed."
        mode: '0755'

    - name: Run the hardening script
      command: /tmp/system_hardening.sh

```

### Step 2: Deploy and Execute the Playbook

1. **Save the Playbook**:

   Save the playbook as `run_hardening_script.yml` in your Ansible project directory.

2. **Run the Playbook**:

   Execute the playbook using the `ansible-playbook` command:

   ```bash
   ansible-playbook run_hardening_script.yml -vv
   ```

   The `-K` option is used to prompt for the sudo password if required. However, since I have configured Ansible for passwordless sudo, this should not prompt for a password if set correctly.

### To create an Ansible playbook that will install and configure AIDE and Postfix

```
nano install_aide_postfix.yml
```

```yaml
---
- name: Install and Configure AIDE and Postfix
  hosts: all
  become: true
  tasks:

    - name: Ensure AIDE and Postfix installation and configuration script is present on the target host
      copy:
        dest: /tmp/install_aide_postfix.sh
        content: |
          #!/bin/bash

          # Install AIDE
          echo "Installing AIDE..."
          apt-get install -y aide

          # Backup the existing AIDE configuration file
          echo "Backing up AIDE configuration..."
          cp /etc/aide/aide.conf /etc/aide/aide.conf.bkup

          # Add exclusions to AIDE configuration file
          echo "Configuring AIDE exclusions..."
          echo "!/tmp" | tee -a /etc/aide/aide.conf > /dev/null
          echo "!/var/spool" | tee -a /etc/aide/aide.conf > /dev/null

          # Initialize AIDE database
          echo "Initializing AIDE database. This may take a while..."
          aide -c /etc/aide/aide.conf --init

          # Move the new AIDE database to the correct location
          echo "Moving AIDE database..."
          mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db

          # Run an AIDE check against the initialized database
          echo "Running AIDE check. This may take a while..."
          aide -c /etc/aide/aide.conf --check

          # Install Postfix
          echo "Installing Postfix..."
          apt-get install -y postfix

          # Configure Postfix for Gmail relay
          echo "Configuring Postfix..."
          sed -i '/^relayhost =/d' /etc/postfix/main.cf
          echo "relayhost = [smtp.gmail.com]:587" | tee -a /etc/postfix/main.cf > /dev/null
          echo "smtp_use_tls = yes" | tee -a /etc/postfix/main.cf > /dev/null
          echo "smtp_sasl_auth_enable = yes" | tee -a /etc/postfix/main.cf > /dev/null
          echo "smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd" | tee -a /etc/postfix/main.cf > /dev/null
          echo "smtp_sasl_security_options = noanonymous" | tee -a /etc/postfix/main.cf > /dev/null

          # Create sasl_passwd file for Postfix
          echo "Creating SASL password file..."
          touch /etc/postfix/sasl_passwd
          echo "[smtp.gmail.com]:587 balogsun@gmail.com:hbstkssqemmjfqyi" | tee -a /etc/postfix/sasl_passwd > /dev/null

          # Secure the SASL password file and update Postfix lookup table
          echo "Securing SASL password file..."
          chmod 600 /etc/postfix/sasl_passwd
          postmap /etc/postfix/sasl_passwd

          # Allow ports 25 and 587 on firewall
          echo "Allowing ports 25 and 587 on firewall..."
          ufw allow out 25
          ufw allow out 587

          # Restart Postfix to apply changes
          echo "Restarting Postfix..."
          systemctl restart postfix

          # Test the Postfix configuration
          echo "Testing Postfix configuration by sending a test email..."
          echo "If you get this mail, it means the mail application is working" | mail -s "Postfix mail from ServerA@seunb.com" balogsun@gmail.com

          # Print mail logs
          echo "Printing mail logs..."
          tail /var/log/mail.log

          # Set up a cron job for daily AIDE checks and email reports
          echo "Setting up daily AIDE checks cron job..."
          (crontab -l 2>/dev/null; echo "0 2 * * * /usr/bin/aide -c /etc/aide/aide.conf --check | mail -s 'AIDE Daily Report' balogsun@gmail.com") | crontab -

          echo "Setup complete. Check your email for the test message and AIDE reports."
      mode: '0755'

    - name: Run the AIDE and Postfix installation script
      command: /tmp/install_aide_postfix.sh
```

 **Save and run the Playbook**:

   ```bash
   ansible-playbook install_aide_postfix.yml -K
   ```

### Below is an Ansible playbook that will apply specific security configurations based on CIS Benchmarks

```
nano secure_ubuntu.yml
```

```
---

- name: Apply Security Configurations for Ubuntu Server
  hosts: all
  become: yes

  vars:
    sensitive_files:
      - /etc/passwd
      - /etc/shadow
      - /etc/ssh/sshd_config
      - /etc/gshadow
    unnecessary_services:
      - cups
      - avahi-daemon
      - nfs-server
      - rpcbind
      - vsftpd
    ntp_servers:
      - time.google.com
      - time.cloudflare.com
    secure_delete_tool: "secure-delete"
    files_to_secure_delete:
      - /tmp/abc.log  # Add the path to the file you want to securely delete
  
  tasks:

  - name: Restrict permissions on sensitive files
      file:
        path: "{{ item }}"
        owner: root
        group: root
        mode: 0600
      loop: "{{ sensitive_files }}"
      tags: restrict_permissions

  - name: Disable IPv6 (if not needed)
      sysctl:
        name: net.ipv6.conf.all.disable_ipv6
        value: '1'
        sysctl_set: yes
        state: present
        reload: yes
      tags: disable_ipv6

  - name: Disable uncommon network protocols
      lineinfile:
        path: /etc/modprobe.d/blacklist.conf
        line: "blacklist {{ item }}"
        create: yes
      loop:
    - dccp
    - sctp
    - rds
    - tipc
      tags: disable_protocols

  - name: Stop and disable unnecessary services
      service:
        name: "{{ item }}"
        enabled: no
        state: stopped
      loop: "{{ unnecessary_services }}"
      tags: disable_services

  - name: Install and configure chrony for NTP synchronization
      apt:
        name: chrony
        state: present
      tags: configure_ntp

  - name: Configure chrony with NTP servers
      lineinfile:
        path: /etc/chrony/chrony.conf
        line: "server {{ item }} iburst"
        create: yes
        state: present
      loop: "{{ ntp_servers }}"
      notify:
    - restart chrony
      tags: configure_ntp

  - name: Install secure deletion tool
      apt:
        name: "{{ secure_delete_tool }}"
        state: present
      tags: secure_delete

  - name: Securely delete files
      command: "srm -v {{ item }}"
      loop: "{{ files_to_secure_delete }}"
      when: files_to_secure_delete | length > 0
      tags: secure_delete

  handlers:
  - name: restart chrony
      service:
        name: chrony
        state: restarted

```

1. **Playbook Structure**:
   - The playbook is organized into tasks, each of which performs a specific security function.
   - `vars` are used to define variables for sensitive files, unnecessary services, NTP servers, and files to be securely deleted.

2. **Save and run the Playbook**:

     ```bash
     ansible-playbook secure_ubuntu.yml
     ```
  
3. **Customize**:
   - Modify the `sensitive_files`, `unnecessary_services`, and `ntp_servers` lists according to your environment's requirements.
   - Define the `files_to_secure_delete` list with the paths of files you wish to securely delete.

## Scaling and Monitoring

### Let's set up an Ansible playbook to deploy secure web servers on 2 additional identical servers

#### Playbook to Installs and configures my custom (pizza booking) web server

```

nano apache.yml

```

```yaml
---
- name: Install and Configure Apache Web Server
  hosts: all
  become: yes

  tasks:
    - name: Install Apache
      apt:
        name: apache2
        state: present
      tags: install_apache

    - name: Ensure required Apache modules are enabled
      apache2_module:
        name: "{{ item }}"
        state: present
      loop:
        - ssl
        - rewrite
        - headers
        - expires
      tags: apache_modules

    - name: Configure firewall to allow HTTP and HTTPS traffic
      ufw:
        rule: allow
        name: 'Apache Full'
      tags: configure_firewall

    - name: Remove the default site configuration
      command: a2dissite 000-default.conf
      notify:
        - reload apache
      tags: remove_default_site

    - name: Create a custom HTML file for the site
      copy:
        content: |
          <!DOCTYPE html>
          <html lang="en">
          <head>
              <meta charset="UTF-8">
              <meta http-equiv="X-UA-Compatible" content="IE=edge">
              <meta name="viewport" content="width=device-width, initial-scale=1.0">
              <title>Order Your Pizza</title>
              <style>
                  body { background-color: #f4f4f9; font-family: Arial, sans-serif; margin: 0; padding: 0; color: #333; text-align: center; }
                  header { background-color: #808080; color: white; padding: 20px; }
                  h1 { margin: 0; font-size: 2.5em; }
                  h2 { font-size: 1.5em; margin-top: 0.5em; color: #555; }
                  h3 { font-size: 1.2em; margin-top: 1em; color: #333; }
                  .container { max-width: 800px; margin: 20px auto; padding: 20px; background-color: white; border-radius: 8px; box-shadow: 0 0 10px rgba(0, 0, 0, 0.1); }
                  .form-group { margin-bottom: 15px; text-align: left; }
                  .form-group input { width: 100%; padding: 10px; border: 1px solid #ddd; border-radius: 4px; }
                  .toppings { display: flex; justify-content: center; flex-wrap: wrap; }
                  .toppings div { margin: 10px; font-size: 1.1em; }
                  .button { background-color: #4CAF50; border: none; color: white; padding: 15px 25px; text-align: center; text-decoration: none; display: inline-block; font-size: 16px; margin: 10px; border-radius: 5px; cursor: pointer; transition: background-color 0.3s ease; }
                  .button:hover { background-color: #45a049; }
                  .button.reset { background-color: #e7e7e7; color: black; }
                  .button.reset:hover { background-color: #d5d5d5; }
                  img { max-width: 100%; height: auto; margin-top: 20px; border-radius: 8px; }
                  footer { margin-top: 20px; font-size: 14px; color: #777; }
              </style>
          </head>
          <body>
              <header>
                  <h1>Your One Stop Shop for Great Pizzas</h1>
              </header>
              <div class="container">
                  <h2>Kindly Make Your Order by Filling the Details Below</h2>
                  <form>
                      <div class="form-group">
                          <label for="name">Name:</label>
                          <input type="text" id="name" placeholder="Your Name" required>
                      </div>
                      <div class="form-group">
                          <label for="email">Email:</label>
                          <input type="email" id="email" placeholder="Your Email" required>
                      </div>
                      <div class="form-group">
                          <label for="phone">Phone No:</label>
                          <input type="tel" id="phone" placeholder="Your Phone Number" required>
                      </div>
                      <h3>Select Pizza Toppings</h3>
                      <div class="toppings">
                          <div><input type="checkbox" id="chicken"><label for="chicken"> Chicken</label></div>
                          <div><input type="checkbox" id="pepperoni"><label for="pepperoni"> Pepperoni</label></div>
                          <div><input type="checkbox" id="sausage"><label for="sausage"> Sausage</label></div>
                          <div><input type="checkbox" id="mushrooms"><label for="mushrooms"> Mushrooms</label></div>
                      </div>
                      <button type="submit" class="button">Submit</button>
                      <button type="reset" class="button reset">Reset</button>
                  </form>
              </div>
              <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/9/91/Pizza-3007395.jpg/1280px-Pizza-3007395.jpg" alt="Pizza">
              <footer>
                  <h4>All Rights Reserved</h4>
              </footer>
          </body>
          </html>
        dest: /var/www/html/index.html
        mode: '0644'
      tags: configure_website

    - name: Enable the custom site configuration
      copy:
        content: |
          <VirtualHost *:80>
              DocumentRoot /var/www/html
              <Directory /var/www/html>
                  Options Indexes FollowSymLinks
                  AllowOverride All
                  Require all granted
              </Directory>
              ErrorLog ${APACHE_LOG_DIR}/error.log
              CustomLog ${APACHE_LOG_DIR}/access.log combined
          </VirtualHost>
        dest: /etc/apache2/sites-available/custom-site.conf
        mode: '0644'
      notify:
        - reload apache
      tags: configure_custom_site

    - name: Enable the custom site configuration
      command: a2ensite custom-site.conf
      notify:
        - reload apache
      tags: enable_custom_site

    - name: Restart Apache to apply any configuration changes
      service:
        name: apache2
        state: restarted
      tags: restart_apache

    - name: Display Apache status
      command: systemctl status apache2
      register: apache_status
      changed_when: false
      tags: apache_status

  handlers:
    - name: reload apache
      service:
        name: apache2
        state: reloaded
```

 **Run the Playbook**:

   Execute the playbook using the `ansible-playbook` command:

   ```bash
   ansible-playbook apache.yml -K
   ```

- You notice a failure on one of the servers, all I had to do was integrate the `apt-get update` command into the ansible script to make it successful.

<img width="520" alt="image" src="https://github.com/user-attachments/assets/3dc7ee30-8e77-4deb-9150-ac6cb5ad1a32">


<img width="490" alt="image" src="https://github.com/user-attachments/assets/3f57a021-e45a-463a-b3c7-3e9ed569cd85">


<img width="343" alt="image" src="https://github.com/user-attachments/assets/cee431bf-b5a7-4124-ac7a-c9412143d35d">


### Monitoring with ELK stack

- To implement a central logging solution using the ELK stack (Elasticsearch, Logstash, and Kibana) and create a dashboard or report to overview the security status of all servers, follow these detailed steps. This guide will cover setting up the ELK stack on a central logging server and configuring other servers to send their logs.

### **1. Central Logging Server Setup (192.168.5.20)**

#### **1.1 Install Java Environment Packages**

Elasticsearch and logstash requires Java to run. Ensure its comes pre-installed once they are installed.
Keep below commands and run it to check java version after ELK installation.

```
/usr/share/elasticsearch/jdk/bin/java -version
/usr/share/logstash/jdk/bin/java -version
```

#### **1.2 Install and Configure Elasticsearch**

```bash
sudo apt-get install apt-transport-https
```

- Import the Elasticsearch PGP Key

```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
```

Installing from the APT repository:

```
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
```

```
sudo apt-get update && sudo apt-get install elasticsearch
```

### Configure Elasticsearch

Edit the `/etc/elasticsearch/elasticsearch.yml` file

```
cluster.name: my-application
node.name: node-1
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
network.host: 192.168.5.20
http.port: 9200
discovery.seed_hosts: ["192.168.5.20"]
cluster.initial_master_nodes: ["node-1"]
xpack.security.enabled: false
```

# [optional] You can Configure JVM heap memory or leave as it is for a dev environment

-Xms512m
-Xmx512m

# Start and enable Elasticsearch

```
sudo systemctl start elasticsearch
sudo systemctl enable elasticsearch

```

#### Check elasticsearch logs

```
tail /var/log/elasticsearch/my-application.log
```

#### **1.3 Install and Configure Logstash**

```bash
# Install Logstash
sudo apt install -y logstash

# Create Logstash configuration file for Filebeat input
sudo tee /etc/logstash/conf.d/filebeat.conf > /dev/null <<EOF
input {
  beats {
    port => 5044
  }
}

output {
  elasticsearch {
    hosts => ["http://192.168.5.20:9200"]
    index => "security-logs-%{+YYYY.MM.dd}"
  }
}
EOF
```

### Start and enable Logstash

```
sudo systemctl start logstash
sudo systemctl enable logstash
sudo systemctl status logstash

```

#### **1.4 Install and Configure Kibana**


#### Install Kibana

```
sudo apt install -y kibana
sudo apt-get update && sudo apt-get install kibana
```

#### Configure Kibana, make modifications to `/etc/kibana/kibana.yml` file

#### Uncomment and set `server.port`

```
sudo sed -i 's/#server.port: 5601/server.port: 5601/' /etc/kibana/kibana.yml
```

#### Uncomment and modify `elasticsearch.hosts`

```
sudo sed -i 's/#elasticsearch.hosts: \["http:\/\/localhost:9200"\]/elasticsearch.hosts: \["http:\/\/192.168.5.20:9200"\]/' /etc/kibana/kibana.yml
```

#### Uncomment and modify `server.host`

```
sudo sed -i 's/#server.host: "localhost"/server.host: "192.168.5.20"/' /etc/kibana/kibana.yml
```

```
server.name: "ServerA"
server.publicBaseUrl: "http://192.168.5.20:5601"
```

#### Start and enable Kibana

```
sudo systemctl start kibana
sudo systemctl enable kibana
sudo systemctl status kibana
```

#### **Configure UFW on Central Logging Server**

```bash
# Allow necessary ports
sudo ufw allow 9200/tcp   # Elasticsearch
sudo ufw allow 5601/tcp   # Kibana
sudo ufw allow 5044/tcp   # Logstash
sudo ufw enable
```

### Install and Configure Filebeat on All Servers that will feed logs to kibana**

```bash
# Install Filebeat
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list

sudo apt update
sudo apt install -y filebeat
```

Edit the Filebeat configuration file `/etc/filebeat/filebeat.yml` on each server:

- setting up filebeat to ship logs to elastic search,

```  
output.elasticsearch:
  hosts: ["192.168.5.20:9200"]
  username: "kibana_system"
  password: "password"
  
setup.kibana:
    host: "192.168.5.20:5601"
    username: "kibana_system"  
    password: "password"
```

#### But if setting up filebeat to ship logs through logstash, Uncomment and modify the `output.logstash` section:**, `output.logstash` and `output.elasticsearch` cannot be enabled at same time.

```
# output.logstash
# The Logstash hosts
  #hosts: ["192.168.5.20:5044"]
```

#### List the available modules:

```
cd /usr/share/filebeat/bin
filebeat modules list
```

I have installed apache previously so we’re going to configure Filebeat to ship apache logs, system logs and authentication logs.

```
cd /etc/filebeat/modules.d
filebeat modules enable apache
```

```
nano /etc/filebeat/modules.d/apache.yml
```

```
- module: apache

# Access logs

  access:
    enabled: true
    var.paths:
      - /var/log/apache2/access.log*

# Error logs

  error:
    enabled: true
    var.paths:
      - /var/log/apache2/error.log*
```

```
filebeat modules enable system
```
```
nano /etc/filebeat/modules.d/system.yml
```

```
- module: system

# Syslog

  syslog:
    enabled: true
    var.paths:
      - /var/log/syslog*
  
# Authorization logs

  auth:
    enabled: true
    var.paths:
      - /var/log/auth.log*
```

Below can be used if it were nginx server that was installed.

```
cd /usr/share/filebeat/bin
filebeat modules enable nginx
```

```
nano /etc/filebeat/modules.d/nginx.yml
```

```
- module: nginx
  access:
    enabled: true
    var.paths: ["/var/log/nginx/access.log*"]
```

Test that the configuration is okay

```
$ sudo filebeat test config
sudo filebeat test config -e
Config OK
```

#### **2.3 Start and Enable Filebeat on All Servers**

```bash
# Start and enable Filebeat
sudo systemctl start filebeat
sudo systemctl enable filebeat
sudo systemctl status filebeat
```

#### **2.4 Adjust Firewall on All Servers if enabled**

- Ensure UFW allows traffic to and from the Central Logging Server:

```bash
# Allow traffic to the Central Logging Server
sudo ufw allow 5044/tcp   # Logstash
sudo ufw enable
```

- Test for connectivity amongst all filebeats host servers

```
curl -X GET "192.168.5.20:9200/"
curl -I <http://192.168.5.20:5601> [firewall]
```

### **Create Dashboards and Reports in Kibana** [Run this on central logging server only]

```
cd /usr/share/filebeat/bin
sudo filebeat setup --dashboards
```

- Enable default dashboard

```
sudo filebeat setup -e
```

#### **4.1 Access Kibana**

- Open your web browser and navigate to `http://192.168.5.20:5601`.

- In the side navigation, click Discover. To see Filebeat data, make sure the predefined filebeat-* data view is selected.

- If you don’t see data in Kibana, try changing the time filter to a larger range. By default, Kibana shows the last 15 minutes.

- In the side navigation, click Dashboard, then select the dashboard that you want to open.

  **syslog dashboard**
  ![Screenshot 2024-08-14 125932](https://github.com/user-attachments/assets/69b79df6-52b4-413a-97cf-8a795d502649)

  **ssh login dashboard**
  ![Screenshot 2024-08-14 130933](https://github.com/user-attachments/assets/25293241-aa0c-4adb-bf9c-57f5091bbb62)

  **sudo commands dashboard**
  ![Screenshot 2024-08-14 131206](https://github.com/user-attachments/assets/34060b0a-c89c-458f-96b1-8b89ebc56636)


 - Optionally, Configure your visualization to display the status data you are interested in (e.g., number of failed login attempts), 
 select a pre-set dashboard and Add visualizations, Click **Save**.

  In wrapping up, the journey of securing and automating cloud servers is a critical aspect of modern DevOps practices. Through meticulous server hardening, continuous monitoring, and strategic use of automation tools like Ansible, we've crafted a resilient and scalable infrastructure. This process not only fortifies your servers against vulnerabilities but also enhances efficiency and consistency across deployments. By adopting these best practices, your infrastructure would be positioned to maintain a secure, reliable, and scalable environment, enabling you to confidently support your company's growth and technological advancements.
