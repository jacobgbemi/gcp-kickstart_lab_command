Task 1: Working with backends
Add a local backend
1. Create your main.tf configuration file:
	touch main.tf
2. To retrieve your Project ID, run the following command:
	gcloud config list --format 'value(core.project)'
3. Copy the Cloud Storage bucket resource code into your main.tf configuration file
	provider "google" {
  project     = "# REPLACE WITH YOUR PROJECT ID"
  region      = "us-central-1"
}
resource "google_storage_bucket" "test-bucket-for-state" {
  name        = "# REPLACE WITH YOUR PROJECT ID"
  location    = "US"
  uniform_bucket_level_access = true
}
4. Add a local backend to your main.tf file:
	terraform {
  backend "local" {
    path = "terraform/state/terraform.tfstate"
  }
}
5. then initialize Terraform:
	terraform init
6. Apply the changes
	terraform apply
7. Examine your state file:
	terraform show

Add a Cloud Storage backend
9. Navigate back to your main.tf file
10. Copy the following configuration into your file, replacing the local backend:
	terraform {
  backend "gcs" {
    bucket  = "# REPLACE WITH YOUR BUCKET NAME"
    prefix  = "terraform/state"
  }
}
11. Initialize your backend again, this time to automatically migrate the state.
	terraform init -migrate-state
12. In the Cloud Console, in the Navigation menu, click Cloud Storage > Browser.
13. Click on your bucket and navigate to the file terraform/state/default.tfstate.

Refresh the state
14. Return to your storage bucket in the Cloud Console. Select the check box next to the name, and click Show info panel. 
15. Click the Labels tab.
16. Click Add Label. Set the Key 1 = key and Value 1 = value.
17. Click Save.
18. Return to Cloud Shell and use the following command to update the state file:
	terraform refresh
19. Examine the updates:
	terraform show

Clean up your workspace
20. First, revert your backend to local. Copy and replace the gcs configuration with the following:
	terraform {
  backend "local" {
    path = "terraform/state/terraform.tfstate"
  }
}
21. Initialize the local backend again.
	terraform init -migrate-state
22. In the main.tf file, add the force_destroy = true argument to your google_storage_bucket resource
	resource "google_storage_bucket" "test-bucket-for-state" {
  name        = "qwiklabs-gcp-03-c26136e27648"
  location    = "US"
  uniform_bucket_level_access = true
  force_destroy = true
}
23. Apply the changes:
	terraform apply
24. Destroy your infrastructure.
	terraform destroy

Task 2: Import Terraform configuration
Create a Docker container
1. Create a container named hashicorp-learn using the latest NGINX image from Docker Hub, and preview the container on the Cloud Shell virtual machine over port 80 (HTTP):
	docker run --name hashicorp-learn --detach --publish 8080:80 nginx:latest
2. Verify that the container is running:
	docker ps

Import the container into Terraform
3. Clone the example repository:
	git clone https://github.com/hashicorp/learn-terraform-import.git
4. Change into that directory:
	cd learn-terraform-import
5. Initialize your Terraform workspace:
	terraform init
6.  Navigate to learn-terraform-import/main.tf.	
7. Find the provider: docker resource and comment out or delete the host argument:
	provider "docker" {
#   host    = "npipe:////.//pipe//docker_engine"
}
8. Next, navigate to learn-terraform-import/docker.tf.
9. Under the commented-out code, define an empty docker_container resource in your docker.tf file, which represents a Docker container with the Terraform resource ID docker_container.web:
	resource "docker_container" "web" {}
10. Find the name of the container you want to import: in this case, the container you created in the previous step:
	docker ps
11. Run the following terraform import command to attach the existing Docker container to the docker_container.web resource you just created.
	terraform import docker_container.web $(docker inspect -f {{.ID}} hashicorp-learn)
12. Verify that the container has been imported into your Terraform state:
	terraform show

Create configuration
13. Create Terraform configuration
	terraform plan
14. Copy your Terraform state into your docker.tf file:
	terraform show -no-color > docker.tf
15. Inspect the docker.tf file to see that its contents have been replaced
16. Run the following code:
	terraform plan
17. After removing these optional attributes, your configuration should match the following:
	resource "docker_container" "web" {
    image = "sha256:87a94228f133e2da99cb16d653cd1373c5b4e8689956386c1c12b60a20421a02"
    name  = "hashicorp-learn"
    ports {
        external = 8080
        internal = 80
        ip       = "0.0.0.0"
        protocol = "tcp"
    }
}
18. Verify that the errors have been resolved:
	terraform plan
19. Apply the changes and finish the process of syncing your updated Terraform configuration and state with the Docker container
	terraform apply

Create image resource
20. To retrieve the image's tag name, run the following command, replacing <IMAGE-ID> with the image ID from docker.tf:
	docker image inspect <IMAGE-ID> -f {{.RepoTags}}
21. Add the following configuration to your docker.tf file to represent this image as a resource:
	resource "docker_image" "nginx" {
  name         = "nginx:latest"
}
22. Create an image resource in state:
	terraform apply
23. Change the image value for docker_container.web to reference the new image resource:
	resource "docker_container" "web" {
    image = docker_image.nginx.latest
    name  = "hashicorp-learn"
    ports {
        external = 8080
        internal = 80
        ip       = "0.0.0.0"
        protocol = "tcp"
    }
}
24. Look for changes:
	terraform apply

Manage the container with Terraform
25. In your docker.tf file, change the container's external port from 8080 to 8081:
	resource "docker_container" "web" {
  name  = "hashicorp-learn"
  image = docker_image.nginx.latest
  ports {
    external = 8081
    internal = 80
    ip       = "0.0.0.0"
    protocol = "tcp"
  }
}
25. Apply the change:
	terraform apply
26. Verify that the container has been replaced with a new one with the new configuration:
	docker ps

Destroy infrastructure
27. Destroy the container and image:
	terraform destroy
28. Validate that the container was destroyed:
	docker ps --filter "name=hashicorp-learn"

