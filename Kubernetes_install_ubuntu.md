
## **Here’s a step-by-step guide with commands and explanations for setting up a Kubernetes cluster with a master and worker nodes using kubeadm and Docker.**

### 1. Prepare All Nodes (Master and Workers)
Perform the following steps on all nodes (master and workers):

1.1. Switch to the root user:
> sudo su -

1.2. Update the System:

> apt-get update -y

This updates the local package index so you can get the latest versions of installed packages.

1.3. Install apt-transport-https:

apt-get install -y apt-transport-https
This ensures the system can retrieve packages over HTTPS, required for accessing the Kubernetes repositories.

### 2. Install Docker on All Nodes
Kubernetes uses Docker (or another container runtime) to manage containers.

2.1. Install Docker:

apt install -y docker.io

2.2. Verify Docker Installation:

docker --version

2.3. Start Docker:

systemctl start docker

2.4. Enable Docker to start on boot:

systemctl enable docker

This ensures Docker starts automatically after each reboot.

### 3. Add Kubernetes APT Repository on All Nodes
To install Kubernetes components, add its repository.

3.1. Add Google Cloud GPG Key:

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -


This adds the GPG key to ensure that the downloaded Kubernetes packages are trusted.

3.2. Add the Kubernetes APT Source:

nano /etc/apt/sources.list.d/kubernetes.list

Add the following line to this file:

deb http://apt.kubernetes.io/ kubernetes-xenial main

Save and exit (Ctrl+X, then Y, and Enter).

3.3. Update the Package List:

apt-get update -y

3.4. Install Kubernetes Components:


apt-get install -y kubelet kubeadm kubectl kubernetes-cni

kubelet: Runs on all nodes and manages the containers.
kubeadm: Tool to bootstrap the Kubernetes cluster.
kubectl: Command-line tool to interact with the cluster.
kubernetes-cni: Installs the Container Network Interface (CNI) plugins.

### 4. Initialize the Master Node (Only on Master Node)
Perform the following steps only on the master node:

4.1. Initialize the Kubernetes Master Node:

kubeadm init

This initializes the control plane (master) of your Kubernetes cluster. After running, this command will provide a join command like:


kubeadm join <Master-IP>:6443 --token <token> --discovery-token-ca-cert-hash <hash>



Save this join command. You will need it to connect worker nodes to the master.

4.2. Configure kubectl for the Master:

After initializing the master, set up kubectl (the command-line tool) to communicate with the cluster:


mkdir -p $HOME/.kube

cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

chown $(id -u):$(id -g) $HOME/.kube/config

This copies the admin configuration file (admin.conf) to the .kube directory in your home, allowing you to use kubectl to manage the cluster.

4.3. Deploy the Network Add-On (Flannel):
Kubernetes needs a networking plugin to allow communication between pods. Here, we use Flannel as an example:


kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

Flannel configures a network interface for inter-node communication in your Kubernetes cluster.

### 5. Add Worker Nodes to the Cluster

On each worker node, run the join command provided by the master after the kubeadm init step.

5.1. Join Worker Nodes:

On each worker node, execute the join command (the one printed during the master node initialization):


kubeadm join <Master-IP>:6443 --token <token> --discovery-token-ca-cert-hash <hash>

This command will connect the worker node to the master and register it with the cluster.

### 6. Verify the Cluster (Run on Master Node)
After all nodes (master and workers) are configured, verify the nodes from the master node:


kubectl get nodes

You should see the master node and all worker nodes listed with a status of Ready.

## Conclusion:

You now have a working Kubernetes cluster with one master node and multiple worker nodes. The process involved:

Installing and setting up Docker on all nodes.
Adding the Kubernetes APT repository and installing required components.
Initializing the master node with kubeadm.
Connecting worker nodes to the master node.
Verifying the setup using kubectl get nodes.
This basic setup can now be used to deploy applications, manage pods, and run workloads across the cluster.






