### Task 1: Setup
1. Set zone
	```
	gcloud config set compute/zone us-central1-a
	```
2. Get the sample code for creating and running containers and deployments:
	```
	gsutil -m cp -r gs://spls/gsp053/orchestrate-with-kubernetes .
	cd orchestrate-with-kubernetes/kubernetes
	```
3. Create a cluster with five n1-standard-1 nodes (this will take a few minutes to complete):
	```
	gcloud container clusters create bootcamp --num-nodes 5 --scopes "https://www.googleapis.com/auth/projecthosting,storage-rw"
	```

Task 2: Learn about the deployment object
1. The explain command in kubectl can tell us about the Deployment object.
	```
	kubectl explain deployment
	```
2. We can also see all of the fields using the --recursive option.
	```
	kubectl explain deployment --recursive
	```
3. You can use the explain command as you go through the lab to help you understand the structure of a Deployment object and understand what the individual fields do.
	```
	kubectl explain deployment.metadata.name
	```

Task 3: Create a deployment
1. Update the deployments/auth.yaml configuration file:
	```
	vi deployments/auth.yaml
	```
2. Start the editor:
	```
	i
	```
3. Change the image in the containers section of the Deployment to the following:
	```
	containers:
	- name: auth
	  image: "kelseyhightower/auth:1.0.0"
	...
4. Save the auth.yaml file: press <Esc> then type:
	```
	:wq
	```
5. Press <Enter>. Now let's create a simple deployment. Examine the deployment configuration file:
	```
	cat deployments/auth.yaml
	```
6. Go ahead and create your deployment object using kubectl create:
	```
	kubectl create -f deployments/auth.yaml
	```
7. Once you have created the Deployment, you can verify that it was created.
	```
	kubectl get deployments
	```
8. Verify that a ReplicaSet was created for the Deployment:
	kubectl get replicasets
9. The single Pod is created by the Kubernetes when the ReplicaSet is created.
	kubectl get pods
10. Use the kubectl create command to create the auth service.
	kubectl create -f services/auth.yaml
11. Now, do the same thing to create and expose the hello Deployment.
	kubectl create -f deployments/hello.yaml
	kubectl create -f services/hello.yaml
12. And one more time to create and expose the frontend Deployment.
	kubectl create secret generic tls-certs --from-file tls/
	kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf
	kubectl create -f deployments/frontend.yaml
	kubectl create -f services/frontend.yaml
13. Interact with the frontend by grabbing its external IP and then curling to it.
	kubectl get services frontend
14. To get the output
	curl -ks https://<EXTERNAL-IP>
15. You can also use the output templating feature of kubectl to use curl as a one-liner:
	curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`

Task 4: Scale a Deployment
1. look at an explanation of this field using the kubectl explain command again.
	kubectl explain deployment.spec.replicas
2. The replicas field can be most easily updated using the kubectl scale command:
	kubectl scale deployment hello --replicas=5
3. Verify that there are now 5 hello Pods running:
	kubectl get pods | grep hello- | wc -l
4. Now scale back the application:
	kubectl scale deployment hello --replicas=3
5. Again, verify that you have the correct number of Pods
	kubectl get pods | grep hello- | wc -l

Task 5: Rolling update

Trigger a rolling update
1. To update your Deployment, run the following command:
	kubectl edit deployment hello
2. Change the image in the containers section of the Deployment to the following:
	...
containers:
  image: kelseyhightower/hello:2.0.0
...
3. See the new ReplicaSet that Kubernetes creates.:
	kubectl get replicaset
4. You can also see a new entry in the rollout history:
	kubectl rollout history deployment/hello

Pause a rolling update
5. If you detect problems with a running rollout, pause it to stop the update. Give that a try now:
	kubectl rollout pause deployment/hello
6. Verify the current state of the rollout:
	kubectl rollout status deployment/hello
7. You can also verify this on the Pods directly:
	kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'

Resume a rolling update
8. We can continue the rollout using the resume command.
	kubectl rollout resume deployment/hello
9. When the rollout is complete, you should see the following when running the status command.
	kubectl rollout status deployment/hello

Rollback an update
10. Use the rollout command to roll back to the previous version:
	kubectl rollout undo deployment/hello
11. Verify the roll back in the history:
	kubectl rollout history deployment/hello
12. Finally, verify that all the Pods have rolled back to their previous versions:
	kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'

Task 6: Canary deployments

Create a canary deployment
1. First, create a new canary deployment for the new version:
	cat deployments/hello-canary.yaml
2. Now create the canary deployment:
	kubectl create -f deployments/hello-canary.yaml
3. After the canary deployment is created, you should have two deployments, hello and hello-canary. Verify it with this kubectl command:
	kubectl get deployments

Verify the canary deployment
4. You can verify the hello version being served by the request:
	curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version

The service
5. First update the service:
	kubectl apply -f services/hello-blue.yaml

Updating using Blue-Green Deployment
6. Create the green deployment:
	kubectl create -f deployments/hello-green.yaml
7. Once you have a green deployment and it has started up properly, verify that the current version of 1.0.0 is still being used:
	curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
8. Now, update the service to point to the new version:
	kubectl apply -f services/hello-green.yaml
9. When the service is updated, the "green" deployment will be used immediately. You can now verify that the new version is always being used.
	curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version

Blue-Green Rollback
10. While the "blue" deployment is still running, just update the service back to the old version.
	kubectl apply -f services/hello-blue.yaml
11. Again, verify that the right version is now being used:
	curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
