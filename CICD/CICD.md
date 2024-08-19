# **Setting Up a Cost-Effective CI/CD and Kubernetes Environment on Windows: A Comprehensive Guide**

If you’ve ever worried about the costs associated with setting up your own continuous integration and deployment (CI/CD) workflow, especially while trying to minimize cloud expenses, this guide is for you.

This guide provides a detailed approach to installing and configuring key tools needed for CI/CD on a Windows environment. It includes step-by-step instructions for setting up Chocolatey, Docker, Git, Minikube, kubectl, CircleCI, and ArgoCD. By following these steps, you'll create a robust development workflow that leverages these tools effectively while keeping costs low.

**Prerequisites:**

- GitHub or GitLab account
- Docker Hub account

## **Install Chocolatey for Windows**

Chocolatey is a package manager for Windows, designed to make the installation, upgrading, and management of software easier on the Windows platform. It's similar to package managers on other operating systems, such as `apt` on Debian-based systems or `yum` on Red Hat-based systems.

1. Search for and run the PowerShell application as an Administrator.
2. Run the command below in the PowerShell terminal:

   ```powershell
   Get-ExecutionPolicy
   ```

   If it returns `Restricted`, then run:

   ```powershell
   Set-ExecutionPolicy AllSigned
   ```

   or

   ```powershell
   Set-ExecutionPolicy Bypass -Scope Process
   ```

3. Run the following command to install Chocolatey:

   ```powershell
   Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
   ```

   If you don't see any errors, you are ready to use Chocolatey! You can also visit the [Chocolatey website](https://chocolatey.org/install) for an extensive guide.

### **Install Docker Desktop**

1. Follow the instructions at [Docker's official documentation](https://docs.docker.com/desktop/install/windows-install/).

### **Install Git**

1. Download and install Git from [Git SCM](https://git-scm.com/download/win).

### **Minikube for Windows**

Minikube is local Kubernetes, focusing on making it easy to learn and develop for Kubernetes.

All you need is Docker (or similarly compatible) container or a Virtual Machine environment, and Kubernetes is a single command away: minikube start

You may visit [Minikube's start guide](https://minikube.sigs.k8s.io/docs/start/) for setup.

 Requirements:

- 2 CPUs or more
- 2-4GB of free memory
- 20GB of free disk space
- Internet connection
- Virtual machine manager (e.g., Docker Desktop, QEMU, Hyperkit, Hyper-V, Podman, VirtualBox, VMware Fusion/Workstation)

 Install minikube with chocolatey and start your cluster:

   ```powershell
   choco install minikube
   minikube start
   ```

   If that returns an error related to the virtual environment, try:

   ```powershell
   docker context ls
   docker context use desktop-linux
   minikube start --driver=hyperv --docker-env="desktop-linux"
   ```

   Additional Minikube commands:

   ```bash
   minikube version
   minikube pause
   minikube unpause
   minikube stop
   minikube delete
   ```

   For more information, visit [Minikube's documentation](https://minikube.sigs.k8s.io/docs/start/) where you can find basic sample deployments you can try your hands on.

1. Follow the instructions at [Kubernetes official documentation](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/).

2. To install kubectl using Chocolatey:

   ```powershell
   choco install kubernetes-cli
   ```

3. Test to ensure the version installed is up-to-date:

   ```bash
   kubectl version --client
   ```

### **Clone the Repository**

1. Open Windows Command Prompt (search for CMD, right-click, and run as Administrator).
2. Change directory to a path away from the Windows installation path:

   ```bash
   cd C:\Users\username\Desktop
   ```

3. Clone the repository:

   ```bash
   git clone https://github.com/balogsun/hotel-booking.git
   cd hotel-booking
   ```

### **Set Up CircleCI**

To set up and configure CircleCI for continuous integration, follow these detailed steps:

## **1. Sign Up and Connect Your Repository**

- Go to the [CircleCI website](https://circleci.com) and sign up for a free account using GitHub or Bitbucket.

- Connect an organization
- Give any name of your choice as an organization
- In the CircleCI dashboard, select **Projects** from the sidebar.
- Click **Create Project** and choose the repository you want to connect.
- What would you like to do, select `Build, test, and deploy a software application`
- Give your project a name.
- Next, setup pipeline
- Name your pipeline
- Choose a repo, select either github, gitlab or gitbucket as repo source.
- Authorize CircleCI to access your GitHub or Bitbucket account.

## **2. CircleBot, will prepare a custom starter config file to build and test your code.**

2. **CircleCI Configuration**
   - A `.circleci/config.yml` file will be created automatically. Review it and click `submit and run`. You may not want to run the one that is suggested for you, you may simply select to run a `simlpe hello world` option, to proceed, then go to the github repo and change it to a working config that will build and store your codes as images in the github repository.

## **4. Configure Project Settings**

### **Environment Variables**

- In the CircleCI dashboard, go to **Project Settings** > **Environment Variables**.
- Add any necessary environment variables (e.g., `DOCKER_USER`, `DOCKER_PASS` for Docker authentication).
- Put in you credentials for your dockerhub account user here.

### **I have written below a Config File that will securely integrate the hotel booking application for circleCI**

   **Note**, I have deliberately instructed the security tools to bypass any vulnerabilty just for demo purposes only.
   The workflow will fail with an `exit code` if there are any vulnerbilities found in the code or docker images built. For production purpose, remove the string `|| true` in the config file.

    ```yaml
    version: 2.1
    
    executors:
      default:
        docker:
          - image: cimg/node:22.6.0
        working_directory: ~/project
    
    jobs:
      code_security_scan:
        executor: default
        steps:
          - checkout
    
          - run:
              name: Install dependencies
              command: npm install
    
          - run:
              name: Run npm audit (ignore failures)
              command: npm audit --audit-level=high || true
    
      build_and_scan:
        executor: default
        steps:
          - checkout
    
          - setup_remote_docker:
              version: default  # Ensure Docker is available
    
          - run:
              name: Docker login
              command: |
                echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
    
          - run:
              name: Build Docker Image
              command: docker build -t <githubuser>/hotel:v0 .
    
          - run:
              name: Install Trivy
              command: |
                curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /tmp/trivy v0.54.1
                sudo ln -s /tmp/trivy/trivy /usr/local/bin/trivy
    
          - run:
              name: Scan Docker Image for Vulnerabilities
              command: |
                trivy image --severity HIGH,CRITICAL <githubuser>/hotel:v0 || true
    
          - run:
              name: Push Docker Image
              command: docker push <githubuser>/hotel:v0
    
          - run:
              name: Remove Docker Image
              command: docker rmi <githubuser>/hotel:v0
    
    workflows:
      version: 2
      build_and_deploy:
        jobs:
          - code_security_scan
          - build_and_scan:
              requires:
                - code_security_scan
    
    ```

## **3. Commit and Push Changes**

- Commit the `.circleci/config.yml` file to your repository:

  ```bash
  git add .circleci/config.yml
  git commit -m "Add CircleCI configuration"
  git push
  ```

## **5. Trigger a Build**

- Push a commit to your repository to trigger the first build.
- Monitor the build process in the CircleCI dashboard under the **Pipelines** section.

## **6. Monitor and Manage**

- Use the CircleCI dashboard to view build logs, test results, and deployment status.
- Failed builds are automatically sent to your email, sometime in junk/spam folder, you may add it to safe sender list.

By following these steps, you can set up CircleCI to automate your project's build and test processes, integrating seamlessly with your existing GitHub or Bitbucket repositories.

### **Install ArgoCD in EKS Cluster**

1. Clone the repository and navigate to the directory:

   ```bash
   cd C:\Users\username\Desktop
   git clone https://github.com/balogsun/hotel-booking.git
   cd hotel-booking
   ```

2. Create the namespace and install ArgoCD:

   ```bash
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   kubectl -n argocd get all
   kubectl get svc -n argocd
   ```

3. Download and install Argo CD CLI:
    - Visit [ArgoCD releases](https://github.com/argoproj/argo-cd/releases/tag/v2.12.1) for the latest version. Explore the github repo for newer relaease if neccessary.
    - Run ArgoCD CLI commands from the Windows command prompt, Open windows cmd as administrator, change directory to location where you downloaded the argocd exe file. Do not double click on the file and try to run directly from windows, it may be flagged.

      ```bash
      argocd login $ARGOCD_SERVER --username admin --password $ARGO_PWD --insecure
      OR
      argocd login $ARGOCD_SERVER --username admin --password $ARGO_PWD --secure

      #Sample command below:
      argocd-windows-amd64.exe login <IpAddress>:portNo> --username admin --password xxxxxx –-insecure

      ```

    - Agrocd-server service is using “ClusterIP”. We can change it to NodePort” to access the agrocd UI from browser.

4. Change ArgoCD server service type to NodePort:

   ```bash
   kubectl edit svc argocd-server -n argocd
   OR
   kubectl edit svc argocd-server -n argocd -o yaml
   ```

   A notepad will be automatically opened, scroll down and change from `ClusterIP` to `NodePort`. Now `service` will be changed to “NodePort”

5. Note the Control Plane Node IP and ArgoCD service port that will be used to access the argoCD URL:

   ```bash
   kubectl get node -o wide
   kubectl get svc -n argocd
   ```

6. Access ArgoCD UI:

   Visit `http://ControlPlaneNodeIP:ArgocdServicePort`.

7. Retrieve the initial admin password:
 After reaching the UI the first time you can login with username: admin and the random password generated during the installation. You can find the password by running:

   ```bash
   kubectl get secret -n argocd
   kubectl describe secret argocd-initial-admin-secret -n argocd
   kubectl get secret -n argocd argocd-initial-admin-secret -o yaml
   ```

 The password is still encryped so you have to decrypt it.

 Windows may not support native base64 decoding, you can use an online website at <https://www.base64decode.org/>, insert the values and decode it.

- We can use this password to login. After login it is recommended to change the password.
- Update password in the GUI, User Info Section --> update password --> Save

9. You should delete the initial secret afterwards:

   ```bash
   kubectl get secret -n argocd
   kubectl -n argocd delete secret argocd-initial-admin-secret
   ```

- In the ArgoCD web interface, click on **Settings** -> **Repositories**.

- On the next page, click on **Connect Repo** and fill in the necessary details.

- Under “Repository URL,” provide the GitHub link where our code exists. Repo: `https://github.com/balogsun/hotel-booking.git`.

- Fill out just these three places and leave the rest of the options. Head up and click on **Connect**.

### **Now we have connected our GitHub repository with ArgoCD. Let's go build our application.**

- Click on the **Applications** page and click on **Create New App**.

- Fill in the details as above.

- Fill in the source details. The deployment YAML for me was inside the `K8S` path; we need to put that as our path.

- Select the cluster URL and namespace. Now click on **Create**. It will create the app.

- We can see our app deployed in our cluster server.

- We can access the app using the NodeIP URL.

- Now we can update our app with our CI tool to create a new image and push it to Docker Hub. Update that image details in your `K8S/deployment.yml` file in your repo and click on **Sync** in the app.

- When you click on **Sync**, you have some options tick boxes to select from. If using Argo CD for the first time in a development environment, here are some basic options you might consider selecting:

### **Options**

- **Revision**: The revision is set to `HEAD`, which means the latest commit in the default branch will be used.

- **DRY RUN**: This is a safe option to start with as it allows you to simulate the synchronization process without making any actual changes. It helps you verify what changes would be applied.

### **Sync Options**

- **AUTO-CREATE NAMESPACE**: Useful if you want Argo CD to automatically create the namespace for your application if it doesn’t exist. This can simplify setup in a development environment.

- **APPLY OUT OF SYNC ONLY**: This option ensures that only resources that are out of sync with the Git repository are updated, which can be useful for incremental updates.

### **Additional Considerations**

- **PRUNE**: You might want to use this cautiously. In a development environment, it can be useful to remove resources not defined in your Git repository, but ensure you understand its impact first.

- **RETRY**: Consider enabling this if you want Argo CD to automatically retry synchronization in case of failure, which can be helpful during development when changes are frequent.

These options provide a balance between safety and functionality, allowing you to manage your applications effectively while minimizing risks.

### **Prune Propagation Policy**

- **FOREGROUND**: Deletes resources in the foreground, blocking until the resource is fully deleted.

### **Additional Options**

- **REPLACE**: Replaces resources instead of updating them, which may lead to downtime.

- **RETRY**: Retries the synchronization process if it fails.

This configuration is used to manage how Argo CD synchronizes application manifests from a Git repository to a Kubernetes cluster. Each option provides control over how resources are applied and managed during the sync process.

Certainly! Here’s a conclusion statement for your report:

### **In Conclusion**

By following this guide, you’ll have set up a budget-friendly CI/CD system on Windows using tools like Chocolatey, Docker, Git, Minikube, kubectl, CircleCI, and ArgoCD. This setup makes your development process smoother and cuts down on cloud costs.

You’ll learn how to set up and use each tool, connect them together, and automate your development tasks. With Minikube, you can run Kubernetes locally, and with ArgoCD, you can deploy and manage your apps.

Using these tools will help you manage your code, build projects, and deploy apps more easily and effectively, making your development work more reliable and efficient.
