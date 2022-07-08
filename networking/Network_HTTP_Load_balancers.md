### Task 1: Set the default region and zone for all resources
1. In Cloud Shell, set the default zone:
	```
	gcloud config set compute/zone us-central1-a
	```
2. Set the default region:
	```
	gcloud config set compute/region us-central1
	```

### Task 2: Create multiple web server instances
1. Create virtual machines in your default zone
	```
	gcloud compute instances create www1 \
	  --image-family debian-9 \
	  --image-project debian-cloud \
	  --zone us-central1-a \
	  --tags network-lb-tag \
	  --metadata startup-script="#! /bin/bash
	    sudo apt-get update
	    sudo apt-get install apache2 -y
	    sudo service apache2 restart
	    echo '<!doctype html><html><body><h1>www1</h1></body></html>' | tee /var/www/html/index.html"
	 ```
2. Create a firewall rule to allow external traffic to the VM instances:
	```
	gcloud compute firewall-rules create www-firewall-network-lb \
    	--target-tags network-lb-tag --allow tcp:80
        ```
3. Run the following to list your instances
	```
	gcloud compute instances list
	```
4. Verify that each instance is running with curl, replacing [IP_ADDRESS] with the IP address for each of your VMs:
	```
	curl http://[IP_ADDRESS]
	```

### Task 3: Configure the load balancing service
1. Create a static external IP address for your load balancer:
	```
	gcloud compute addresses create network-lb-ip-1 \
 	--region us-central1
 	```
2. Add a legacy HTTP health check resource:
	```
	gcloud compute http-health-checks create basic-check
	```
3. Add a target pool in the same region as your instances.
	```gcloud compute target-pools create www-pool \
    	--region us-central1 --http-health-check basic-check
	```
4. Add the instances to the pool:
	```gcloud compute target-pools add-instances www-pool \
    	--instances www1,www2,www3
	```
5. Add a forwarding rule:
	```
	gcloud compute forwarding-rules create www-rule \
	    --region us-central1 \
	    --ports 80 \
	    --address network-lb-ip-1 \
	    --target-pool www-pool
	```

### Task 4: Sending traffic to your instances
1. Enter the following command to view the external IP address of the www-rule forwarding rule used by the load balancer:
	```
	gcloud compute forwarding-rules describe www-rule --region us-central1
	```

### Task 5: Create an HTTP load balancer
1. First, create the load balancer template:
	```
	gcloud compute instance-templates create lb-backend-template \
	   --region=us-central1 \
	   --network=default \
	   --subnet=default \
	   --tags=allow-health-check \
	   --image-family=debian-9 \
	   --image-project=debian-cloud \
	   --metadata=startup-script='#! /bin/bash
	     apt-get update
	     apt-get install -y nginx
	     service nginx start
	     sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
	```
2. Create a managed instance group based on the template:
	```
	gcloud compute instance-groups managed create lb-backend-group \
   	--template=lb-backend-template --size=2 --zone=us-central1-a
	```
3. Create the fw-allow-health-check firewall rule
	```
	gcloud compute firewall-rules create fw-allow-health-check \
	    --network=default \
	    --action=allow \
	    --direction=ingress \
	    --source-ranges=130.211.0.0/22,35.191.0.0/16 \
	    --target-tags=allow-health-check \
	    --rules=tcp:80
    	```
4. set up a global static external IP address that your customers use to reach your load balancer.
	```
	gcloud compute addresses create lb-ipv4-1 \
	    --ip-version=IPV4 \
	    --global
    	```
5. Create a health check for the load balancer:
	```
	gcloud compute health-checks create http http-basic-check \
    	--port 80
    	```
6. Create a backend service:
	```
	gcloud compute backend-services create web-backend-service \
	    --protocol=HTTP \
	    --port-name=http \
	    --health-checks=http-basic-check \
	    --global
    	```
7. Add your instance group as the backend to the backend service:
	```
	gcloud compute backend-services add-backend web-backend-service \
	    --instance-group=lb-backend-group \
	    --instance-group-zone=us-central1-a \
	    --global
    	```
8. Create a URL map to route the incoming requests to the default backend service:
	```
	gcloud compute url-maps create web-map-http \
    	--default-service web-backend-service
    	```
9. Create a target HTTP proxy to route requests to your URL map:
	```
	gcloud compute target-http-proxies create http-lb-proxy \
    	--url-map web-map-http
	```
10. Create a global forwarding rule to route incoming requests to the proxy:
	```
	gcloud compute forwarding-rules create http-content-rule \
	    --address=lb-ipv4-1\
	    --global \
	    --target-http-proxy=http-lb-proxy \
	    --ports=80
    	```

