

# Minimum system requirements for minikube:


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


# Deploying Java based RESTFUL application 

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


# Viewing the complete application


minikube service shopfront


<img width="881" alt="image" src="https://user-images.githubusercontent.com/90141704/181884333-1ae94553-5088-4956-9709-f11baf65900c.png">


# Prometheus and Grafana setup in Minikube


**Requirements**

Minikube
helm

# Install Prometheus

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm install prometheus prometheus-community/prometheus

kubectl expose service prometheus-server --type=NodePort --target-port=9090 --name=prometheus-server-np


minikube service prometheus-server-np


# Install Grafana

helm repo add grafana https://grafana.github.io/helm-charts

helm install grafana stable/grafana
kubectl expose service grafana --type=NodePort --target-port=3000 --name=grafana-np

kubectl get secret --namespace default grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

minikube service grafana-np


Configure Prometheus Datasource
Once we’re logged in to the admin interface, it’s time to configure the Prometheus Datasource.

We need to head to Configuration > Datasources and add a new Prometheus instance.




<img width="1679" alt="image" src="https://user-images.githubusercontent.com/90141704/181879253-d08f2e44-3d4a-4d8a-85d1-341132f52c0d.png">





# Setup Grafana/Loki on Local K8s Cluster — Minikube


What Is Loki?
Loki is a horizontally-scalable, highly-available, multi-tenant log aggregation system inspired by Prometheus. It is designed to be very cost effective and easy to operate. It does not index the contents of the logs, but rather a set of labels for each log stream.

**Install Loki on Minikube
Add Helm Repo for Loki**

$ helm repo add loki https://grafana.github.io/loki/charts

"loki" has been added to your repositories

Then, update the repos.

$ helm repo update

Before installing Loki, let’s have a check for the correct repo of it.


$ helm search repo loki


NAME                     CHART VERSION APP VERSION DESCRIPTION


grafana/loki             2.5.0         v2.2.0      Loki: like Prometheus, but for logs.


grafana/loki-canary      0.3.0         2.2.0       Helm chart for Grafana Loki Canary


grafana/loki-distributed 0.31.3        2.2.1       Helm chart for Grafana Loki in microservices mode


grafana/loki-stack       2.4.1         v2.1.0      Loki: like Prometheus, but for logs.


loki/loki                2.1.1         v2.0.0      DEPRECATED Loki: like Prometheus, but for logs.


loki/loki-stack          2.1.2         v2.0.0      DEPRECATED Loki: like Prometheus, but for logs.


loki/fluent-bit          2.0.2         v2.0.0      DEPRECATED Uses fluent-bit Loki go plugin for g...

loki/promtail            2.0.2         v2.0.0      DEPRECATED Responsible for gathering logs and s...

grafana/fluent-bit       2.3.0         v2.1.0      Uses fluent-bit Loki go plugin for gathering lo...

grafana/promtail         3.5.1         2.2.1       Promtail is an agent which ships the contents o...




**Install Loki**
I created a dedicated namespace monitoring in the Minikube cluster for Loki and Grafana.


$ kubectl create namespace monitoring
namespace/monitoring created


$ helm upgrade --install loki --namespace=monitoring grafana/loki-stack

Release "loki" does not exist. Installing it now.
NAME: loki
LAST DEPLOYED: Mon May 31 22:13:19 2021
NAMESPACE: monitoring
STATUS: deployed
REVISION: 1
NOTES:
The Loki stack has been deployed to your cluster. Loki can now be added as a datasource in Grafana.


**Verify Loki Installed**
Loki is installed as a StatefulSet and it could be checked as:

$ kubectl get statefulset -A
NAMESPACE    NAME   READY   AGE
monitoring   loki   1/1     49m


While, loki-promtail is a DaemonSet:

$ kubectl get daemonset -A
NAMESPACE     NAME            DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
monitoring    loki-promtail   1         1         1       1            1           <none>                   49m
  
  


<img width="1499" alt="image" src="https://user-images.githubusercontent.com/90141704/181879514-c9ef8af0-514e-42be-afb1-1b4fcd68459f.png">

  
  
  
  <img width="1599" alt="image" src="https://user-images.githubusercontent.com/90141704/181879530-4f97e4fa-ebe9-4d1a-bba0-f7f420a6efbe.png">
  
  
  
  <img width="1792" alt="image" src="https://user-images.githubusercontent.com/90141704/181879707-37a32932-3ceb-44f1-917e-ac48c200874c.png">

  
  
  
  <img width="1738" alt="image" src="https://user-images.githubusercontent.com/90141704/181888193-1435056c-ac5a-463f-8a77-34cd96540181.png">


  
  
  
**  Install Tempo On kuberntes:**
  
 # 
  **helm install my-tempo \
  --set tempo.traces.jaeger.grpc=true \
  bitnami/grafana-tempo**
  
  **kubectl expose service my-tempo-grafana-tempo-query-frontend --type=NodePort --target-port=3100 --name=my-tempo-grafana-tempo-query-frontend1**
  
  
  
  <img width="1236" alt="image" src="https://user-images.githubusercontent.com/90141704/181879618-ada0986a-9f9a-4dbb-864a-dd2158fa57a6.png">

  
  
  






