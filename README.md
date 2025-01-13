# how-to-setup-prometheus-and-grafana-in-k8s
Guide how to set up Prometheus and Grafana for Kubernetes cluster monitoring 

### Prerequisites
- Kubernetes cluster
- Helm

### Install Kubernetes cluster
You can use Minikube for local development or any cloud provider like GCP, AWS, Azure, etc.

I'm using kind (Kubernetes in Docker) for local development.

1. Pre-requisites:
- Docker
- kubectl
- Go

2. Install kind
```bash
go install sigs.k8s.io/kind@v0.26.0
```

3. Create a cluster
```bash
kind create cluster
```

4. Check the cluster
```bash
kubectl get pods -A
```

### Install Helm
1. Download Helm
```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
```

2. Install Helm
```bash
chmod 700 get_helm.sh
./get_helm.sh
```

3. Check Helm version
```bash
helm version
```

### Install Prometheus
1. Add Prometheus Helm repository
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

2. Update Helm repositories
```bash
helm repo update
```

3. Install Prometheus
```bash
helm install prometheus prometheus-community/prometheus
```

4. Check Prometheus services
```bash
kubectl get svc
```
> Note: prometheus-server is the core monitoring and alerting engine, but kube-state-metrics provides kubernetes-specific metrics.

5. Expose prometheus-server service 
```bash
kubectl expose service prometheus-server --type=NodePort --target-port=9090 --name=prometheus-server-ext
```

6. Check Prometheus service
```bash
kubectl get svc
```
> Note: You can now see a new entry for prometheus-server-ext service with a NodePort.

7. Forward the port to access the Prometheus dashboard (only if you're having troubles connecting to the nodeport, all the clusters set on containers like kind can show some problems)
```bash
kubectl port-forward svc/prometheus-server-ext 9090:80
```
> Note: With minikube, you dont need to forward the port. You can access the dashboard directly using the `minikube ip` and the NodePort (if you are not using Docker Desktop).

8. Open Prometheus dashboard in the browser
```bash
http://localhost:9090
```
> Note: On production environments, you will be using a LoadBalancer or Ingress to expose the service.


### Install Grafana
1. Add Grafana Helm repository
```bash
helm repo add grafana https://grafana.github.io/helm-charts
```

2. Update Helm repositories
```bash
helm repo update
```

3. Install Grafana
```bash
helm install grafana grafana/grafana
```

4. Create your admin password
```bash
kubectl get secret --namespace default grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

5. Check Grafana services
```bash
kubectl get svc
```

6. Expose Grafana service
```bash
kubectl expose service grafana --type=NodePort --target-port=3000 --name=grafana-ext
```

7. Check Grafana service
```bash
kubectl get svc
```

8. Forward the port to access the Grafana dashboard
```bash
kubectl port-forward svc/grafana-ext 3000:80
```

9. Open Grafana dashboard in the browser
```bash
http://localhost:3000
```

10. Login with the admin user and the password you created in step 4.

11. Add Prometheus as a data source
- Click on the gear icon on the left side menu
- Click on Data Sources
- Click on Add data source
- Select Prometheus
- Set the URL to http://prometheus-server-ext
- Click on Save & Test

12. Import a dashboard
- Click on the "+" icon on the left side menu
- Click on Import
- Enter the dashboard ID (3662 is a good one)
- Select the Prometheus data source
- Click on Import

13. Check the dashboard

### Clean up
#### Delete Kubernetes cluster
```bash
kind delete cluster
```



