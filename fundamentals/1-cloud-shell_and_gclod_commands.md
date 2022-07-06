## Task 1. Configure your environment
1. Set the region to us-central1:
	```
	gcloud config set compute/region us-central1
	```
2. To view the project region setting, run the following command:
	```
	gcloud config get-value compute/region
	```
3. Set the zone to us-central1-b:
	```
	gcloud config set compute/zone us-central1-b
	```
4. To view the project zone setting, run the following command:
	```
	gcloud config get-value compute/zone
	```
### Finding project information
5. In Cloud Shell, run the following gcloud command, to view the project id for your project:
	```
	gcloud config get-value project
	```
6. In Cloud Shell, run the following gcloud command to view details about the project:
	```
	gcloud compute project-info describe --project $(gcloud config get-value project)
	```
### Setting environment variables
7. Create an environment variable to store your Project ID
	```
	export PROJECT_ID=$(gcloud config get-value project)
	```
8. Create an environment variable to store your Zone
	```
	export ZONE=$(gcloud config get-value compute/zone)
	```
9. To verify that your variables were set properly, run the following commands:
	```
	echo -e "PROJECT ID: $PROJECT_ID\nZONE: $ZONE"
	```
### Creating a virtual machine with the gcloud tool
10. To create your VM, run the following command:
	```
	gcloud compute instances create gcelab2 --machine-type n1-standard-2 --zone $ZONE
	```
11. To open help for the create command, run the following command:
	```
	gcloud compute instances create --help
	```

### Explore gcloud commands
12. gcloud tool for simple usage guidelines
	```
	gcloud -h
	gcloud config --help
	gcloud help config
	```
13. View the list of configurations in your environment:
	```
	gcloud config list
	```
14. To see all properties and their settings:
	```
	gcloud config list --all
	```
15. List your components:
	```
	gcloud components list
	```
## Task 2. Filtering command line output
1. List the compute instance available in the project:
	```
	gcloud compute instances list
	```
2. List the gcelab2 virtual machine:
	```
	gcloud compute instances list --filter="name=('gcelab2')"
	```
3. List the Firewall rules in the project:
	```
	gcloud compute firewall-rules list
	```
4. List the Firewall rules for the default network:
	```
	gcloud compute firewall-rules list --filter="network='default'"
	```
5. List the Firewall rules for the default network where the allow rule matches an ICMP rule:
	```
	gcloud compute firewall-rules list --filter="NETWORK:'default' AND ALLOW:'icmp'"
	```
## Task 3. Connecting to your VM instance
1. To connect to your VM with SSH, run the following command:
	```
	gcloud compute ssh gcelab2 --zone $ZONE
	```
2. To continue, type ```Y```.
3. To leave the passphrase empty, press ```ENTER``` twice.
4. Install ```nginx web server``` on to virtual machine:
	```
	sudo apt install -y nginx
	```
5. To disconnect from SSH and exit the remote shell, run the following command:
	```
	exit
	```
## Task 4. Updating the Firewall
1. List the firewall rules for the project:
	```
	gcloud compute firewall-rules list
	```
2. Add a tag to the virtual machine:
	```
	gcloud compute instances add-tags gcelab2 --tags http-server,https-server
	```
3. Update the firewall rule to allow:
	```
	gcloud compute firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server
	```
4. List the firewall rules for the project:
	```
	gcloud compute firewall-rules list --filter=ALLOW:'80'
	```
5. Verify communication is possible for http to the virtual machine:
	```
	curl http://$(gcloud compute instances list --filter=name:gcelab2 --format='value(EXTERNAL_IP)')
	```
## Task 5. Viewing the system logs
1. View the available logs on the system:
	```
	gcloud logging logs list 
	```
2. View the logs that relate to compute resources:
	```
	gcloud logging logs list --filter="compute" 
	```
3. Read the logs related to the resource type of gce_instance:
	```
	gcloud logging read "resource.type=gce_instance" --limit 5
	```
4. Read the logs for a specific virtual machine:
	```
	gcloud logging read "resource.type=gce_instance AND labels.instance_name='gcelab2'" --limit 5
	```
