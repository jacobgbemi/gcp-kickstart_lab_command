Task 1: Verifying Terraform installation
1. Open a new Cloud Shell tab, and verify that Terraform is available:
	terraform

Task 2: Build infrastructure
1. create an empty configuration file named instance.tf containing:
	resource "google_compute_instance" "terraform" {
  project      = "<PROJECT_ID>"
  name         = "terraform"
  machine_type = "n1-standard-1"
  zone         = "us-central1-a"
  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-9"
    }
  }
  network_interface {
    network = "default"
    access_config {
    }
  }
}

Task 3: Initialization
1. Download and install the provider binary:
	terraform init
2. Create an execution plan:
	terraform plan
3. Apply changes
	terraform apply
4. Inspect the current state:
	terraform show

