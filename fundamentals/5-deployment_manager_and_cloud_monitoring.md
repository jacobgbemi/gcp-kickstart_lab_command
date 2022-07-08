### Task 1: Confirm that needed APIs are enabled
1. Confirm that these three APIs are enabled, if not enable them
	```
	Cloud Deployment Manager v2 API
	Cloud Runtime Configuration API
	Stackdriver monitoring API
	```

### Task 2: Create a Deployment Manager deployment
1. Place the zone into an environment variable called MY_ZONE
	```
	export MY_ZONE=us-central1-a
	```
2. At the Cloud Shell prompt, download an editable Deployment Manager template:
	```
	gsutil cp gs://cloud-training/gcpfcoreinfra/mydeploy.yaml mydeploy.yaml
	```
3. replace the PROJECT_ID placeholder string with your Google Cloud Platform project ID
	```
	sed -i -e "s/PROJECT_ID/$DEVSHELL_PROJECT_ID/" mydeploy.yaml
	```
4. replace the ZONE placeholder string with your Google Cloud Platform zone
	```
	sed -i -e "s/ZONE/$MY_ZONE/" mydeploy.yaml
	```
5. View the mydeploy.yaml file,
	```
	cat mydeploy.yaml
	```
6. Build a deployment from the template:
	```
	gcloud deployment-manager deployments create my-first-depl --config mydeploy.yaml
	```
### Task 3: Update a Deployment Manager deployment
1. Launch the nano text editor to edit the mydeploy.yaml file:
	```
	nano mydeploy.yaml
	```
2. Find the line that sets the value of the startup script, value: "apt-get update", and edit it so that it looks like this:
	```
	value: "apt-get update; apt-get install nginx-light -y"
	```
3. Press ```Ctrl+O``` and then press ```Enter``` to save your edited file.
4. Press ```Ctrl+X``` to exit the nano text editor.
5. Deployment Manager to update your deployment to install the new startup script:
	```
	gcloud deployment-manager deployments update my-first-depl --config mydeploy.yaml
	```

### Task 4: View the Load on a VM using Cloud Monitoring
1. To open a command prompt on the my-vm instance, click SSH in its row in the VM instances list.
2. In the ssh session on my-vm, execute this command to create a CPU load:
	```
	dd if=/dev/urandom | gzip -9 >> /dev/null &
	```
	### Create a Monitoring workspace
3. In the Google Cloud Platform Console, click on Navigation menu > Monitoring.
4. Click on Settings option from the left panel and confirm that the GCP project which Qwiklabs created for you is shown under the GCP Projects section.
5. Run the commands shown on screen in the SSH window of your VM instance to install both the Monitoring and Logging agents.
	```
	curl -sSO https://dl.google.com/cloudagents/install-monitoring-agent.sh
	sudo bash install-monitoring-agent.sh
	curl -sSO https://dl.google.com/cloudagents/install-logging-agent.sh
	sudo bash install-logging-agent.sh
	```
6. Once both of the agents have been installed on your project's VM, click Metrics Explorer under the main Cloud Monitoring menu on the far left.
7. In the Metrics Explorer pane, Click on ```Select a Metric``` dropdown under Resource & Metric and then select ```VM instance > Instance > CPU usage```.
8. Click ```Apply```.
9. Terminate your workload generator. Return to your ssh session on my-vm and enter this command:
	```
	kill %1
	```
