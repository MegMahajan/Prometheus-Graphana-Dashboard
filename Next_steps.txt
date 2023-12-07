kubectl create namespace grafana
helm install grafana grafana/grafana --namespace grafana --set persistence.storageClassName="gp2" --set persistence.enabled=true --set adminPassword='EKS!sAWSome' --set service.type=LoadBalancer
Step 11.1 – Verify Grafana Installation

bash
Copy code
kubectl get pods -n grafana
kubectl get service -n grafana
Step 11.2 – Open Grafana in Browser

bash
Copy code
kubectl get service -n grafana
# Copy the EXTERNAL-IP and paste it in the browser
Step 11.3 – Add Prometheus as the Datasource to Grafana

Go to Grafana Dashboard -> Add the Datasource -> Select Prometheus
Step 11.4 – Configure the endpoints of Prometheus and Save

URL: http://prometheus-server.prometheus.svc.cluster.local
Step 11.5 – Import Grafana Dashboard from Grafana Labs

Go to left side -> Click on Dashboards -> Click on New -> Import
Load and select the source as Prometheus
Step 12 – Visualize the Java Application

Step 13 – Deploy a Java Application and Monitor it on Grafana

bash
Copy code
git clone https://github.com/praveen1994dec/kubernetes_java_deployment.git
cd kubernetes_java_deployment/Kubernetes/
kubectl apply -f shopfront-service.yaml
kubectl get deployment
kubectl get pods
kubectl logs shopfront-7868468c56-4r2kk-cshopfront
Step 14 – Clean Up EKS Cluster

bash
Copy code
eksctl delete cluster --name eks2 --region us-east-1
These steps should provide a comprehensive guide for setting up, monitoring, and visualizing your Java application on EKS using Prometheus and Grafana.
