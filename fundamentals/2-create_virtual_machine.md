Task 1. Create a new instance from the Cloud Console

Task 2. Install an NGINX web server
1. In the SSH terminal, to get root access, run the following command:
	sudo su -
2. As the root user, update your OS:
	apt-get update
3. Install NGINX:
	apt-get install nginx -y
4. Confirm that NGINX is running:
	ps auwx | grep nginx
5. To see the web page, add the External IP value to http://EXTERNAL_IP/ in a new browser window or tab.

Task 3. Create a new instance with gcloud
1. In the Cloud Shell, use gcloud to create a new virtual machine instance from the command line:
	gcloud compute instances create gcelab2 --machine-type n1-standard-2 --zone us-central1-f
2. To see all the defaults, run:
	gcloud compute instances create --help
3. To exit help, press CTRL + C.
4. You can also use SSH to connect to your instance via gcloud. 
	gcloud compute ssh gcelab2 --zone us-central1-f
5. Type Y, and press ENTER
6. After connecting, disconnect from SSH by exiting from the remote shell:
	exit
