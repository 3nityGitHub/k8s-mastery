# Talium Production Kubernetes Stack

This README explains the full Kubernetes stack for the Talium application.  
It is written so that **a new engineer can understand exactly what each file does and how to deploy the entire system from scratch**.


##  What This Stack Contains

This folder includes everything required to run the Talium backend in Kubernetes:

- **Namespace** — isolates all Talium resources  
- **ConfigMap** — non sensitive configuration (DB host, Redis host, ports, etc.)  
- **Secret** — sensitive configuration (passwords, tokens)  
- **PVCs** — persistent storage for PostgreSQL and Redis  
- **Deployments** — PostgreSQL, Redis, and Flask API workloads  
- **Services** — stable networking for each component  
- **Ingress** — exposes the API externally at `http://talium.local:8080`

All manifests are numbered so they can be applied in the correct dependency order.


##  Prerequisites

Before deploying, ensure you have:

- A running Kubernetes cluster (minikube, kind, or cloud)
- `kubectl` installed and configured
- An Ingress controller (e.g., nginx ingress)
- A hosts entry so your machine can resolve `talium.local`

For minikube:

```bash
echo "$(minikube ip) talium.local" | sudo tee -a /etc/hosts
This maps the hostname to your cluster’s ingress IP.

How to Deploy the Stack
From inside the talium-production directory:
kubectl apply -f ./
Watch the pods come online:
kubectl get pods -n talium --watch
You should wait until all pods show:
1/1 Running
If anything is stuck, inspect it:
kubectl describe pod <pod-name> -n talium
kubectl logs <pod-name> -n talium

How to Verify the Stack Is Working
1. Check Kubernetes resources
kubectl get pods,svc,ingress -n talium
You should see:
•	All pods running
•	Services with valid endpoints
•	An Ingress pointing to the API service
2. Test the API endpoints
curl http://talium.local:8080/
curl http://talium.local:8080/patients
You should receive valid JSON responses. If you get a 500 error, check the API logs:
kubectl logs -l app=talium-api -n talium

How to Seed the Database
Once PostgreSQL is running, you can seed it using a temporary client pod:
kubectl run psql-client -n talium --rm -it --image=postgres:15 -- bash
psql -h db -U admin -d talium
Inside the psql shell, run your schema and seed SQL.
If your API performs migrations automatically, simply hitting the API root endpoint may trigger them.

How to Tear Down the Stack
To remove everything:
kubectl delete -f ./
This deletes:
•	Deployments
•	Services
•	PVCs (depending on reclaim policy)
•	ConfigMap and Secret
•	The entire talium namespace
If you want to preserve database data, delete resources individually instead of deleting the namespace.

