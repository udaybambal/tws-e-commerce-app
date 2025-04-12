
# 🚀 **TWS-EasyShop - DevOps Mega Project**  
> A modern full-stack e-commerce platform with smooth performance and rich features.

Built using **Next.js 14**, **TypeScript**, and **MongoDB**, with a polished UI via **Tailwind CSS**, real-time cart updates, and secure authentication.  

---

##  **Tech Stack & Tools**
- **Frontend:** Next.js 14 + TypeScript  
- **Backend:** MongoDB + API Routes  
- **UI:** Tailwind CSS  
- **Infrastructure:** Terraform + AWS  
- **CI/CD:** Jenkins + Argo CD  
- **Containerization & Orchestration:** Docker + Kubernetes  
- **Other Tools:** kubectl, AWS CLI

---

##  **Prerequisites**
Make sure the following tools are installed:

| Tool        | Purpose                  |
|-------------|--------------------------|
| Terraform   | Infra provisioning       |
| AWS CLI     | AWS interaction          |
| kubectl     | Kubernetes CLI           |
| Jenkins     | CI pipeline              |
| Docker      | Containerization         |

---

##  **Project Setup Guide**

<details>
<summary><strong> Install Terraform</strong></summary>

```bash
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt-get update && sudo apt-get install terraform
```

 **Verify:**
```bash
terraform -v
```
</details>

<details>
<summary><strong> Install AWS CLI</strong></summary>

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install
```

 **Configure:**
```bash
aws configure
```
</details>

---

##  **Infrastructure Setup (Terraform)**

<details>
<summary><strong> Provision EKS Cluster</strong></summary>

1. **Clone & Navigate:**
   ```bash
   git clone https://github.com/LondheShubham153/tws-e-commerce-app.git
   cd terraform
   ```

2. **Generate SSH Key:**
   ```bash
   ssh-keygen -f terra-key
   chmod 400 terra-key
   ```

3. **Initialize & Apply:**
   ```bash
   terraform init
   terraform plan
   terraform apply
   ```

4. **Access EC2:**
   ```bash
   ssh -i terra-key ubuntu@<public-ip>
   ```

5. **Configure kubectl:**
   ```bash
   aws eks --region eu-west-1 update-kubeconfig --name tws-eks-cluster
   kubectl get nodes
   ```
</details>

---

##  **Jenkins Setup**

<details>
<summary><strong> Start & Configure Jenkins</strong></summary>

```bash
sudo systemctl status jenkins
sudo systemctl enable jenkins
sudo systemctl restart jenkins
```

 Access Jenkins:  
`http://<public-ip>:8080`

 Get Admin Password:
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

 Install Plugins:
- Docker Pipeline
- Pipeline View

 Add Credentials:
- GitHub → `github-credentials`
- DockerHub → `docker-hub-credentials`
</details>

<details>
<summary><strong> Create Pipeline</strong></summary>

- Job Name: `EasyShop`
- Type: `Pipeline`
- SCM:  
  - Repo: `https://github.com/<your-username>/tws-e-commerce-app`  
  - Credentials: `github-credentials`  
  - Branch: `master`  
  - Path: `Jenkinsfile`

 Fork Repos & Update:
- App repo: Update DockerHub username in `Jenkinsfile`
- Shared lib repo: Update `vars/update_k8s_manifest.groovy`
</details>

<details>
<summary><strong> GitHub Webhook</strong></summary>

- Go to: Repo → Settings → Webhooks  
- **Payload URL:** `http://<your-Jenkins-IP>:8080/github-webhook/`
</details>

---

##  **Continuous Deployment (CD)**

<details>
<summary><strong> Argo CD Setup</strong></summary>

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
watch kubectl get pods -n argocd
```

 Patch Service:
```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
kubectl get svc -n argocd
```

 Port Forward & Access:
```bash
kubectl port-forward svc/argocd-server -n argocd <your-port>:443 --address=0.0.0.0 &
```

 Get Admin Password:
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```
</details>

<details>
<summary><strong> Deploy with Argo CD</strong></summary>

1. Open Argo CD UI → **New App**  
2. Fill details:
   - App Name: `EasyShop`
   - Repo URL: Your Git repo
   - Path: `Kubernetes`
   - Cluster: `https://kubernetes.default.svc`
   - Namespace: `tws-e-commerce-app`
   - Sync Policy: `Automatic`
3. Click **Create**
</details>

---

##  **NGINX Ingress + Cert-Manager**

<details>
<summary><strong> NGINX Ingress (via Helm)</strong></summary>

```bash
kubectl create namespace ingress-nginx
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install nginx-ingress ingress-nginx/ingress-nginx   --namespace ingress-nginx   --set controller.service.type=LoadBalancer
```

 Get IP:
```bash
kubectl get svc -n ingress-nginx
```
</details>

<details>
<summary><strong> Cert-Manager (via Helm)</strong></summary>

```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager jetstack/cert-manager   --namespace cert-manager   --create-namespace   --version v1.12.0   --set installCRDs=true
```

 DNS Configuration:
```bash
kubectl get svc nginx-ingress-ingress-nginx-controller -n ingress-nginx   -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```
Update your DNS provider with a **CNAME** pointing to this hostname.
</details>

---

##  **You're All Set!**
> Your **EasyShop** app is fully deployed and ready to rock!  
> Happy shipping 🚀
