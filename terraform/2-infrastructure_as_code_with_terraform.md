### Task 1: Build Infrastructure
1. Create the main.tf file containing:
	```
	terraform {
	  required_providers {
	    google = {
	      source = "hashicorp/google"
	    }
	  }
	}
	
	provider "google" {
	  version = "3.5.0"
	  project = "<PROJECT_ID>"
	  region  = "us-central1"
	  zone    = "us-central1-c"
	}
	resource "google_compute_network" "vpc_network" {
	  name = "terraform-network"
	}
	```
2. Initialization
	```
	terraform init
	```
3. Creating Resources
	```
	terraform apply
	```
4. Inspect the current state.
	```
	terraform show
	```

### Task 2: Change Infrastructure
1. Add a compute instance resource to main.tf:
	```
	resource "google_compute_instance" "vm_instance" {
	  name         = "terraform-instance"
	  machine_type = "f1-micro"
	  boot_disk {
	    initialize_params {
	      image = "debian-cloud/debian-9"
	    }
	  }
	  network_interface {
	    network = google_compute_network.vpc_network.name
	    access_config {
	    }
	  }
	}
	```
2. Now run terraform apply to create the compute instance:
	```
	terraform apply
	```
3. Changing Resources
	```
	resource "google_compute_instance" "vm_instance" {
	  name         = "terraform-instance"
	  machine_type = "f1-micro"
	  tags         = ["web", "dev"]
	  # ...
	}
	```
4. Run terraform apply again to update the instance.
	```
	terraform apply
	```
5. Destructive Changes
	```
	boot_disk {
	    initialize_params {
	      image = "cos-cloud/cos-stable"
	    }
	 }
	 ```
6. Now run terraform apply again to see how Terraform will apply this change to the existing resources:
	```
	terraform apply
	```
7. Destroy Infrastructure
	```
	terraform destroy
	```

### Task 3: Create Resource Dependencies
1. Recreate your network and instance. After you respond to the prompt with yes, the resources will be created.
	```
	terraform apply
	```
2. Now add to your configuration by assigning a static IP to the VM instance in main.tf.
	```
	resource "google_compute_address" "vm_static_ip" {
	  name = "terraform-static-ip"
	}
	```
3. Next, run terraform plan.
	```
	terraform plan
	```
4. Update the network_interface configuration for your instance like so:
	  ```
	  network_interface {
	    network = google_compute_network.vpc_network.self_link
	    access_config {
	      nat_ip = google_compute_address.vm_static_ip.address
	    }
	  }
	  ```
5. Run terraform plan again, but this time, save the plan:
	```
	terraform plan -out static_ip
	```
6. Run terraform apply "static_ip" to see how Terraform plans to apply this change:
	```
	terraform apply "static_ip"
	```
7. Add a Cloud Storage bucket and an instance with an explicit dependency on the bucket by adding the following to main.tf:
	```
	# New resource for the storage bucket our application will use.
	resource "google_storage_bucket" "example_bucket" {
	  name     = "<UNIQUE-BUCKET-NAME>"
	  location = "US"
	  website {
	    main_page_suffix = "index.html"
	    not_found_page   = "404.html"
	  }
	}
	# Create a new instance that uses the bucket
	resource "google_compute_instance" "another_instance" {
	  # Tells Terraform that this VM instance must be created only after the
	  # storage bucket has been created.
	  depends_on = [google_storage_bucket.example_bucket]
	  name         = "terraform-instance-2"
	  machine_type = "f1-micro"
	  boot_disk {
	    initialize_params {
	      image = "cos-cloud/cos-stable"
	    }
	  }
	  network_interface {
	    network = google_compute_network.vpc_network.self_link
	    access_config {
	    }
	  }
	}
	```
8. Now run terraform plan and terraform apply to see these changes in action:
	```
	terraform plan
	terraform apply
	```

### Task 4: Provision Infrastructure
1. To define a provisioner, modify the resource block defining the first vm_instance in your configuration to look like the following:
	```
	resource "google_compute_instance" "vm_instance" {
	  name         = "terraform-instance"
	  machine_type = "f1-micro"
	  tags         = ["web", "dev"]
	  provisioner "local-exec" {
	    command = "echo ${google_compute_instance.vm_instance.name}:  ${google_compute_instance.vm_instance.network_interface[0].access_config[0].nat_ip} >> ip_address.txt"
	  }
	  # ...
	}
	```
2. Run terraform apply. At this point, the output may be confusing at first.
	```
	terraform apply
	```
3. Use terraform taint to tell Terraform to recreate the instance:
	```
	terraform taint google_compute_instance.vm_instance
	```
4. Run terraform apply now:
	```
	terraform apply
	```
5. Verify everything worked by looking at the contents of the ip_address.txt file.
