# Infrastructure Engineer Assignment

## Objective Overview:

#### Create a script (API or CLI) to automate Kubernetes cluster operations, including:
	1. Connecting to the cluster.
	2. Installing Helm and KEDA (for event-driven autoscaling).
	3. Deploying an application with autoscaling.
	4. Checking the health status of a deployment.


1. Connect to the Kubernetes Cluster:

Steps:
	
	1. Provision the environment:
	   
	I used VMware Workstation to create virtual environments. With the help of Vagrant, I provisioned two virtual machines (VMs) running Ubuntu 22.04. One for control plane and one for worker node. Below is the repository I used ```https://github.com/mialeevs/kube_vagrant.git```
	
	Vagrant command to provision VMs:
	```bash
	   	vagrant up
	```
	
	2. Install kubectl to interact with Kubernetes:
	 Installed the `kubectl` command-line tool to interact with the Kubernetes cluster.
	 Commands to install kubectl:
	```bash
	   	sudo apt-get update
	   	sudo apt-get install -y apt-transport-https ca-certificates curl gnupg
	  	curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
	   	sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg
	   	echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
	   	sudo apt-get update
	   	sudo apt-get install -y kubectl
	```
	
	3. Check cluster status:
	After setting up the cluster, we can check the node status to verify that it's running properly.
	Command to check node status:
	```bash
	   	kubectl get nodes
	```

2. Install Helm for Kubernetes Package Management:

Steps:
1. Install Helm:
 Helm is a package manager that simplifies the deployment of applications on Kubernetes. Used it to install tools like KEDA.
Commands to install Helm:
```bash
   	curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
   	sudo apt-get install apt-transport-https --yes
   	echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
   	sudo apt-get update
   	sudo apt-get install helm
```
2. Verify Helm installation:
 After installing Helm, verified it by checking its version:
```bash
   	helm version
```

3. Install KEDA (Kubernetes Event-Driven Autoscaling):

Steps:
1. Add KEDA Helm repository and install KEDA:
   Used Helm to install KEDA, which enables event-driven autoscaling in the cluster.
   Commands to install KEDA:
```bash
   	helm repo add kedacore https://kedacore.github.io/charts
   	helm repo update
   	helm install keda kedacore/keda --namespace keda --create namespace
```
2. Verify KEDA installation:
  To Check if the KEDA operator is running in the Kubernetes cluster.
```bash
	kubectl get pods --namespace keda
```
4. Create Kubernetes Deployment:
Steps:
1. Defining a Kubernetes Deployment:
Created a deployment that uses a public Docker image (Node.js app).
Deployment YAML:
```YAML
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: dev
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: dev
     template:
       metadata:
         labels:
           app: dev
       spec:
         containers:
         - name: nodeapp
           image: pulivarthijanvi25/nodeapp:v1
           ports:
           - containerPort: 3000
           resources:
             requests:
               cpu: "100m"
               memory: "200Mi"
             limits:
               cpu: "200m"
               memory: "400Mi"
```

2. Created a Kubernetes Service:
Exposed the deployment to the external port using a `NodePort` service.
Service YAML:
```YAML
   apiVersion: v1
   kind: Service
   metadata:
     name: dev
   spec:
     ports:
     - port: 3000
       protocol: TCP
       targetPort: 3000
     selector:
       app: dev
     type: NodePort
```
3. Apply the configurations:
 Applied the deployment and service configurations using `kubectl` commands.
```bash
 	kubectl apply -f dev-deploy.yaml
   	kubectl apply -f dev_svc.yaml
```
5. Setup KEDA Autoscaling (ScaledObject):
Steps:
1. Define a ScaledObject:
Used KEDA to configure autoscaling based on events. Here, I have set up a cron-based scale rule.
ScaledObject YAML:
```YAML
   apiVersion: keda.sh/v1alpha1
   kind: ScaledObject
   metadata:
     name: dev-deployment
     namespace: default
   spec:
     scaleTargetRef:
       apiVersion: apps/v1
       kind: Deployment
       name: dev
     minReplicaCount: 1
     maxReplicaCount: 5
     triggers:
     - type: cron
       metadata:
         timezone: Asia/Kolkata
         start: 16 * * * *
         end: 18 * * * *
         desiredReplicas: "2"
```
2. Apply the ScaledObject:
After defining the ScaledObject, applied it to enable KEDA autoscaling.
```bash
	kubectl apply -f dev-keda.yaml 
```
6. Retrieve Health Status of Deployment:
For Deployment Status:
 To get the health status of the deployment, we can check the status of the deployment and its pods.
Commands to check deployment and pod status:
```bash
	kubectl get deployment dev
   	kubectl get pods --selector=app=dev
```
