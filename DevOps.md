# 🚀 **TWS-EasyShop - DevOps Mega Project**  
A full-stack, modern e-commerce platform built with **Next.js 14**, **TypeScript**, and **MongoDB**.  
🔹 Features: Tailwind CSS UI,  secure auth,  real-time cart updates, and a seamless shopping experience.

---

##  **Pre-Requisites**

>  **Important:**  
> Make sure the following tools are installed on your system before starting the setup:

- [Terraform](https://developer.hashicorp.com/terraform/downloads)
- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- SSH access (for EC2 & Bastion)
- Git & GitHub account

---

##  **Setup & Initialization**

<details>
<summary> <strong>1. Install & Configure Terraform</strong></summary>

####  For Linux & macOS:
```bash
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt-get update && sudo apt-get install terraform
```

####  Verify Installation:
```bash
terraform -v
```

####  Initialize Terraform:
```bash
terraform init
```
</details>

<details>
<summary> <strong>2. Install & Configure AWS CLI</strong></summary>

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install
```

####  Configure AWS CLI:
```bash
aws configure
```

> You'll be prompted for:
> - **AWS Access Key ID**
> - **AWS Secret Access Key**
> - **Default Region**
> - **Output Format**

>  **Note:** Ensure your IAM user has programmatic access and necessary permissions.
</details>

<details>
<summary> <strong>3. Clone the Repository</strong></summary>

```bash
git clone https://github.com/LondheShubham153/tws-e-commerce-app.git
cd terraform
```
</details>

<details>
<summary> <strong>4. Generate & Secure SSH Key</strong></summary>

```bash
ssh-keygen -f terra-key
chmod 400 terra-key
```
</details>

<details>
<summary> <strong>5. Deploy Infrastructure Using Terraform</strong></summary>

```bash
terraform init
terraform plan
terraform apply
```

>  Confirm with `yes` when prompted.
</details>

<details>
<summary> <strong>6. Connect to EC2 & Configure kubeconfig</strong></summary>

```bash
ssh -i terra-key ubuntu@<public-ip>
```

####  Update kubeconfig:
```bash
aws configure
aws eks --region eu-west-1 update-kubeconfig --name tws-eks-cluster
kubectl get nodes
```
</details>

---

##  **Jenkins Setup**

<details>
<summary> <strong>Check & Start Jenkins</strong></summary>

```bash
sudo systemctl status jenkins
sudo systemctl enable jenkins
sudo systemctl restart jenkins
```
</details>

<details>
<summary> <strong>Access Jenkins & Install Plugins</strong></summary>

- Open: `http://<public_ip>:8080`
- Get admin password:
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

###  Recommended Plugins:
- Docker Pipeline  
- Pipeline View
</details>

<details>
<summary> <strong>Set Up Jenkins Credentials</strong></summary>

- **GitHub:**  
  - Kind: `Username with password / PAT`  
  - ID: `github-credentials`

- **DockerHub:**  
  - Kind: `Username with password`  
  - ID: `docker-hub-credentials`
</details>

<details>
<summary> <strong>Configure Shared Library & Create Pipeline</strong></summary>

- Go to **Jenkins → Manage Jenkins → Configure System**
- Add **Global Pipeline Library**:
  - Name: `shared`
  - Repo: `https://github.com/<your-username>/jenkins-shared-libraries`
  - Default Version: `main`

- Create a new pipeline job:
  - Name: `EasyShop`
  - Type: `Pipeline`
  - Use GitHub hook trigger
  - Script from SCM:
    - Repo: `https://github.com/<your-username>/tws-e-commerce-app`
    - Credentials: `github-credentials`
    - Branch: `master`
    - Script Path: `Jenkinsfile`
</details>

<details>
<summary> <strong>Set Up Webhook in GitHub</strong></summary>

- Go to: GitHub → Settings → Webhooks
- URL: `http://<jenkins-ip>:8080/github-webhook/`
- Enable: **"GitHub hook trigger for GITScm polling"** in Jenkins job
</details>

---

##  **Continuous Deployment with Argo CD**

<details>
<summary> <strong>Install Argo CD</strong></summary>

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
watch kubectl get pods -n argocd
```
</details>

<details>
<summary> <strong>Access Argo CD GUI</strong></summary>

```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
kubectl port-forward svc/argocd-server -n argocd <your-port>:443 --address=0.0.0.0 &
```

Open in browser:  
`https://<bastion-ip>:<your-port>`

 Get login password:
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

Login:  
- Username: `admin`  
- Password: *(use decoded output)*
</details>

<details>
<summary> <strong>Deploy App in Argo CD</strong></summary>

- Click `New App` in Argo CD UI  
- Fill in the following:
  - App Name: `easyshop` (or your preferred)
  - Repo URL: *Your GitHub repo with K8s manifests*
  - Path: `/kubernetes`
  - Cluster URL: `https://kubernetes.default.svc`
  - Namespace: `tws-e-commerce-app`
  - Sync Policy: `Automatic`

>  Click **Create**
</details>

---

##  **Congratulations!**  
Your **EasyShop** infrastructure is now fully operational with automated CI/CD using **Jenkins**, **Terraform**, and **Argo CD**.  
Happy shipping! 

