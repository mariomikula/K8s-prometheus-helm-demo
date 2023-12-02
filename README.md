# K8s-prometheus-helm-demo

This repo was created to practise deploying Prometheus Stack in a Kubernetes cluster using Helm.

It follows steps from thess video tutorials:

https://youtu.be/QoDqxm7ybLc

https://youtu.be/mLPg49b33sA

## How to start

### Prereqs

These need to be installed to start the cluster locally:
- docker (https://www.docker.com/)
- minikube (https://minikube.sigs.k8s.io/docs/start/)
- kubectl (https://kubernetes.io/docs/reference/kubectl/)
- helm (https://helm.sh/docs/intro/install/)

### Steps

1. Start minikube by running
    ```
    minikube start
    ```
    Wait for it to complete and run
    ```
    minikube status
    ```
    to check if it works. You should get something like
    ```
    minikube
    type: Control Plane
    host: Running
    kubelet: Running
    apiserver: Running
    kubeconfig: Configured
    ```
2. Install Prometheus
   ```
   helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
   helm repo update
   helm install prometheus prometheus-community/kube-prometheus-stack
   ```
   You should see something like
   ```
   NAME: prometheus
   LAST DEPLOYED: Sat Dec  2 17:05:28 2023
   NAMESPACE: default
   STATUS: deployed
   REVISION: 1
   NOTES:
   kube-prometheus-stack has been installed. Check its status by running:
     kubectl --namespace default get pods -l "release=prometheus"
   
   Visit https://github.com/prometheus-operator/kube-prometheus for instructions on how to create & configure Alertmanager and Prometheus instances using the Operator.
   ```
   You can see what was installed by running
   ```
   helm ls
   kubectl get all
   kubectl get configmap
   kubectl get crd
   ```
4. To connect to Grafana to test it do this. Start port-forwarding (below). Connect via browser to `localhost:3000` using default credentials defined by the chart `admin:prom-operator`.
   ```
   kubectl port-forward deployment/prometheus-grafana 3000
   ```
5. To access Prometheus UI on it's pod do somethis similar
   ```
   kubectl port-forward prometheus-prometheus-kube-prometheus-prometheus-0 9090
   ```
6. Create MongoDB deployment and service
   ```
   kubectl apply -f mongodb.yaml
   ```
   Check that everything is ok by running and looking and mongodb related pod, service, deployment and replicaset
   ```
   kubectl get all
   ```
7. Install Prometheus MongoDB exporter.
   First go to https://github.com/prometheus-community/helm-charts and find the chart for MongoDB exporter. At the time of writing it was `prometheus-mongodb-exporter`.
   Secondly add the repo if it has not been added yet by running
   ```
   helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
   ```
   There is `prometheus-mongodb-exporter-values.yaml` overriding some values for the exporter. In particular MongoDB URI and ServiceMonitor creation. Then install the exporter by running
   ```
   helm install mongodb-exporter prometheus-community/prometheus-mongodb-exporter -f prometheus-mongodb-exporter-values.yaml
   ```
   Check it by running
   ```
   helm ls
   kubectl get pod
   kubectl get service
   kubectl get servicemonitor
8. See the exported metrics by enabling port-forwaring to the exporter service and connecting to is via browser `localhost:9216`
   ```
   kubectl port-forward service/mongodb-exporter-prometheus-mongodb-exporter 9216
   ```
9. Go to the Prometheus UI again (run the port-forwarding from step 5 again) and MongoDB Exporter should be there as Prometheus should have added it by itself using the ServiceMonitor component.
