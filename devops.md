EasyShop is a modern, full-stack e-commerce platform built with Next.js 14, TypeScript, and MongoDB. It features a beautiful UI with Tailwind CSS, secure authentication, real-time cart updates, and a seamless shopping experience.

## Features

- Modern and responsive UI with dark mode support
- Secure JWT-based authentication
- Real-time cart management with Redux
- Mobile-first design approach
- Advanced product search and filtering
- Secure checkout process
- Multiple product categories
- User profiles and order history
- Dark/Light theme support

## PreRequisites
Before you begin setting up this project, make sure the following tools are installed and configured properly on your system:
### 1. Install Terraform
Terraform is an open-source infrastructure as code (IaC) tool used for provisioning and managing cloud resources.
Setup & Initialization
Install Terraform
# Linux & macOS
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt-get update && sudo apt-get install terraform

# Verify Installation
terraform -v

Initialize Terraform
terraform init

2. Install AWS CLI
AWS CLI (Command Line Interface) allows you to interact with AWS services directly from the command line.
aws configure
This will prompt you to enter:
AWS Access Key ID


AWS Secret Access Key


Default region name


Default output format


Make sure the IAM user you're using has the necessary permissions. You’ll need an AWS IAM Role with programmatic access enabled, along with the Access Key and Secret Key.
AWS IAM
Role: Access Key & Secret Key
Getting Started
Follow the steps below to get your infrastructure up and running using Terraform:
1. Clone the Repository
First, clone this repo to your local machine:
git clone https://github.com/LondheShubham153/tws-e-commerce-app.git
cd terraform
2. Generate SSH Key Pair
Create a new SSH key to access your EC2 instance:
ssh-keygen -f terra-key
This will prompt you to create a new key file named terra-key.
3. Initialize Terraform
Initialize the Terraform working directory to download required providers:
terraform init

4. Review the Execution Plan
Before applying changes, always check the execution plan:
terraform plan


5. Apply the Configuration
Now, apply the changes and create the infrastructure:
terraform apply
Confirm with yes when prompted.

6. Access Your EC2 Instance
After deployment, grab the public IP of your EC2 instance from the output or AWS Console, then connect using SSH:
ssh -i terra-key ubuntu@<public-ip>
Make sure the key file has proper permissions:
chmod 400 terra-key

Jenkins Setup Steps
Debug Tip
Check if jenkins service is running:

systemctl status jenkins

Steps to Access Jenkins & Install Plugins
1️. Open Jenkins in Browser
Use your public IP with port 8080:

http://<public-ip>:8080

2️. Start Jenkins (If Not Running)
Start the service and get the Jenkins initial admin password:

sudo systemctl start jenkins
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

3️. Install Essential Plugins
Navigate to:
Manage Jenkins → Plugins → Available Plugins
Search and install the following:
Docker Pipeline
Pipeline View

Step 4: Set Up Docker & GitHub Credentials in Jenkins (Global Credentials)
GitHub Credentials:
Go to:
Jenkins → Manage Jenkins → Credentials → (Global) → Add Credentials


Use:
Kind: Username with password or Personal Access Token
ID: github-credentials




DockerHub Credentials:
Go to the same Global Credentials section
Use:
Kind: Username with password
ID: docker-hub-credentials
Use these IDs in your Jenkins pipeline for secure access to GitHub and DockerHub.

Step 5: Jenkins Shared Library Setup
 Configure Trusted Pipeline Library:
Go to:
 Jenkins → Manage Jenkins → Configure System


Scroll to Global Pipeline Libraries section

Add a New Shared Library:
Name: shared
Default Version: main
Project Repository URL: URL of the shared library repo that the learner has forked.
Note: Make sure the repo contains a proper directory structure like vars/, src/, and resources/
	
Step 6: Setup Pipeline [New Item in Jenkins]
Create New Pipeline Job
Name: EasyShop


Type: Pipeline


Configure GitHub Repository
GitHub Repo URL: Provide the learner’s forked repo URL


Branch: main or master (as per the repo)


Fork Required Repos
Fork App Repo:


Open the Jenkinsfile


Change the DockerHub username to yours


Fork Shared Library Repo:


Edit vars/update_k8s_manifest.groovy


Update with your DockerHub username


Setup Webhook
In GitHub:


Go to Settings → Webhooks


Add a new webhook pointing to your Jenkins URL


Select: "GitHub hook trigger for GITScm polling" in Jenkins job


Trigger the Pipeline
Click Build Now in Jenkins


Step 7: CD – Continuous Deployment Setup
Prerequisites:
Before configuring CD, make sure the following tools are installed:
Installations Required:
kubectl
AWS CLI


SSH into Bastion Server
Connect to your Bastion EC2 instance via SSH.


Note:
This is not the node where Jenkins is running. This is the intermediate EC2 (Bastion Host) used for accessing private resources like your EKS cluster.

Step 8: Configure AWS CLI on Bastion Server
Run the AWS configure command:

aws configure
Add your Access Key and Secret Key when prompted.







Step 9: Update Kubeconfig for EKS
Run the following important command:
aws eks update-kubeconfig --region eu-west-1 --name tws-eks-cluster
This command maps your EKS cluster with your Bastion server.


It helps to communicate with EKS components.


Step 10: Argo CD Setup
Create a Namespace for Argo CD
kubectl create namespace argocd
Install Argo CD using Manifest
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml


Watch Pod Creation
watch kubectl get pods -n argocd


This helps monitor when all Argo CD pods are up and running.


Check Argo CD Services
kubectl get svc -n argocd


Change Argo CD Server Service to NodePort
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'





Step 11: Access Argo CD GUI
Check Argo CD Server Port (again, post NodePort change)
kubectl get svc -n argocd

Port Forward to Access Argo CD in Browser
 Forward Argo CD service to access the GUI:
kubectl port-forward svc/argocd-server -n argocd <your-port>:443 --address=0.0.0.0 &


Replace <your-port> with a local port of your choice (e.g., 8080).

 Now, open https://<bastion-ip>:<your-port> in your browser.


Get the Argo CD Admin Password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
Log in to the Argo CD GUI


Username: admin


Password: (Use the decoded password from the previous command)


Update Your Password


On the left panel of Argo CD GUI, click on "User Info"


Select Update Password and change it.


Deploy Your Application in Argo CD GUI
On the Argo CD homepage, click on the “New App” button.


Fill in the following details:


Application Name:
 [Enter your desired app name]


Project Name:
 Select default from the dropdown.


Sync Policy:
 Choose Automatic.


In the “Source” section:


Repo URL:
 Add the Git repository URL that contains your Kubernetes manifests.


Path:
 Kubernetes (or the actual path inside the repo where your manifests reside)


In the “Destination” section:


Cluster URL:
 https://kubernetes.default.svc (usually shown as "default")


Namespace:
 tws-e-commerce-app (or your desired namespace)


Click on “Create”.
Congratulations!
Your project is now deployed via Argo CD 

