Step 1 – Setup EC2 Instance
Instance Type: t2.medium
AMI: Ubuntu US-EAST-1

Step 1.1 – Create the IAM role having full access

Go to IAM -> Create role
Select EC2 -> Give Full admin access

Step 1.2 – Attach the IAM role having full access

Go to EC2 -> Click on Actions on the left-hand side
Navigate to Security -> Modify IAM role

Step 2 – Install AWS CLI and Configure

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
/usr/local/bin/aws --version

Step 3 – Install and Setup Kubectl

curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short

(Repeat Step 3 as it seems to have been duplicated)

Step 4 – Install and Setup eksctl


curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
Step 5 – Install Helm chart

bash
Copy code
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
Step 6 – Create EKS Cluster

bash
Copy code
eksctl create cluster --name eks4 --version 1.24 --region us-east-1 --zones=us-east-1b,us-east-1c --nodegroup-name worker-nodes --node-type t2.medium --nodes 2 --nodes-min 2 --nodes-max 3
aws eks update-kubeconfig --region us-east-1 --name eks5
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
Continue with the subsequent steps, following the detailed instructions for Prometheus and Grafana setup. Don't forget to verify and validate each step before proceeding to the next one.

kubectl get deployment metrics-server -n kube-system
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
kubectl create namespace prometheus
helm install prometheus prometheus-community/prometheus --namespace prometheus --set alertmanager.persistentVolume.storageClass="gp2" --set server.persistentVolume.storageClass="gp2"


oidc_id=$(awseks describe-cluster --name eks4 --region us-east-1 --query "cluster.identity.oidc.issuer" --output text | cut -d'/' -f5)
aws iam list-open-id-connect-providers | grep $oidc_id | cut -d"/" -f4
eksctl utils associate-iam-oidc-provider --cluster eks4 --approve --region us-east-1


Step 5 – Create IAM Service Account with Role

eksctl create iamserviceaccount --name ebs-csi-controller-sa --namespace kube-system --cluster eks5 --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy --approve --role-only --role-name AmazonEKS_EBS_CSI_DriverRole --region us-east-1

Step 5.1 – Attach Role to EKS Cluster

eksctl create addon --name aws-ebs-csi-driver --cluster eks5 --service-account-role-arn arn:aws:iam::152984724345:role/AmazonEKS_EBS_CSI_DriverRole --force --region us-east-1


Step 5.2 – Verify EBS CSI Driver Pods

kubectl get pods -n prometheus

Step 6 – Forward Prometheus Dashboard Ports

kubectl port-forward deployment/prometheus-server 9090:9090 -n prometheus

Step 6.1 – Open Prometheus Dashboard in Browser

curl localhost:9090/graph

Step 7 – Install Grafana

helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

kubectl create namespace grafana
helm install grafana grafana/grafana --namespace grafana --set persistence.storageClassName="gp2" --set persistence.enabled=true --set adminPassword='EKS!sAWSome' --set service.type=LoadBalancer

Step 11.1 – Verify Grafana Installation

kubectl get pods -n grafana
kubectl get service -n grafana

Step 11.2 – Open Grafana in Browser

kubectl get service -n grafana

# Copy the EXTERNAL-IP and paste it in the 

Step 11.3 – Add Prometheus as the Datasource to Grafana

Go to Grafana Dashboard -> Add the Datasource -> Select Prometheus

Step 11.4 – Configure the endpoints of Prometheus and Save

URL: http://prometheus-server.prometheus.svc.cluster.local

Step 11.5 – Import Grafana Dashboard from Grafana Labs

Go to left side -> Click on Dashboards -> Click on New -> Import

Load and select the source as Prometheus

Step 12 – Visualize the Java Application

Step 13 – Deploy a Java Application and Monitor it on Grafana

git clone https://github.com/praveen1994dec/kubernetes_java_deployment.git
cd kubernetes_java_deployment/Kubernetes/
kubectl apply -f shopfront-service.yaml
kubectl get deployment
kubectl get pods
kubectl logs shopfront-7868468c56-4r2kk-cshopfront

Step 14 – Clean Up EKS Cluster

eksctl delete cluster --name eks4 --region us-east-1

These steps should provide a comprehensive guide for setting up, monitoring, and visualizing your Java application on EKS using Prometheus and Grafana.
