# From Manual to Automated: Optimizing Cloud Infrastructure with Terraform, Load Balancing, and Auto-Scaling

<img width="352" alt="image" src="https://github.com/user-attachments/assets/8ade1741-a537-4bfb-8c1b-47182dc49a73">

Let's dive into how **Terraform**, **load balancing**, and **auto-scaling** can make your cloud setup more efficient and reliable.

**Terraform** is a tool that lets you manage your cloud resources using code. This means you can set up and control your infrastructure in a consistent and automated way, making it easy to duplicate, update, and maintain.

**Load balancing** helps spread incoming traffic across several servers so no single server gets overwhelmed. This improves your app’s performance and keeps it available even if one server has issues.

**Auto-scaling** automatically adjusts the number of resources you’re using based on traffic. It scales up during busy times and scales down when things are quieter, which helps with both performance and cost.

Let's see how these technologies work together to build robust and flexible cloud environments.

I once had the opportunity to tackle a challenging yet rewarding task, transforming a client's manually-provisioned AWS infrastructure into a more agile and cost-efficient setup. The client’s web application was struggling to handle traffic spikes due to marketing campaigns and seasonal sales, which led to potential downtime and sometimes over-provisioning costs.

To tackle this, I used a two-step approach. First, I moved their setup to Terraform, which allowed us to automate and streamline the process. Then, I added auto-scaling to help their system adjust automatically to different traffic levels, making it more efficient and cost-effective.

Join me as I dive into the hands-on details of how these changes made the client’s system more agile and effective.

### Prerequisites

- An Ubuntu/Linux Environment OR A Windows desktop Environment
- Github account
- An EC2 instance running Apache web service

#### Install AWS CLI

The AWS CLI is a unified tool that provides a consistent interface for interacting with Amazon Web Services. It’s essential for managing your AWS services from the command line. To install AWS CLI on Ubuntu, follow these steps:

```bash
sudo apt update
sudo curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

Alternatively, you can follow the official AWS documentation for more detailed instructions: [AWS CLI Installation Guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html).

### **Create AWS IAM User and Configure AWS CLI**

#### **Sign in to AWS**

- Go to the [AWS Console](https://aws.amazon.com/console/) and log in.

#### **Navigate to IAM**

- Search for **IAM** in the AWS Console and select it.

#### **Create a New IAM User**

- Click **Users** > **Add users**.
- Enter a user name (e.g., `cli-user`).

#### **Set Permissions**

- Attach necessary permissions:
  - **Attach existing policies**: Select `AdministratorAccess` or another policy.
  - Or **Add user to group**.

#### **Review and Create User**

- Review details and click **Create user**.

#### **Download AWS Credentials**

- Click **Create access key** in the `Access Key` section.
- Download the `.csv` file with the credentials.

#### **Configure AWS CLI**

- Open a terminal on your machine.
- Run:

```bash
aws configure
```

- Enter the **Access Key ID** and **Secret Access Key** from the `.csv` file:

```bash
AWS Access Key ID [None]: YOUR_ACCESS_KEY_ID
AWS Secret Access Key [None]: YOUR_SECRET_ACCESS_KEY
Default region name [None]: us-east-1
Default output format [None]: json
```

#### **Verify Configuration**

- Run:

```bash
aws sts get-caller-identity
```

- This should return IAM user information, confirming your CLI setup.

#### Install Terraform

Terraform is an open-source infrastructure as code software tool that provides a consistent CLI workflow to manage hundreds of cloud services. Here’s how you can install Terraform on Ubuntu:

```bash
sudo apt-get update && sudo apt-get install -y gnupg software-properties-common
sudo apt update && sudo apt install gpg
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
gpg --no-default-keyring --keyring /usr/share/keyrings/hashicorp-archive-keyring.gpg --fingerprint
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update
sudo apt install terraform
terraform version
```

For more details or alternative installation methods, visit the official Terraform documentation: [Terraform Installation Guide](https://developer.hashicorp.com/terraform/install).

### Step 2: Initialize Terraform Project

1. **Create Terraform Project Directory:**

```bash
mkdir terraform-project
cd terraform-project
```

2. **Initialize Terraform:**

- Create a `main.tf` file in the directory, place in the cloud provider details and initialize Terraform:

```bash
nano main.tf
```

```hcl
provider "aws" {
  region = "ca-central-1"
```

```bash
terraform init
```

### Step 3: Import all Existing AWS Resources you can find in the application infrastructure

1. **Identify AWS Resources:**

- Determine the AWS resources to import, such as EC2 instances, security groups, VPCs etc.

```bash
# List all EC2 Instances and their Instance IDs
aws ec2 describe-instances --query "Reservations[*].Instances[*].{InstanceID:InstanceId,Name:Tags[?Key=='Name']|[0].Value}" --output table
```

This shows you the instance IDs, then run again by querying each IDs:

```bash
aws ec2 describe-instances --instance-ids i-0340a349e9709dd41 --output json
aws ec2 describe-instances --instance-ids i-0340a349e9709dd41 --query "Reservations[*].Instances[*].{InstanceID:InstanceId,InstanceType:InstanceType,PublicIP:PublicIpAddress,PrivateIP:PrivateIpAddress,SecurityGroups:SecurityGroups,KeyName:KeyName,SubnetID:SubnetId,VpcID:VpcId,State:State.Name,Tags:Tags}" --output json
```

```bash
# List all Security Groups and their Group IDs
aws ec2 describe-security-groups --query "SecurityGroups[*].{GroupID:GroupId,GroupName:GroupName,VpcID:VpcId}" --output table
```

This shows you the security-groups IDs, then run again by querying each IDs:

```bash
aws ec2 describe-security-groups --group-ids sg-04184f0a2129e6959 --query "SecurityGroups[*].{GroupID:GroupId,GroupName:GroupName,VpcID:VpcId,InboundRules:IpPermissions,OutboundRules:IpPermissionsEgress}" --output json
```

```bash
# List all VPCs and their VPC IDs
aws ec2 describe-vpcs --query "Vpcs[*].{VpcID:VpcId,CidrBlock:CidrBlock,State:State}" --output table
```

This shows you the VPC IDs, then run again by querying each IDs:

```bash
aws ec2 describe-vpcs --vpc-ids vpc-0cb7f49b4c1b06a91 --query "Vpcs[*].{VpcID:VpcId,CidrBlock:CidrBlock,State:State,DhcpOptionsId:DhcpOptionsId,InstanceTenancy:InstanceTenancy,IsDefault:IsDefault,Tags:Tags}" --output json
```

```bash
# List all SSH Key Pairs and their Key Names, keep the details to be used when creating the terraform resource for the ec2 instances.
aws ec2 describe-key-pairs --query "KeyPairs[*].{KeyName:KeyName,KeyFingerprint:KeyFingerprint}" --output table
```

### 2. **Import Resources into Terraform State:**

Now that we have details about the unmanaged resources of the AWS environments, we can now use the `terraform import` command to import each resource into Terraform state:

```bash
terraform import aws_instance.web i-0340a349e9709dd41
terraform import aws_security_group.launch-wizard-1 sg-04184f0a2129e6959
terraform import aws_vpc.main vpc-0cb7f49b4c1b06a91
terraform import aws_ami_from_instance.web_ami ami-0ca616d4f1e4d86d4
```

- Repeat the above process for all relevant AWS resources.

### 3. **Generate Configuration Files:**

Now we will create Terraform configuration `.tf` files for each resource using the information gotten from the `aws ec2 describe` command we recently ran.

#### Create Terraform files similar to the below Structure Overview

```plaintext
terraform-project/...
├── main.tf/ - will contain configurations related to EC2 instances and provider.
├── sg.tf/ - will contain configurations for security groups.
├── ami.tf/ - will contain configurations the AMI we will create as we proceed.
└── vpc.tf/ - will contain configurations for VPCs.
```

#### Updated `main.tf`

This Terraform configuration file will create an EC2 instance named web-svr in the ca-central-1 region. The instance will be based on a specific AMI, with a t3.xlarge instance type. It will be launched in a predefined subnet, with security rules governed by a specified security group, and access will be managed using an existing SSH key pair.

```hcl
provider "aws" {
  region = "ca-central-1"
  resource "aws_instance" "web" {
    ami = "ami-0c6d358ee9e264ff1"
    instance_type = "t3.xlarge"
    key_name = "your-key"
    subnet_id = aws_subnet.subnet_a.id
    vpc_security_group_ids = [aws_security_group.launch_wizard_1.id]
    tags = {
      Name = "web-svr"
    }
  }
}
```

### Create storage group configuration file `sg.tf`

The Terraform configuration file (`sg.tf`) creates a security group named `launch-wizard-1` within a specified VPC. It allows inbound traffic on ports 22 (SSH), 5000-6000, 443 (HTTPS), 8080, and 80 (HTTP) from all IP addresses (`0.0.0.0/0`). SSH access is also allowed from a specific IP (`142.114.25.200/32`). The configuration permits all outbound traffic.

```hcl
resource "aws_security_group" "launch_wizard_1" {
  name        = "launch-wizard-1"
  description = "launch-wizard-1 created"
  vpc_id      = aws_vpc.main1.id
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0", "142.114.25.200/32"]
  }
  ingress {
    from_port   = 5000
    to_port     = 6000
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 8080
    to_port     = 8080
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

### Run the `terraform plan` command.
 
 ```
 terraform plan
 ```
 
If all goes well, a result showing that there is nothing to be created/destroyed would be shown, implying that the state of the existing AWS infrastructure and the Terraform state file is in sync.

### 4. Create an AMI from the running web instance

Create an AMI from an existing EC2 instance that will be used to launch new instances with the autoscaling group.

This Terraform resource creates an Amazon Machine Image (AMI) from an existing EC2 instance. It captures the current state of the instance specified by its ID, generates a uniquely named AMI, and tags it as `web-ami`. The `snapshot_without_reboot` option allows the AMI to be created without rebooting the instance, ensuring no downtime.

```hcl
resource "aws_ami_from_instance" "web_ami" {
  name               = "web-ami-${formatdate("YYYYMMDD-HHmm", timestamp())}"
  source_instance_id = "i-0340a349e9709dd41" # Replace with your EC2 instance ID
  description        = "AMI created from instance i-0340a349e9709dd41"
  snapshot_without_reboot = true
  tags = {
    Name = "web-ami"
  }
}
```

#### Run terraform apply to deploy create the AMI and snapshot, once completed, remove the ami state from the statefile so that the next `terraform apply` command won't create another AMI to overwrite the existing one

```bash
terraform state list
terraform state rm aws_ami_from_instance.web_ami
```

### 5. Creating Auto Scaling Group (ASG) with Auto Scaling Policy and Load Balancer (ALB)

Lets create an Auto Scaling Group (ASG) with an Auto Scaling Policy and a Load Balancer (ALB), and ensure that any new instances launched by the ASG using the newly created AMI are automatically registered with the Load Balancer. When creating an Application Load Balancer (ALB) in AWS, you must specify at least two subnets in different Availability Zones (AZs). AWS requires this for high availability purposes.

The below Terraform configuration sets up a scalable web application infrastructure by:

1. **Creating an Application Load Balancer (ALB)** that distributes traffic across specified subnets and is protected by a security group.
2. **Defining a Target Group** for the ALB to route HTTP traffic and perform health checks on instances.
3. **Configuring a Load Balancer Listener** to forward HTTP requests to the Target Group.
4. **Creating a Launch Template** that specifies instance configuration details using a pre-existing AMI.
5. **Setting up an Auto Scaling Group (ASG)** to manage instance scaling based on demand, with connections to the Load Balancer and Target Group.
6. **Defining Scaling Policies** to automatically adjust the number of instances based on CPU usage.
7. **Configuring CloudWatch Alarms** to trigger scaling actions based on high and low CPU utilization.
8. **When CPU usage exceeds 70%, a high CPU alarm is triggered. After 90 seconds, the auto scale-out policy adds a new EC2 instance to handle the increased load, and when CPU usage drops below 30%, a low CPU alarm is triggered. After a 5-minute wait (300 seconds), the auto scale-in policy removes one instance at a time to adjust to the reduced load.

nano `asg_alb.tf`

```hcl
# Define the Load Balancer
resource "aws_lb" "web_lb" {
  name = "web-load-balancer"
  internal = false
  load_balancer_type = "application"
  security_groups = [aws_security_group.launch_wizard_1.id]
  subnets = [aws_subnet.subnet_a.id, aws_subnet.subnet_b.id]
  enable_deletion_protection = false
  tags = {
    Name = "web-load-balancer"
  }
}

# Define the Target Group for the Load Balancer
resource "aws_lb_target_group" "web_target_group" {
  name = "web-target-group"
  port = 80
  protocol = "HTTP"
  vpc_id = "vpc-0cb7f49b4c1b06a91" # Replace with your VPC ID
  health_check {
    path = "/"
    interval = 30
    timeout = 5
    healthy_threshold = 3
    unhealthy_threshold = 3
  }
  tags = {
    Name = "web-target-group"
  }
}

# Define the Load Balancer Listener
resource "aws_lb_listener" "web_listener" {
  load_balancer_arn = aws_lb.web_lb.arn
  port = 80
  protocol = "HTTP"
  default_action {
    type = "forward"
    target_group_arn = aws_lb_target_group.web_target_group.arn
  }
}

# Directly reference the existing AMI in your launch template
resource "aws_launch_template" "web_template" {
  name_prefix = "web-launch-template-"
  image_id = "ami-0ca616d4f1e4d86d4" # Use the existing AMI ID here
  instance_type = "t2.micro"
  key_name = "project-key"
  block_device_mappings {
    device_name = "/dev/sda1"
    ebs {
      delete_on_termination = true
      volume_size = 8
      volume_type = "gp3"
      iops = 3000
      throughput = 125
    }
  }
  monitoring {
    enabled = false
  }
  network_interfaces {
    security_groups = [aws_security_group.launch_wizard_1.id]
  }
  tag_specifications {
    resource_type = "instance"
    tags = {
      # Tag added for completeness; you can change or remove this if necessary
      Project = "web-application"
    }
  }
}

# Auto Scaling Group
resource "aws_autoscaling_group" "web_asg" {
  launch_template {
    id = aws_launch_template.web_template.id
    version = "$Latest"
  }
  min_size = 1
  max_size = 3
  desired_capacity = 1
  vpc_zone_identifier = [aws_subnet.subnet_a.id, aws_subnet.subnet_b.id] # Replace with your subnet IDs
  target_group_arns = [aws_lb_target_group.web_target_group.arn] # Attach ASG to Target Group
  tag {
    key = "Name"
    value = "web"
    propagate_at_launch = true
  }
  health_check_type = "EC2"
  health_check_grace_period = 300
  lifecycle {
    create_before_destroy = true
  }
}

# Scaling Policies
resource "aws_autoscaling_policy" "scale_out_policy" {
  name = "scale_out"
  scaling_adjustment = 1
  adjustment_type = "ChangeInCapacity"
  cooldown = 90 # Set cooldown to 90 seconds
  autoscaling_group_name = aws_autoscaling_group.web_asg.name
}

resource "aws_autoscaling_policy" "scale_in_policy" {
  name = "scale_in"
  scaling_adjustment = -1
  adjustment_type = "ChangeInCapacity"
  cooldown = 300 # Set cooldown to 5 minutes
  autoscaling_group_name = aws_autoscaling_group.web_asg.name
}

# CloudWatch Alarms
resource "aws_cloudwatch_metric_alarm" "high_cpu_alarm" {
  alarm_name = "high_cpu"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods = 2 # Trigger alarm after 2 min wait period
  metric_name = "CPUUtilization"
  namespace = "AWS/EC2"
  period = 120 # 1 minute period
  statistic = "Average"
  threshold = 70
  alarm_description = "This metric monitors EC2 CPU utilization"
  dimensions = {
    AutoScalingGroupName = aws_autoscaling_group.web_asg.name
  }
  alarm_actions = [aws_autoscaling_policy.scale_out_policy.arn]
}

resource "aws_cloudwatch_metric_alarm" "low_cpu_alarm" {
  alarm_name = "low_cpu"
  comparison_operator = "LessThanThreshold"
  evaluation_periods = 2 # Trigger alarm after 2 evaluation period
  metric_name = "CPUUtilization"
  namespace = "AWS/EC2"
  period = 120 # 3 minute period
  statistic = "Average"
  threshold = 30
  alarm_description = "This metric monitors EC2 CPU utilization"
  dimensions = {
    AutoScalingGroupName = aws_autoscaling_group.web_asg.name
  }
  alarm_actions = [aws_autoscaling_policy.scale_in_policy.arn]
}
```

### Run terraform apply to deploy the new resources

#### 5. **Initialize and Apply**

 ```bash
 terraform init
 terraform plan
 terraform apply
 ```

## Step 6: Version Control

1. **Initialize Git Repository:**
   - Initialize a Git repository in your Terraform project directory:

   ```bash
   git init
   git add .
   git commit -m "Initial commit with Terraform configurations"
   ```

2. **Set Up Remote Repository:**
   - Link your local repository to a remote Git repository:

   ```bash
   git remote add origin
   git push -u origin main
   ```

## Step 7: Remote State Storage

1. **Create an S3 bucket to store your Terraform state files:**

   ```bash
   aws s3api create-bucket --bucket seun-terraform-state --region ca-central-1 --create-bucket-configuration LocationConstraint=ca-central-1
   ```

   response: "Location": "<http://seun-terraform-state.s3.amazonaws.com/>"

2. **Update Terraform Configuration for Remote State:**
   The terraform block should be placed at the top level of your Terraform configuration file, outside of any resource blocks.
   - Update your `main.tf` to use remote state storage:

   ```hcl
   terraform {
     backend "s3" {
       bucket = "web-terraform-state"
       key    = "terraform.tfstate"
       region = "ca-central-1"
     }
   }
   ```

3. **Run Terraform Initialization:**
   Since you've changed the backend configuration, you need to reinitialize Terraform with the new settings. This will set up the remote state storage and migrate any existing state if needed.

   ```sh
   terraform init -reconfigure
   ```

   The `-reconfigure` flag tells Terraform to reconfigure the backend with the new settings, even if it was previously initialized.

   After running the `terraform init -reconfigure`, Terraform will set up the remote state backend as specified in your `main.tf` file. It will also prompt you to confirm that the state file should be copied to the new backend if you have existing state.

4. **Migrate Local State to Remote State:**
   - Run Terraform init below to migrate your state to the remote backend:

   ```bash
   terraform init -migrate-state
   ```

## Step 8: Terraform Workspaces

1. **Create Terraform Workspaces:**
   - You may also set up different workspaces for different environments (e.g., dev, prod), placing a copy of all configuration files in each workspace and naming the resources accordingly.

   ```bash
   terraform workspace new dev
   terraform workspace new prod
   ```

## Step 9: Apply and Test Configuration

1. **Run `terraform plan`:**
   - Check your configurations:

   ```bash
   terraform plan
   ```

2. **Apply Configuration:**
   - Apply the changes to set up the infrastructure:

   ```bash
   terraform apply
   ```

## Step 10: Perform Load Testing

To perform load testing with Apache Benchmark (ab), you should run the benchmark tool from a location where you can generate load against your application. Here's a guide on where and how to run it:

### Where to Run Apache Benchmark

1. **From a Local Machine:**
   - You can run Apache Benchmark from your local computer, especially if it has sufficient network bandwidth and resources. Ensure that your local machine can reach the load balancer DNS name or IP address.

2. **From an EC2 Instance:**
   - For a more controlled environment, you might run the benchmark from an EC2 instance in the same region as your application to reduce latency and network issues. This can also simulate real-world usage more accurately.

3. **From a Dedicated Testing Environment:**
   - For more extensive testing, use a dedicated load testing environment or a cloud-based load testing service to generate significant traffic and measure performance under load.

#### 1. **Install Apache Benchmark:**

- **On Ubuntu/Debian:**

   ```bash
   sudo apt-get update
   sudo apt-get install apache2-utils
   ```

#### 2. **Run Load Test:**

- **Basic Load Test Command:**

   ```bash
   curl http://web-load-balancer-1581165346.ca-central-1.elb.amazonaws.com/index.html
   sudo ab -n 6000000 -c 1000 http://web-load-balancer-1581165346.ca-central-1.elb.amazonaws.com/index.html
   ```

- **Explanation of Parameters:**
  - `-n 6000000`: Total number of requests to perform.
  - `-c 1000`: Number of multiple requests to perform at a time (concurrency level).
  - `http:///index.html`: The URL of the resource you're testing.

- **Monitor Metrics:** While running the load test, monitor the metrics on the EC2 instance to check its performance (e.g., CPU utilization, memory usage).

- **Adjust Parameters:** Depending on your testing requirements, adjust the `-n` and `-c` parameters to simulate different load scenarios.

- **Scale Testing:** For more realistic testing, scale up your load test to simulate higher traffic volumes, especially if you need to test the limits of your Auto Scaling Group and Load Balancer.

   Running the benchmark from a location with good network connectivity to your application will give you the most accurate results.

3. **Monitor Auto-Scaling:**
   - Observe how the Auto-Scaling Group reacts to the increased load.

4. **Examine the Application and its resources:**
   - Ensure the application is running smoothly and the infrastructure scales as expected.

5. **Review Logs and Metrics:**
   - Check the logs and CloudWatch metrics to verify the performance and scaling behavior.

6. **Commit and Push Final Changes:**
   - Ensure all changes are committed and pushed to your version control repository.

In summary, moving from manual to automated cloud setup with tools like Terraform, load balancing, and auto-scaling makes your system more flexible, efficient, and cost-effective. Terraform helps manage everything through code, while load balancing and auto-scaling adjust resources to handle traffic and prevent downtime. This means your application will perform better, be more reliable, and save you money. Using these tools together creates a stronger, smarter cloud environment.
