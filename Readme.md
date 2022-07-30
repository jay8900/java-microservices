Minimum system requirements for minikube:
2 GB RAM or more
2 CPU / vCPU or more
20 GB free hard disk space or more
Docker / Virtual Machine Manager – KVM & VirtualBox

Prerequisites for minikube: 

Minimal Ubuntu 20.04 LTS / 21.04
Sudo User with root privileges
Stable Internet Connection

Step 1) Apply updates


$ sudo apt update -y
$ sudo apt upgrade -y

Step 2) Install Minikube dependencies

$ sudo apt install -y curl wget apt-transport-https

Step 3) Download Minikube Binary

$ wget https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

$ sudo cp minikube-linux-amd64 /usr/local/bin/minikube
$ sudo chmod +x /usr/local/bin/minikube

$ minikube version
minikube version: v1.21.0
commit: 76d74191d82c47883dc7e1319ef7cebd3e00ee11
$

Step 4) Install Kubectl utility

$ curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl

$ chmod +x kubectl
$ sudo mv kubectl /usr/local/bin/
$ kubectl version -o yaml


Step 4) Start the minikube

$ minikube start --driver=docker


**MetalLB Configuration in Minikube — To enable Kubernetes service of type “LoadBalancer”**

minikube addons list will show that metalLB is disabled.
Enable the addon using the command minikube addons enable metallb

As we have enabled the addon, now we have to configure the same for the addon to come into effect. We have to configure using the following command 

"minikube addons configure metallb" .

It will prompt for the IP Address range. As my minikube host IP is 192.168.99.100, I have given the range as below

Check the configuration which is stored in a config map, kubectl describe configmap config -n metallb-system
