

**Minimum system requirements for minikube:**


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


**Deploying Java based RESTFUL application **

git clone https://github.com/jay8900/java-microservices.git

cd java-microservices/java-app1

mvn clean install 

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 17.210 s
[INFO] Finished at: 2017-09-30T11:28:37+01:00
[INFO] Final Memory: 41M/328M
[INFO] ------------------------------------------------------------------------

docker build -t jay8900/jay1:latest .


docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username:
Password:
Login Succeeded


docker push jay8900/jay1:latest


**Building the remaining applications**

$ cd ..

$ cd java-app2/

$ mvn clean install

…
$ docker build -t jay8900/jay2:latest  .
...
$ docker push jay8900/jay2:latest 
...

$ cd ..

$ cd java-app3/

$ mvn clean install

...

$ docker build -t jay8900/jay3:latest .
...


$ docker push jay8900/jay3:latest
…



**Deploying onto Kubernetes**


**Deploying the entire Java application in Kubernetes**

cd ../kubernetes

kubectl apply -f *.yaml


Viewing the complete application


minikube service shopfront




**Prometheus and Grafana setup in Minikube**


**Requirements**

Minikube
helm

Install Prometheus

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm install prometheus prometheus-community/prometheus

kubectl expose service prometheus-server --type=NodePort --target-port=9090 --name=prometheus-server-np


minikube service prometheus-server-np


**Install Grafana**

helm repo add grafana https://grafana.github.io/helm-charts

helm install grafana stable/grafana
kubectl expose service grafana --type=NodePort --target-port=3000 --name=grafana-np

kubectl get secret --namespace default grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

minikube service grafana-np












