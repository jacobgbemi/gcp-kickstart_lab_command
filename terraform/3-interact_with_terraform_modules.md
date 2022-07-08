### Task 1. Use modules from the Registry
#### Create a Terraform configuration
1. Clone the example simple project from the Google Terraform modules GitHub repository and switch to the v3.3.0 branch.
	```
	git clone https://github.com/terraform-google-modules/terraform-google-network
	cd terraform-google-network
	git checkout tags/v3.3.0 -b v3.3.0
	```

	### Define root input variables
	### Set values for module input variables
2. Navigate to terraform-google-network/examples/simple_project, and open the main.tf file.

3. Retrieve your Project ID
	```
	gcloud config list --format 'value(core.project)'
	```
4. Navigate to variables.tf.
5. Fill in the variable project_id with the output of the previous command.
	```
	variable "project_id" {
	  description = "The project ID to host the network in"
	  default     = "FILL IN YOUR PROJECT ID HERE"
	}
	```
6. In variables.tf, fill in the variable network_name. You can use the name example-vpc or any other name you'd like
	```
	variable "network_name" {
	  description = "The name of the VPC network being created"
	  default     = "example-vpc"
	}
	```

	### Define root output values
7. Navigate to the outputs.tf file inside of your configuration's directory. Verify the content
	
	### Provision infrastructure
8. Navigate to your simple_project directory:
	```
	cd ~/terraform-google-network/examples/simple_project
	```
9. Initialize your Terraform configuration:
	```
	terraform init
	```
10. Create your VPC:
	```
	terraform apply
	```
11. Respond to the prompt with ```yes```.

	### Clean up your infrastructure
12. Destroy the infrastructure you created:
	```
	terraform destroy
	```

### Task 2. Build a module
	```
	Module structure
	├── LICENSE
	├── README.md
	├── main.tf
	├── variables.tf
	├── outputs.tf
	```

### Create a module
1. Create the directory for your new module:
	```
	cd ~
	touch main.tf
	mkdir -p modules/gcs-static-website-bucket
	```
2. Navigate to the module directory and run the following commands to create three empty files:
	```
	cd modules/gcs-static-website-bucket
	touch website.tf variables.tf outputs.tf
	```
3. Inside the gcs-static-website-bucket directory, create a file called README.md with the following content:
	```
	# GCS static website bucket
	This module provisions Cloud Storage buckets configured for static website hosting.
	```
4. Create another file called LICENSE with the following content:
	```
	Licensed under the Apache License, Version 2.0 (the "License");
	you may not use this file except in compliance with the License.
	You may obtain a copy of the License at
	    http://www.apache.org/licenses/LICENSE-2.0
	Unless required by applicable law or agreed to in writing, software
	distributed under the License is distributed on an "AS IS" BASIS,
	WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
	See the License for the specific language governing permissions and
	limitations under the License.
	```
5. Add this Cloud Storage bucket resource to your website.tf file inside the modules/gcs-static-website-bucket directory:
	```
	resource "google_storage_bucket" "bucket" {
	  name               = var.name
	  project            = var.project_id
	  location           = var.location
	  storage_class      = var.storage_class
	  labels             = var.labels
	  force_destroy      = var.force_destroy
	  uniform_bucket_level_access = true
	  versioning {
	    enabled = var.versioning
	  }
	  dynamic "retention_policy" {
	    for_each = var.retention_policy == null ? [] : [var.retention_policy]
	    content {
	      is_locked        = var.retention_policy.is_locked
	      retention_period = var.retention_policy.retention_period
	    }
	  }
	  dynamic "encryption" {
	    for_each = var.encryption == null ? [] : [var.encryption]
	    content {
	      default_kms_key_name = var.encryption.default_kms_key_name
	    }
	  }
	  dynamic "lifecycle_rule" {
	    for_each = var.lifecycle_rules
	    content {
	      action {
		type          = lifecycle_rule.value.action.type
		storage_class = lookup(lifecycle_rule.value.action, "storage_class", null)
	      }
	      condition {
		age                   = lookup(lifecycle_rule.value.condition, "age", null)
		created_before        = lookup(lifecycle_rule.value.condition, "created_before", null)
		with_state            = lookup(lifecycle_rule.value.condition, "with_state", null)
		matches_storage_class = lookup(lifecycle_rule.value.condition, "matches_storage_class", null)
		num_newer_versions    = lookup(lifecycle_rule.value.condition, "num_newer_versions", null)
	      }
	    }
	  }
	}
	```
6. Navigate to the variables.tf file in your module and add the following code:
	```
	variable "name" {
	  description = "The name of the bucket."
	  type        = string
	}
	variable "project_id" {
	  description = "The ID of the project to create the bucket in."
	  type        = string
	}
	variable "location" {
	  description = "The location of the bucket."
	  type        = string
	}
	variable "storage_class" {
	  description = "The Storage Class of the new bucket."
	  type        = string
	  default     = null
	}
	variable "labels" {
	  description = "A set of key/value label pairs to assign to the bucket."
	  type        = map(string)
	  default     = null
	}
	variable "bucket_policy_only" {
	  description = "Enables Bucket Policy Only access to a bucket."
	  type        = bool
	  default     = true
	}
	variable "versioning" {
	  description = "While set to true, versioning is fully enabled for this bucket."
	  type        = bool
	  default     = true
	}
	variable "force_destroy" {
	  description = "When deleting a bucket, this boolean option will delete all contained objects. If false, Terraform will fail to delete buckets which contain objects."
	  type        = bool
	  default     = true
	}
	variable "iam_members" {
	  description = "The list of IAM members to grant permissions on the bucket."
	  type = list(object({
	    role   = string
	    member = string
	  }))
	  default = []
	}
	variable "retention_policy" {
	  description = "Configuration of the bucket's data retention policy for how long objects in the bucket should be retained."
	  type = object({
	    is_locked        = bool
	    retention_period = number
	  })
	  default = null
	}
	variable "encryption" {
	  description = "A Cloud KMS key that will be used to encrypt objects inserted into this bucket"
	  type = object({
	    default_kms_key_name = string
	  })
	  default = null
	}
	variable "lifecycle_rules" {
	  description = "The bucket's Lifecycle Rules configuration."
	  type = list(object({
	    # Object with keys:
	    # - type - The type of the action of this Lifecycle Rule. Supported values: Delete and SetStorageClass.
	    # - storage_class - (Required if action type is SetStorageClass) The target Storage Class of objects affected by this Lifecycle Rule.
	    action = any
	    # Object with keys:
	    # - age - (Optional) Minimum age of an object in days to satisfy this condition.
	    # - created_before - (Optional) Creation date of an object in RFC 3339 (e.g. 2017-06-13) to satisfy this condition.
	    # - with_state - (Optional) Match to live and/or archived objects. Supported values include: "LIVE", "ARCHIVED", "ANY".
	    # - matches_storage_class - (Optional) Storage Class of objects to satisfy this condition. Supported values include: MULTI_REGIONAL, REGIONAL, NEARLINE, COLDLINE, STANDARD, DURABLE_REDUCED_AVAILABILITY.
	    # - num_newer_versions - (Optional) Relevant only for versioned objects. The number of newer versions of an object to satisfy this condition.
	    condition = any
	  }))
	  default = []
	}
	```
7. Add an output to your module in the outputs.tf file inside your module:
	```
	output "bucket" {
	  description = "The created storage bucket"
	  value       = google_storage_bucket.bucket
	}
	```
8. Return to the main.tf in your root directory and add a reference to the new module:
	```
	module "gcs-static-website-bucket" {
	  source = "./modules/gcs-static-website-bucket"
	  name       = var.name
	  project_id = var.project_id
	  location   = "us-east1"
	  lifecycle_rules = [{
	    action = {
	      type = "Delete"
	    }
	    condition = {
	      age        = 365
	      with_state = "ANY"
	    }
	  }]
	}
	```
9. In your home directory, create an outputs.tf file for your root module:
	```cd ~
	touch outputs.tf
	```
10. Add the following code in the outputs.tf file:
	```
	output "bucket-name" {
	  description = "Bucket names."
	  value       = "module.gcs-static-website-bucket.bucket"
	}
	```
11. IN your home directory, create a variables.tf file:
	```
	touch variables.tf
	```
12. Add the following code to the variables.tf file and define the variables project_id and name:
	```
	variable "project_id" {
	  description = "The ID of the project in which to provision resources."
	  type        = string
	  default     = "FILL IN YOUR PROJECT ID HERE"
	}
	variable "name" {
	  description = "Name of the buckets to create."
	  type        = string
	  default     = "FILL IN YOUR (UNIQUE) BUCKET NAME HERE"
	}
	```

	### Install the local module
13. Install the module:
	```
	terraform init
	```
14. Provision your bucket:
	```
	terraform apply
	```
15. Respond ```yes``` to prompt

	### Upload files to the bucket
16. Download the sample contents to your home directory:
	```cd ~
	curl https://raw.githubusercontent.com/hashicorp/learn-terraform-modules/master/modules/aws-s3-static-website-bucket/www/index.html > index.html
	curl https://raw.githubusercontent.com/hashicorp/learn-terraform-modules/blob/master/modules/aws-s3-static-website-bucket/www/error.html > error.html
	```
17. Copy the files over to the bucket, replacing YOUR-BUCKET-NAME with the name of your storage bucket:
	```
	gsutil cp *.html gs://YOUR-BUCKET-NAME
	```
18. In a new tab in your browser, go to the website
	```
	https://storage.cloud.google.com/YOUR-BUCKET-NAME/index.html
	```

	### Clean up the website and infrastructure
19. Lastly, Destroy your Terraform resources:
	```
	terraform destroy
	```
