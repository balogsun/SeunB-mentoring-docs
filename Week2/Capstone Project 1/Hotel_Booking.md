# **End-to-End Deployment of a Microservice Application with Terraform, AWS EKS, GitHub Actions, and Monitoring**

Let's walk through the process of deploying a hotel booking application from start to finish. This guide will take you through building the Dockerfile, using Terraform to set up a cluster on AWS EKS, and creating a secure CI/CD pipeline with GitHub Actions. Additionally, we'll cover setting up monitoring tools like Prometheus and Grafana to ensure the application's operational efficiency and reliability. By the end of this guide, you should have a clear understanding of how these components integrate to build and maintain a robust, scalable application environment.

### **Open the GitHub URL [Hotel-Booking](https://github.com/balogsun/hotel-booking.git) and fork the repository.**

- Follow the instructions to complete cloning the repository to your own github account so that you can work with the copy you have created.

  <img width="706" alt="Screenshot 2024-08-19 205107" src="https://github.com/user-attachments/assets/6b80385a-038c-4f70-9ce8-f726db047693">

[GitHub Repository](https://github.com/balogsun/hotel-booking.git)

**The microservice architecture of this application includes:**

- **User Service**: Manages user registration, authentication, and profiles.
- **Booking Service**: Handles booking processes, availability checks, and reservations.
- **Payment Service**: Manages payment processing and transactions.
- **Notification Service**: Sends booking confirmations, reminders, and notifications.
- **Review Service**: Manages user reviews and ratings for hotels.
- **Hotel Management Service**: Handles hotel listings, room details, and amenities.

However, in this scenario, these services have been bundled together into a single deployment.

Bundling microservices into a single deployment can simplify the process and reduce complexity, particularly in environments with limited resources. This approach lowers operational overhead, makes scaling easier, and can be advantageous when speed of delivery or resource constraints are key factors. While this method can be practical in certain situations, it is typically a temporary solution, with the goal of eventually transitioning to a fully decoupled microservices architecture as infrastructure and capabilities improve.

### Writing the docker file:
In the repo provided above, the code is written with javascript, so the Dockerfile would be written with a Node.js base image, specifying the necessary version, installing dependencies using npm, and setting up the environment to build and run the application.

### The intention is to write a Dockerfile, test the code build, and upon successful build, insert the Dockerfile path into our proposed GitHub workflow

## Dockerfile

```dockerfile
# Use Node.js LTS version as the base image
#FROM node:lts-alpine
FROM node:14

# Set the working directory in the container
WORKDIR /app

# Copy package.json and package-lock.json 
COPY package*.json ./
#COPY package.json package-lock.json /app/

# Install project dependencies
RUN npm install

# Copy the entire project files to the container
COPY . /app/
#COPY . .

# Build the Next.js application for production
RUN npm run build

# Expose the port used by your Next.js app (if needed)
EXPOSE 3000

# Define the default command to start the Next.js app
CMD ["npm", "start"]
#CMD ["node", "server.js"]
```

### Dockerfile Explanation

- **Base Image**: Using Node.js LTS version as the base image ensures stability and compatibility.
- **Working Directory**: Sets the working directory in the container to `/app`.
- **Copying Dependencies**: Copies `package.json` and `package-lock.json` to the container to install dependencies.
- **Installing Dependencies**: Runs `npm install` to install project dependencies.
- **Copying Project Files**: Copies the entire project files to the container.
- **Building the Application**: Runs `npm run build` to build the Next.js application for production.
- **Exposing Port**: Exposes port 3000, which the Next.js app listens on.
- **Starting the Application**: Defines the default command to start the Next.js app using `npm start`.

## Building, Testing, and Pushing Your Docker Image

With your Dockerfile ready, you can now build, test, and push your Docker image to a container repository. 

### Install Docker

To get started with Docker on an Ubuntu system, follow these steps:

#### Set Up Docker's APT Repository

1. **Add Docker's Official GPG Key**:
   
   ```bash
   sudo apt-get update
   sudo apt-get install ca-certificates curl
   sudo install -m 0755 -d /etc/apt/keyrings
   sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
   sudo chmod a+r /etc/apt/keyrings/docker.asc
   ```

2. **Add the Repository to APT Sources**:
   
   ```bash
   echo \
     "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
     $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
     sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   sudo apt-get update
   ```

   **Note**: For Ubuntu derivatives like Linux Mint, you might need to use `UBUNTU_CODENAME` instead of `VERSION_CODENAME`.

#### Install Docker Packages

To install Docker, run:

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Alternatively, follow [Docker’s installation guide](https://docs.docker.com/engine/install/ubuntu/) for more details.

### Build the Docker Image

First, clone the repository containing your Dockerfile:

```bash
git clone https://github.com/balogsun/hotel-booking.git
cd hotel-booking
```

Build your Docker image with:

```bash
docker build -t hotel-app:v1 .
```

This command creates an image named `hotel-app` with the tag `v1` based on your Dockerfile.

### 3. Test the Docker Image

To ensure that your Docker image functions correctly, run it using:

```bash
docker run -p 3000:3000 hotel-app:v1
```

Visit `http://localhost:3000` in your browser to see your application in action.

### 4. Pushing the Image to a Container Repository

To make your Docker image accessible globally, push it to a container repository. Follow these steps for Docker Hub:

#### Create a Docker Hub Account

If you don’t already have a Docker Hub account, [sign up here](https://hub.docker.com/).

#### Log In to Docker Hub

Authenticate with Docker Hub using:

```bash
docker login
```

Enter your Docker Hub credentials when prompted.

#### Tag the Docker Image

Tag your Docker image with your Docker Hub repository:

```bash
docker tag hotel-app:v1 your-dockerhub-username/hotel-app:v1
```

#### Push the Docker Image

Upload the tagged image to Docker Hub:

```bash
docker push your-dockerhub-username/hotel-app:v1
```

Your Docker image is now stored on Docker Hub and can be accessed from anywhere.

### 5. Clean Up Local Images

To free up space on your local machine, delete the Docker image with:

```bash
docker rmi hotel-app:v1
```

## Kubernetes Cluster Setup
Setting up a Kubernetes cluster can be a complex process, but with the right tools and instructions, you can simplify the process significantly. Below is a comprehensive guide to getting your Kubernetes cluster up and running on a Linux environment, specifically Ubuntu.

#### Prerequisites:

- **A Linux Environment (Preferably Ubuntu)**: This guide is tailored for Ubuntu, but similar steps can be adapted for other Linux distributions.

####: Install AWS CLI

The AWS CLI is a unified tool that provides a consistent interface for interacting with Amazon Web Services. It’s essential for managing your AWS services from the command line.

To install AWS CLI on Ubuntu, follow these steps:

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

#### Install kubectl

`kubectl` is the command-line tool for Kubernetes. It allows you to run commands against Kubernetes clusters to deploy applications, manage the cluster, and inspect resources.

To install `kubectl`, execute the following commands:

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --short --client
kubectl version --client
```

For a detailed installation guide, refer to the official Kubernetes documentation: [kubectl Installation Guide](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html).



### Creating an AWS Kubernetes Cluster with Terraform

Follow these steps to set up your AWS Kubernetes cluster using Terraform:

#### **1. Prepare Your Environment**
Create a folder on your Ubuntu machine:

```bash
mkdir eks_cluster
cd eks_cluster
touch eks-cluster.tf eks-worker-nodes.tf outputs.tf providers.tf variables.tf vpc.tf workstation-external-ip.tf
```

#### **2. Define EKS Cluster Resources**
Edit `eks-cluster.tf`:

```bash
nano eks-cluster.tf
```

**Functions:**
- **Create IAM Role:** Defines an IAM role for EKS to manage AWS services.
- **Attach IAM Policies:** Attaches necessary policies for EKS to manage clusters.
- **Create Security Group:** Sets up a security group for cluster communication.
- **Create EKS Cluster:** Defines the EKS cluster and its configuration.

```hcl
# EKS Cluster Resources
resource "aws_iam_role" "my-cluster" {
  name = "my-cluster"
  assume_role_policy = <<POLICY
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "eks.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
POLICY
}

resource "aws_iam_role_policy_attachment" "my-cluster-AmazonEKSClusterPolicy" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSClusterPolicy"
  role       = aws_iam_role.my-cluster.name
}

resource "aws_iam_role_policy_attachment" "my-cluster-AmazonEKSServicePolicy" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSServicePolicy"
  role       = aws_iam_role.my-cluster.name
}

resource "aws_security_group" "my-cluster" {
  name        = "my-cluster-sg"
  description = "Cluster communication with worker nodes"
  vpc_id      = aws_vpc.my-vpc.id
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  tags = { Name = "my-cluster" }
}

resource "aws_security_group_rule" "my-cluster-ingress-workstation-https" {
  cidr_blocks       = [local.workstation-external-cidr]
  description       = "Allow workstation to communicate with the cluster API Server"
  from_port         = 443
  protocol          = "tcp"
  security_group_id = aws_security_group.my-cluster.id
  to_port           = 443
  type              = "ingress"
}

resource "aws_eks_cluster" "my-cluster" {
  name     = var.cluster-name
  role_arn = aws_iam_role.my-cluster.arn
  vpc_config {
    security_group_ids = [aws_security_group.my-cluster.id]
    subnet_ids         = aws_subnet.my-vpc[*].id
  }
  depends_on = [
    aws_iam_role_policy_attachment.my-cluster-AmazonEKSClusterPolicy,
    aws_iam_role_policy_attachment.my-cluster-AmazonEKSServicePolicy,
  ]
}
```

#### **3. Define EKS Worker Nodes**
Edit `eks-worker-nodes.tf`:

```bash
nano eks-worker-nodes.tf
```

**Functions:**
- **Create IAM Role for Nodes:** Defines an IAM role for EKS worker nodes.
- **Define IAM Policies for Autoscaling:** Specifies policies for autoscaling worker nodes.
- **Attach IAM Policies:** Attaches necessary policies for EKS worker nodes.
- **Create Node Group:** Defines the node group configuration for worker nodes.

```hcl
# EKS Worker Nodes Resources
resource "aws_iam_role" "my-cluster-node" {
  name = "my-cluster-node"
  assume_role_policy = <<POLICY
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
POLICY
}

data "aws_iam_policy_document" "worker_autoscaling" {
  statement {
    sid    = "eksWorkerAutoscalingAll"
    effect = "Allow"
    actions = [
      "autoscaling:DescribeAutoScalingGroups",
      "autoscaling:DescribeAutoScalingInstances",
      "autoscaling:DescribeLaunchConfigurations",
      "autoscaling:DescribeTags",
      "ec2:DescribeLaunchTemplateVersions",
    ]
    resources = ["*"]
  }
  statement {
    sid    = "eksWorkerAutoscalingOwn"
    effect = "Allow"
    actions = [
      "autoscaling:SetDesiredCapacity",
      "autoscaling:TerminateInstanceInAutoScalingGroup",
      "autoscaling:UpdateAutoScalingGroup",
    ]
    resources = ["*"]
    condition {
      test     = "StringEquals"
      variable = "autoscaling:ResourceTag/kubernetes.io/cluster/${aws_eks_cluster.my-cluster.id}"
      values   = ["owned"]
    }
    condition {
      test     = "StringEquals"
      variable = "autoscaling:ResourceTag/k8s.io/cluster-autoscaler/enabled"
      values   = ["true"]
    }
  }
}

resource "aws_iam_role_policy_attachment" "workers_autoscaling" {
  policy_arn = aws_iam_policy.worker_autoscaling.arn
  role       = aws_iam_role.my-cluster-node.name
}

resource "aws_iam_policy" "worker_autoscaling" {
  name_prefix = "eks-worker-autoscaling-${aws_eks_cluster.my-cluster.id}"
  description = "EKS worker node autoscaling policy for cluster ${aws_eks_cluster.my-cluster.id}"
  policy      = data.aws_iam_policy_document.worker_autoscaling.json
}

resource "aws_iam_role_policy_attachment" "my-cluster-node-AmazonEKSWorkerNodePolicy" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy"
  role       = aws_iam_role.my-cluster-node.name
}

resource "aws_iam_role_policy_attachment" "my-cluster-node-AmazonEKS_CNI_Policy" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy"
  role       = aws_iam_role.my-cluster-node.name
}

resource "aws_iam_role_policy_attachment" "my-cluster-node-AmazonEC2ContainerRegistryReadOnly" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly"
  role       = aws_iam_role.my-cluster-node.name
}

resource "aws_eks_node_group" "my-cluster-node-group" {
  cluster_name    = aws_eks_cluster.my-cluster.name
  node_group_name = "my-cluster-node-group"
  node_role_arn   = aws_iam_role.my-cluster-node.arn
  subnet_ids      = aws_subnet.my-vpc[*].id
  instance_types  = [var.eks_node_instance_type]
  remote_access {
    ec2_ssh_key = var.key_pair_name
  }
  scaling_config {
    desired_size = 2
    max_size     = 4
    min_size     = 2
  }
  depends_on = [
    aws_iam_role_policy_attachment.workers_autoscaling,
    aws_iam_role_policy_attachment.my-cluster-node-AmazonEKSWorkerNodePolicy,
    aws_iam_role_policy_attachment.my-cluster-node-AmazonEKS_CNI_Policy,
    aws_iam_role_policy_attachment.my-cluster-node-AmazonEC2ContainerRegistryReadOnly,
  ]
}
```

#### **4. Configure Outputs**
Edit `outputs.tf`:

```bash
nano outputs.tf
```

**Functions:**
- **Generate ConfigMap:** Provides a ConfigMap for AWS authentication with Kubernetes.
- **Generate kubeconfig:** Outputs a kubeconfig file for connecting to the EKS cluster.

```hcl
# Outputs
locals {
  config_map_aws_auth = <<CONFIGMAPAWSAUTH
apiVersion: v1
kind: ConfigMap
metadata:
  name: aws-auth
  namespace: kube-system
data:
  mapRoles: |
    - rolearn: ${aws_iam_role.my-cluster-node.arn}
      username: system:node:{{EC2PrivateDNSName}}
      groups:
        - system:bootstrappers
        - system:nodes
CONFIGMAPAWSAUTH

  kubeconfig = <<KUBECONFIG
apiVersion: v1
clusters:
- cluster:
    server: ${aws_eks_cluster.my-cluster.endpoint}
    certificate-authority-data: ${aws_eks_cluster.my-cluster.certificate_authority.0.data}
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: aws
  name: aws
current-context: aws
kind: Config
preferences: {}
users:
- name: aws
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1beta1
      command: aws-iam-authenticator
      args:
        - "token"
        - "-i"
        - "${var.cluster-name}"
KUBECONFIG
}

output "config_map_aws_auth" {
  value = local.config

_map_aws_auth
}

output "kubeconfig" {
  value = local.kubeconfig
}
```

#### **5. Configure Provider**
Edit `providers.tf`:

```bash
nano providers.tf
```

**Functions:**
- **AWS Provider:** Specifies the AWS region and sets up necessary data sources.

```hcl
# Provider Configuration
provider "aws" {
  region  = "us-east-1"
}

data "aws_region" "current" {}

data "aws_availability_zones" "available" {}
```

#### **6. Configure Variables**
Edit `variables.tf`:

```bash
nano variables.tf
```

**Functions:**
- **Define Variables:** Sets up configurable variables for cluster name, eks version, key pair, and instance type.

```hcl
# Variables Configuration
variable "cluster-name" {
  default = "my-cluster"
  type    = string
}

variable "eks_version" {
  default = "1.30"
  type    = string
}

variable "key_pair_name" {
  default = "mycloud"
}

variable "eks_node_instance_type" {
  default = "t2.medium"
}
```

#### **7. Configure VPC Resources**
Edit `vpc.tf`:

```bash
nano vpc.tf
```

**Functions:**
- **Create VPC:** Defines the VPC for the EKS cluster.
- **Create Subnets:** Sets up public subnets in the VPC.
- **Create Internet Gateway:** Allows internet access for the VPC.
- **Create Route Table:** Routes traffic from subnets to the internet.
- **Associate Route Table:** Links route table with subnets.

```hcl
# VPC Resources
resource "aws_vpc" "my-vpc" {
  cidr_block = "10.0.0.0/16"
  tags = tomap({
    "Name" = "my-eks-node",
    "kubernetes.io/cluster/${var.cluster-name}" = "shared",
  })
}

resource "aws_subnet" "my-vpc" {
  count = 2
  availability_zone       = data.aws_availability_zones.available.names[count.index]
  cidr_block              = "10.0.${count.index}.0/24"
  map_public_ip_on_launch = true
  vpc_id                  = aws_vpc.my-vpc.id
  tags = tomap({
    "Name" = "my-eks-node",
    "kubernetes.io/cluster/${var.cluster-name}" = "shared",
  })
}

resource "aws_internet_gateway" "my-vpc" {
  vpc_id = aws_vpc.my-vpc.id
  tags = { Name = "my-vpc" }
}

resource "aws_route_table" "my-vpc" {
  vpc_id = aws_vpc.my-vpc.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.my-vpc.id
  }
}

resource "aws_route_table_association" "my-vpc" {
  count = 2
  subnet_id      = aws_subnet.my-vpc.*.id[count.index]
  route_table_id = aws_route_table.my-vpc.id
}
```

#### **8. Configure Workstation External IP**
Edit `workstation-external-ip.tf`:

```bash
nano workstation-external-ip.tf
```

**Functions:**
- **Fetch External IP:** Retrieves the external IP of your local workstation for configuring inbound access.

```hcl
# Workstation External IP
data "http" "workstation-external-ip" {
  url = "http://ipv4.icanhazip.com"
}

locals {
  workstation-external-cidr = "${chomp(data.http.workstation-external-ip.response_body)}/32"
}
```

## Final Steps to Deploy Your AWS Kubernetes Cluster with Terraform

After setting up your Terraform configuration files, it is time to deploy and manage your AWS Kubernetes cluster effectively.

### **1. Initialize Terraform Modules**

This will download the necessary provider plugins and prepare your working directory for further operations.

**Command:**

```bash
terraform init
```

### **2. Create Clusters and Resources**

With Terraform initialized, you can now create the execution plan and apply the configuration to set up your Kubernetes cluster and related resources.

- **Create an Execution Plan**

  Run the following command to generate a plan for applying your configuration. This plan will show you the changes Terraform will make.

  **Command:**

  ```bash
  terraform plan
  ```

- **Apply the Configuration**

  Once you're satisfied with the execution plan, apply the configuration to create the resources. The `-auto-approve` flag automatically approves all changes without prompting for confirmation.

  **Command:**

  ```bash
  terraform apply -auto-approve
  ```

### **3. Update Kubeconfig**

To interact with your newly created EKS cluster, you need to update your kubeconfig file. This allows `kubectl` to communicate with your Kubernetes cluster.

**Command:**

```bash
aws eks update-kubeconfig --region <aws region> --name <cluster-name>

aws eks update-kubeconfig --region us-east-1 --name my-cluster
```

Replace `aws region` with your AWS region and `cluster-name` with your cluster name if they differ.

### **4. Confirm Cluster and Nodes**

Finally, verify that your EKS cluster and worker nodes are up and running:

- **List EKS Clusters**

  Use this command to list all EKS clusters in your account, confirming that your cluster was created successfully.

  **Command:**

  ```bash
  aws eks list-clusters
  ```

- **Get Node Details**

  Check the status and details of the nodes in your cluster using `kubectl`. This command provides information about the nodes' health and configuration.

  **Command:**

  ```bash
  kubectl get nodes -o wide
  ```


## Deploying Your Application: Defining Kubernetes Manifests

With your AWS EKS cluster up and running, the next step is to define Kubernetes manifests for deploying and exposing your application. Below is the configuration to deploy your application.

### Kubernetes YAML Manifest

Create a file named `hotel-manifest.yaml` and include the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hotel-deployment
spec:
  replicas: 2  
  selector:
    matchLabels:
      app: hotel
  template:
    metadata:
      labels:
        app: hotel
    spec:
      containers:
      - name: hotel
        image: <dockerhubusername>/hotel-app:v1
        ports:
        - containerPort: 3000  # Port your application listens on
---
apiVersion: v1
kind: Service
metadata:
  name: hotel-service
spec:
  selector:
    app: hotel
  ports:
    - protocol: TCP
      port: 80  # Port exposed by the service externally (outside the cluster)
      targetPort: 3000  # Port your application listens on inside the pods
  type: LoadBalancer
```

### Kubernetes YAML Explanation

- **Deployment**: Manages the deployment of your application.
  - **Replicas**: Ensures high availability by running 2 instances of the application.
  - **Selectors and Labels**: Used to match the pods that the deployment manages.
  - **Containers**: Specifies the container image (`<dockerhubusername>/hotel-app:v1`) and the port (3000) that the application listens on.

- **Service**: Exposes your application to external traffic.
  - **Selector**: Routes traffic to pods with the label `app: hotel`.
  - **Ports**: Maps the external port 80 to the internal port 3000.
  - **Type**: `LoadBalancer` provisions a load balancer to expose the service to the internet.

### Applying the Manifests

To deploy the application, execute the following command:

```bash
kubectl apply -f hotel-manifest.yaml
```

After applying the manifests, check the status of your services using:

```bash
kubectl get svc
```

Look for the `LoadBalancer` type service to find the external URL where your application is accessible. Use this URL to interact with your hotel booking service through your browser.

## Set Up Monitoring and visualization with Prometheus and Grafana

Monitoring and logging are crucial for maintaining the health of your Kubernetes clusters. This guide will walk you through installing and configuring Prometheus and Grafana using Helm charts.

### Helm Chart Installations

First, you need to install Helm on your system:

```bash
curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
sudo apt-get install apt-transport-https --yes
echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

### Add Helm Repositories

Add the Helm repositories necessary for Prometheus and Grafana:

#### Add the Helm Stable Charts Repository

```bash
helm repo add stable https://charts.helm.sh/stable
```

#### Add the Prometheus Helm Repository

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

Verify the added repositories:

```bash
helm repo ls
```
<img width="530" alt="image" src="https://github.com/user-attachments/assets/286962f9-e0ad-40fb-b41a-88a2bd4c9ef4">

Update your Helm repositories:

```bash
helm repo update
```

### Create Prometheus Namespace

Create a namespace for Prometheus:

```bash
kubectl create namespace prometheus
```

### Install Prometheus and the kube-prometheus-stack

Install Prometheus and the kube-prometheus-stack using Helm:

```bash
helm install stable prometheus-community/kube-prometheus-stack -n prometheus
```
<img width="611" alt="image" src="https://github.com/user-attachments/assets/0a831ae1-f389-41a3-9cee-7a873303ee89">

### Check Installation Status

Verify the installation by checking the pods in the `prometheus` namespace:

```bash
kubectl get pods -n prometheus
```
<img width="611" alt="image" src="https://github.com/user-attachments/assets/6d986a88-2f09-42be-992c-89a629a56550">

Check the services to ensure both Prometheus and Grafana are running:

```bash
kubectl get svc -n prometheus
```
<img width="614" alt="image" src="https://github.com/user-attachments/assets/0d4719aa-f210-4b5e-958c-bcf4ced56bf0">

### Expose Prometheus for External Access

To access Prometheus externally, edit the service type from `ClusterIP` to `LoadBalancer`:

```bash
kubectl edit svc stable-kube-prometheus-sta-prometheus -n prometheus
```

Update the configuration:

```yaml
selector:
  app.kubernetes.io/name: prometheus
  operator.prometheus.io/name: stable-kube-prometheus-sta-prometheus
sessionAffinity: None
type: LoadBalancer
```

Access Prometheus using the LoadBalancer URL.

### Provision Grafana

Similarly, expose Grafana externally by changing the service type:

```bash
kubectl edit svc stable-grafana -n prometheus
```

Update the configuration:

```yaml
selector:
  app.kubernetes.io/instance: stable
  app.kubernetes.io/name: grafana
sessionAffinity: None
type: LoadBalancer
```

Check the services again to observe the changes:

```bash
kubectl get svc -n prometheus
```
<img width="959" alt="image" src="https://github.com/user-attachments/assets/88743afb-b856-410b-a8f4-13498022a7f8">

### Access Grafana GUI

Access Grafana using the LoadBalancer URL. Log in with the following credentials:

- **Username**: admin
- **Password**: prom-operator

### Visualize Metrics

To visualize metrics in Grafana:

1. **Add Data Source**: Click **Add data source** and select Prometheus. Alternatively, open the menu, select **Connections**, then **Data sources**.
   
   The data source "prometheus" is already added by default.

2. **View Dashboards**: Open the dashboard panel to see available dashboards. Click on a dashboard that suits your monitoring needs.

This will display monitoring metrics for your cluster nodes.

<img width="683" alt="image" src="https://github.com/user-attachments/assets/995665f9-6e60-4d2c-9f0f-1bb2b019b4f6">

<img width="734" alt="image" src="https://github.com/user-attachments/assets/13265223-f111-48ea-b23c-7bf33b20e5d6">

<img width="728" alt="image" src="https://github.com/user-attachments/assets/faa0f2e1-270d-4d5c-84d1-82d116f3524f">

<img width="724" alt="image" src="https://github.com/user-attachments/assets/b4e2ef24-ace6-43d6-bf85-f59d8f0d3d20">


## Implementing CI/CD Pipelines with GitHub Actions to automate Docker Build, Security Checks, and Application Deployment

Automating your CI/CD pipeline ensures that your application is consistently built, tested, and deployed with minimal manual intervention.

### Configure GitHub Repository Settings

1. **Enable GitHub Actions:**

   - Navigate to your GitHub repository.
   - Go to **Settings**.
   - Click on **Actions** in the left sidebar.
   - Under **General**, check the box for `Allow all actions and reusable workflows`.

2. **Add Repository Secrets:**

   - Go to **Settings** > **Secrets and variables** > **Actions**.
   - Click **New repository secret**.
   - Add your credentials for `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `DOCKERHUB_PASSWORD`, and `DOCKERHUB_USERNAME`.

### Create GitHub Workflow File

Ensure the following path exists in your cloned GitHub repository. If not, create it:

`.github/workflows/hotel-workflow.yml`

Below is the GitHub Actions workflow I configured to automate the Docker build, security checks, and deployment:

```yaml
name: Hotel Pipeline

on:
  push:
    branches:
      - main

jobs:
  codeql-analysis:
    name: Build, Scan, and Deploy Hotel Application
    runs-on: ${{ matrix.language == 'swift' && 'macos-latest' || 'ubuntu-latest' }}
    timeout-minutes: ${{ matrix.language == 'swift' && 120 || 360 }}
    permissions:
      security-events: write
      packages: read
      actions: read
      contents: read

    strategy:
      fail-fast: false
      matrix:
        language:
          - javascript-typescript
        build-mode:
          - none

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Initialize CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: ${{ matrix.language }}
          build-mode: ${{ matrix.build-mode }}

      - if: matrix.build-mode == 'manual'
        name: Build Project
        run: |
          echo 'If manual build mode is set, replace this with your build commands.'
          exit 1

      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v3
        with:
          category: "/language:${{ matrix.language }}"

      - name: Install Trivy
        run: |
         wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
         echo "deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -cs) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
         sudo apt-get update && sudo apt-get install trivy

      - name: Docker login
        run: echo ${{ secrets.DOCKERHUB_PASSWORD }} | docker login -u ${{ secrets.DOCKERHUB_USERNAME }} --password-stdin     

      - name: Build hotel booking image
        run: |
          docker build -t <dockerhubusername>/hotel:v1 .

      - name: Scan Docker image with Trivy
        run: |
          trivy image -f json -o trivy_report.json <dockerhubusername>/hotel:v1
 
      - name: Convert Trivy JSON to human-readable format
        run: |
          trivy image -f table <dockerhubusername>/hotel:v1 > trivy_report.txt
 
      - name: Save Trivy scan results as artifact
        uses: actions/upload-artifact@v4
        with:
          name: trivy-scan-results
          path: trivy_report.txt

      - name: Push hotel booking image
        run: |
         docker push <dockerhubusername>/hotel:v1
         docker rmi <dockerhubusername>/hotel:v1

      - name: Set up kubectl
        uses: azure/setup-kubectl@v3
        with:
         version: 'latest'

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v3
        with:
         aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
         aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
         aws-region: us-east-1

      - name: Update kubeconfig for EKS
        run: |
         aws eks update-kubeconfig --region us-east-1 --name my-cluster
 
      - name: Deploy to EKS
        run: |
         kubectl apply -f K8S/deployment.yml
```

   Replace <dockerhubusername> with your actual docker hub user name.
   Save and commit the yaml.
   
   Any other save/commit/push made to any code/file in this repository will now automatically triger GitHub action to run this pipeline.
   
### GitHub Workflow Explanation

- **Trigger**: The workflow activates on a push to the `main` branch.
- **Jobs**: Defines a job named `codeql-analysis` that handles building, scanning, and deploying the hotel application.
  - **Strategy**: Uses a matrix strategy for different languages and build modes.

- **Steps**:
  - **Checkout**: Checks out the code from the repository.
  - **Initialize CodeQL**: Sets up CodeQL for code security analysis.
  - **Perform CodeQL Analysis**: Analyzes the code for vulnerabilities.
  - **Install Trivy**: Installs Trivy for scanning Docker images.
  - **Docker Login**: Logs in to DockerHub using provided credentials.
  - **Build Docker Image**: Builds the Docker image for the hotel application.
  - **Scan Docker Image**: Scans the Docker image with Trivy and saves the report.
  - **Convert Trivy JSON**: Converts Trivy scan results to a human-readable format and saves as an artifact.
  - **Push Docker Image**: Pushes the Docker image to DockerHub and cleans up local images.
  - **Set up kubectl**: Configures `kubectl` for Kubernetes operations.
  - **Configure AWS Credentials**: Sets up AWS credentials for accessing EKS.
  - **Update kubeconfig**: Updates `kubeconfig` for the EKS cluster.
  - **Deploy to EKS**: Deploys the application to the EKS cluster using Kubernetes manifests.

### Check the Deployment

To verify that your deployment is successful and accessible, use the following command to get the external URL of the load balancer:

```bash
kubectl get svc
```

![Load Balancer URL](https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/3c0631e4-fc05-434c-b2ea-8d710846d330)

![Load Balancer Details](https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/bf2638e3-6a6b-46b9-8f9f-cee476aef5d7)

### Handling Pipeline Failures

Building and testing a pipeline might involve some failures. Check the build logs to diagnose issues. Below are screenshots showing both failed and successful workflow runs.

![Failed Workflow](https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/a4f4b43b-c5e8-49d8-a5d3-4048556992c3)

### Testing Changes

For example, updating the `AboutUs.jsx` file located at `src/routes/about-us/AboutUs.jsx` with new contact information and committing the change triggers the pipeline, applying the update immediately.

![Update Commit](https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/88b84062-507a-4b21-80cf-05268a34808e)
![Pipeline Triggered](https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/59c88e8a-2527-40c5-afb5-7b10e15c5ed1)
![Applied Changes](https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/b922e957-a6f3-4c28-9008-ce0261022a0b)

You can make further changes, for instance, in the `src/routes/home/Home.jsx` file.

### Review Scan Results

The Trivy scan results are available at the following repository download URL: [Scan Results](https://github.com/balogsun/hotel-booking/actions/runs/9589669410/artifacts/1618838601)

![Trivy Scan Results](https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/bf2c3f37-f789-4e78-8867-6ced99b95b97)

### CodeQL Scan Results

The CodeQL analysis scanned 20 out of 20 JavaScript files. Check the status page for overall coverage information: [CodeQL Scan Result](https://github.com/balogsun/hotel-booking/security/code-scanning/tools/CodeQL/status/)

![CodeQL Scan Coverage](https://github.com/Makinates/SeunB-tasks-docs/assets/125329091/91a44d8e-256f-4ce7-b8d0-dbedd83b1ad9)

### Security Best Practices

To enhance security, I utilized GitHub's secrets management by storing sensitive credentials in the repository settings and referencing them in the workflow.

## Conclusion

The microservices architecture for the Hotel Service application ensures modularity, scalability, and maintainability. By using Docker for containerization and Kubernetes for orchestration, along with a robust CI/CD pipeline implemented with GitHub Actions, you create an efficient and automated deployment process. Comprehensive monitoring and logging further guarantee operational visibility and continuous delivery, setting a strong foundation for future growth and scalability.

