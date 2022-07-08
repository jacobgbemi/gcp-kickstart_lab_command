### Task 1: Confirm that needed APIs are enabled
1. confirm that both of these APIs are enabled, if not enable them:
	```
	Kubernetes Engine API
	Container Registry API
	```

### Task 2: Start a Kubernetes Engine cluster
1. place the zone into an environment variable called ```MY_ZONE```
	```
	export MY_ZONE=us-central1-a
	```
2. Start a Kubernetes cluster managed by Kubernetes Engine. Name the cluster webfrontend and configure it to run 2 nodes:
	```
	gcloud container clusters create webfrontend --zone $MY_ZONE --num-nodes 2
	```
3. After the cluster is created, check your installed version of Kubernetes using the kubectl version command:
	```
	kubectl version
	```
4. View your running nodes in the GCP Console
	```click Compute Engine > VM Instances.```

### Task 3: Run and deploy a container
1. From your Cloud Shell prompt, launch a single instance of the nginx container
	```
	kubectl create deploy nginx --image=nginx:1.17.10
	```
2. View the pod running the nginx container:
	```
	kubectl get pods
	```
3. Expose the nginx container to the Internet:
	```
	kubectl expose deployment nginx --port 80 --type LoadBalancer
	```
4. View the new service:
	```
	kubectl get services
	```
5. Open a new web browser tab and paste your cluster's external IP address into the address bar.
6. Scale up the number of pods running on your service:
	```
	kubectl scale deployment nginx --replicas 3
	```
7. Confirm that Kubernetes has updated the number of pods:
	```
	kubectl get pods
	```
8. Confirm that your external IP address has not changed:
	```
	kubectl get services
	```

