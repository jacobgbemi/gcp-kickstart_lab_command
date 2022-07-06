Task 1. Set a default compute zone
gcloud config set compute/zone us-central1-a

Task 2. Create a GKE cluster
1. To create a cluster
	gcloud container clusters create [CLUSTER-NAME]

Task 3. Get authentication credentials for the cluster
1. To authenticate the cluster,
	gcloud container clusters get-credentials [CLUSTER-NAME]

Task 4. Deploy an application to the cluster
1. To create a new Deployment hello-server from the hello-app container image,
  kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0
2. To create a Kubernetes Service, which is a Kubernetes resource that lets you expose your application to external traffic,
  kubectl expose deployment hello-server --type=LoadBalancer --port 8080
3. To inspect the hello-server
  kubectl get service
4. To view the application from your web browser, open a new tab and enter
	http://[EXTERNAL-IP]:8080

Task 5. Deleting the cluster
1. To delete the cluster,
	gcloud container clusters delete [CLUSTER-NAME]
2. When prompted, type Y to confirm.
