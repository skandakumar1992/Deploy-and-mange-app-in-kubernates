1) Create & connect to the EC2 instance 
   Launch EC2 → Ubuntu 22.04 → Instance type t3.medium → attach 20GB storage → set security group to allow SSH from your IP and  NodePort(30080).
   Download keypair mykey.pem.

2) Install Docker, kubectl, and Minikube (on the EC2 instance)
   Run the following on the EC2 instance:
   sudo apt update && sudo apt -y upgrade
   sudo apt -y install apt-transport-https ca-certificates curl gnupg lsb-release conntrack
 
   # install Docker (official repository)
     curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
     echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] \
    https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) stable" \
    | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt update
    sudo apt -y install docker-ce docker-ce-cli containerd.io
    sudo usermod -aG docker $USER
    newgrp docker

  # install kubectl
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    chmod +x kubectl
    sudo mv kubectl /usr/local/bin/

  # verify installs
    docker --version
    kubectl version --client
    minikube version

3) Start Minikube (on the EC2 instance)
   Start with the Docker driver (minikube will run k8s inside containers):
   minikube start --driver=docker --memory=4096 --cpus=2 --disk-size=20g
   minikube status
   kubectl cluster-info

4) Prepare the application image
   Build your own image on the Minikube Docker daemon (recommended to use Docker & Minikube together)
   Do this on EC2 (still inside the SSH session). First point your shell to minikube’s Docker daemon so images are visible to the cluster:
   eval $(minikube -p minikube docker-env)
   Create these files on EC2 (simple static site with nginx):Dockerfile and index.html

   Build the image:
   docker build -t myapp:latest .
   docker images | grep myapp

5) Create Kubernetes manifests
   Create deployment.yaml and service.yaml on the EC2 instance (or in your project directory).

6) Deploy to the Minikube cluster (on EC2)
   kubectl apply -f deployment.yaml
   kubectl apply -f service.yaml
   kubectl get deployments
   kubectl get pods -o wide
   kubectl get svc

7) Access the app from your browser 
   In your browser: http://<EC2_PUBLIC_IP>:30080

8) Verify pods, logs, describe, and scale (on EC2)
   kubectl get pods
 # replace <pod-name> with the name from kubectl get pods
   kubectl describe pod <pod-name>
   kubectl logs <pod-name>
   kubectl logs deployment/myapp-deployment
   kubectl scale deployment myapp-deployment --replicas=3
   kubectl get pods 

10) Push this project to GitHub

    git init
    git add deployment.yaml service.yaml Dockerfile index.html 
    git commit -m "minikube on EC2: deployment + service + Dockerfile"
    git remote add origin git@github.com:YOUR_GITHUB_USERNAME/YOUR_REPO_NAME.git
    git branch -M main
    git push -u origin main
    git remote add origin https://github.com/YOUR_GITHUB_USERNAME/YOUR_REPO_NAME.git
    git push -u origin main


11) Final checklist (copy & run)
⦁	    Create EC2 (Ubuntu 22.04), open SSH and NodePort ports.
⦁	    SSH into EC2.
⦁	    Install Docker, kubectl, minikube (#2).
⦁	    minikube start --driver=docker --memory=4096 --cpus=2
⦁	    eval $(minikube docker-env) then docker build -t myapp:latest .
⦁	    kubectl apply -f deployment.yaml and kubectl apply -f service.yaml
⦁	    kubectl get pods, kubectl logs <pod>, kubectl describe pod <pod>
⦁	    kubectl scale deployment myapp-deployment --replicas=3
⦁	    Push code to GitHub.
