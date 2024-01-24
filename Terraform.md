The dictionary definition of "terraform" is to transform (a planet) so as to resemble the earth, especially so that it can support human life. The software Terraform by HashiCorp does something similar where it sets up infrastructure on a cloud or local platform so that your code can run. From Terraform itself:
>Terraform is an infrastructure as code tool that lets you define both cloud and on-prem resources in human-readable configuration files so that you can version, reuse, and share. You can then use a consistent workflow to provision and manage all of your infrastructure throughout its lifecycle.

## Why Terraform?
* Simplicity in keeping track of infrastructure, for example:
	* what size your disks are 
	* what types of storage you have
* Easier collaboration since it is just a file
* Reproducibility
* Ensure resources are removed to prevent extra costs using a quick Terraform command

## What Terraform is not
* Does not manage and update code on infrastructure
* Does not give you the ability to change immutable resources (such as a virtual machine type)
* Not used to manage resources not defined in your terraform files

## What Terraform is
* Infrastructure as code

Once Terraform is downloaded onto your local/host machine, we will define what providers we want it to have (e.g., Amazon Web Services) in the Terraform file. When Terraform reads the file, it reaches out to get the, in this case, AWS provider then connects to AWS via that provider to give you access. **Providers** are the code that allows Terraform to communicate to manage resources on many platforms, including but not limited to:
* AWS
* Azure
* GCP
* Kubernetes
* VSphere
* Alibaba Cloud
* Oracle Cloud Infrastructure
* Active Directory

## Key Terraform commands
**init** : get me the providers I need
**plan** : what am I about to do?
**apply** : do what is in the terraform (.tf) files
**destroy** : remove everything defined in the terraform files

We start by creating a **service account** in the platform we want Terraform to connect us to. A **service account** is similar to a user account except that is never meant to be logged into as it is going to be used by software to run tasks/programs.

In GCP (Google Cloud Platform), we will create a project named **terraform-demo**. To create the service account, click the **IAM & Admin** button.

![[Screenshot 2024-01-23 at 2.37.30 PM.png]]

Then using the left menu navigate to Service Accounts and click + CREATE SERVICE ACCOUNT in the upper middle.

![[Screenshot 2024-01-23 at 2.37.58 PM.png]]

Give the Service account name **terraform-runner** and select the **create and continue** button.

Since we are creating a GCP bucket, we want the **Storage Admin** role permissions located in the **Cloud Storage** submenu. We also want the **BigQuery Admin** role permissions located in the **BigQuery** submenu. In the industrial uses of service accounts, we would limit these permissions further. Select **continue** and then **done**. To show how easy it is to update permissions on that account, you can click on IAM in the left menu, select the **Edit principal** icon (pencil icon) to easily add/remove roles.

Now navigate back to **Service Accounts** and click the three dots to the right of the service account we just created and select **Manage keys**. Then select **ADD KEY** and **Create new key** (JSON default is good). A .json file will be downloaded to your computer.

>**Warning:** Do not show or give access to the .json file generated to anyone. This .json file has credentials to your account.

Before we get going on building our Terraform file, let's ensure that the appropriate Terraform files are included in our .gitignore file assuming that this project is being hosted on GitHub. To do this, search "terraform gitignore" on Google and copy paste the code from here:
https://github.com/github/gitignore/blob/main/Terraform.gitignore
into your .gitignore file. Furthermore, we will be creating a folder called keys that contain the .json file storing credentials to the service account. Be sure that keys/ is also listed in your .gitignore file.

In your project directory, you can create a new folder called **terrademo** and within that create a folder called **keys**. Be sure to include keys/ in your .gitignore file as this folder will contain the .json file we said should never be made public. You can move or copy this file into that keys folder as a file called **my-creds.json**.

Now create a file in the terrademo directory called **main.tf**, note that .tf is the file extension for Terraform files. Similar to our Dockerfile, we want to define the providers that we need in this file. Since we want a GCP provider, we can search on google "terraform gcp provider" to get the snippets of code we need in our terraform file. You should be directed to this page:

https://registry.terraform.io/providers/hashicorp/google/latest/docs/guides/provider_reference.html

Where you can copy the snippet of code needed via the **USE PROVIDER** button in the upper right corner.

```terraform
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
      version = "5.13.0"
    }
  }
}

provider "google" {
  # Configuration options
}
```

Note that there is a comment requesting the Configuration options. On the same website linked above, the page gives an Example usage with two configuration options (project and region). We can copy that template over.

```terraform
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
      version = "5.13.0"
    }
  }
}

provider "google" {
  project = "my-project-id"
  region = "us-central1"
}
```

As an aside, Terraform can nicely format your .tf document using the command terraform fmt in the terminal that is navigated to the same directory in which the .tf file lives.

Now let's fill in the two pieces of information requested in the configuration options. Our project id is not the friendly name terraform-demo, instead you can find it in GCP by navigating to the Dashboard (click the menu icon to the left of the Google Cloud logo, then Cloud overview and Dashboard). Then you will see your Project ID in the Project info card. For me, the Project ID is iron-cycle-412122.

![[Screenshot 2024-01-23 at 3.08.22 PM.png]]
So now my main.tf file looks like this:

```terraform
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
      version = "5.13.0"
    }
  }
}

provider "google" {
  project = "iron-cycle-412122"
  region = "us-central1"
}
```

Now we also need to tell Terraform where the credentials to the service account are.

```terraform
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
      version = "5.13.0"
    }
  }
}

provider "google" {
  credentials = "./keys/my-creds.json"
  project = "iron-cycle-412122"
  region = "us-central1"
}
```

Instead of hardcoding the location of the credentials, we can export an environment variable called GOOGLE_CREDENTIALS with that filepath and Terraform will know to use the value of that variable when looking for the credentials file. For example, we would do:

```console
export GOOGLE_CREDENTIALS='~/Documents/dataengineering_zoomcamp/week_1/terrademo/keys/my-creds.json'
```

We can check that it was saved using 

```console
echo $GOOGLE_CREDENTIALS
```

Then in our main.tf file, for the "google" provider we can leave off the credentials option.

If you ever want to unset this environment variable, you can use the command:

```console
unset GOOGLE_CREDENTIALS
```

The analogy used to describe the functions of the provider/key is to think of the provider as what allows us to go to the door of GCP and the key gets us through the door.

To start up Terraform, we use the **init** command we previously mentioned in the terminal navigated to the directory where our main.tf file is:

```console
terraform init
```


This command gives the following friendly message.
![[Screenshot 2024-01-23 at 3.22.06 PM.png]]

Before we use the terraform plan command, let's include the creation of a bucket in our Google Cloud Storage account. This bucket is referred to as a **resource** in Terraform, and we can look up what the code is for this type of resource on Google by searching "terraform google cloud storage button" and finding an example usage such as:

```terraform
resource "google_storage_bucket" "auto-expire" {
  name          = "auto-expiring-bucket"
  location      = "US"
  force_destroy = true

  lifecycle_rule {
    condition {
      age = 3
    }
    action {
      type = "Delete"
    }
  }

  lifecycle_rule {
    condition {
      age = 1
    }
    action {
      type = "AbortIncompleteMultipartUpload"
    }
  }
}
```

The name "google_storage_bucket" is important for Terraform to know what resource to create. Let's change the names of a few things with the following notes:
* "auto-expire" is a local variable name which can be changed to something that does not need to be globally unique we will change it to "demo-bucket"
* "auto-expiring-bucket" is the name of the bucket, it does need to be globally unique so we can incorporate our project id here "iron-cycle-412122-terra-bucket"

We will delete the lifecycle_rule of deleting this bucket after 3 days. We will leave the other lifecyle_rule and now here is our new resource code snippet in our main.tf file:

```terraform
resource "google_storage_bucket" "demo-bucket" {
  name          = "iron-cycle-412122-terra-bucket"
  location      = "US"
  force_destroy = true

  lifecycle_rule {
    condition {
      age = 1
    }
    action {
      type = "AbortIncompleteMultipartUpload"
    }
  }
}
```

Now we have everything in our main.tf file for this demo. We will run the following command to get an idea of what Terraform is going to create as a result of our Terraform file:

```console
terraform plan
```

![[Screenshot 2024-01-23 at 3.38.00 PM.png]]

Once you have checked the plan and confirmed that it is going to do what you want, you can create the infrastructure with the following command:

```console
terraform apply
```

You can then type yes if you want to confirm the actions Terraform will take.

![[Screenshot 2024-01-23 at 3.40.35 PM.png]]

Now when we go to Google Cloud Platform and navigate to our Buckets in Cloud Storage, we will see the bucket we just created through Terraform:

![[Screenshot 2024-01-23 at 3.41.46 PM.png]]

Now let's say we want to get rid of the resources after we are done with them. We can use the command

```console
terraform destroy
```

It will tell us the changes that will be made if we execute this. We confirm it by typing yes and we get the following confirmation message:

![[Screenshot 2024-01-23 at 3.43.35 PM.png]]

We will also see in our Cloud Storage on GCP that the bucket is no longer there.

Now instead of just creating a bucket, let's create a BigQuery dataset with Terraform. First, we search "terraform bigquery dataset" in Google and that lands us here:
https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/bigquery_dataset

The basic example given is quite complex already, so to use it we should try to understand a bit of what is going on. Let's look for the word Required in the page and we find dataset_id is the only thing that is required except if we plan on using certain blocks. We may also want to specify the location of the dataset as there may be billing associated with the location. But for now let's go with the really simple:

```terraform
resource "google_bigquery_dataset" "demo_dataset" {
	dataset_id = "example_dataset"
}
```

When we use the **plan** command, we will see the dataset will be housed in the US.

When we use the **apply** command, we can see both the bucket in Cloud Storage and the dataset BigQuery:

![[Screenshot 2024-01-23 at 4.11.04 PM.png]]

## Variables

We will now create a new file called **variables.tf** in our project directory where we will define variables that can be used in our other terraform files. The general form for defining a variable in this file is the following:

```terraform
variable "variable_name" {
	description = "Description of variable"
	default = "default variable value"
}
```

Then when we go to use this variable in any of our other Terraform files, we can refer to it via the name var.variable_name. In using the following variables.tf file:

```terraform
variable "region" {
	description = "Project region"
	default = "uscentral1"
}

variable "project" {
	description = "Project"
	default = "iron-cycle-412122"
}

variable "location" {
	description = "Project location"
	default = "US"
}

variable "bq_dataset_name" {
	description = "My BigQuery Dataset Name"
	default = "example_dataset"
}  

variable "gcs_bucket_name" {
	description = "My Cloud Storage Bucket Name"
	default = "iron-cycle-412122-terra-bucket"
}

variable gcs_storage_class {
	description = "Bucket Storage Class"
	default = "STANDARD"
}
```

We can update our main.tf file to the following:

```terraform
terraform {
	required_providers {
		google = {
			source = "hashicorp/google"
			version = "5.13.0"
		}
	}
}

  

provider "google" {
	project = var.project
	region = var.region
	}

resource "google_storage_bucket" "demo-bucket" {
	name = var.gcs_bucket_name
	location = var.location
	force_destroy = true

	lifecycle_rule {
		condition {
			age = 1
		}
		action {
			type = "AbortIncompleteMultipartUpload"
		}
	}

}

resource "google_bigquery_dataset" "demo_dataset" {
	dataset_id = var.bq_dataset_name
	location = var.location
}
```

Instead of using an environment variable, we can even set the path to our Google credentials in our variables.tf file. This is what that would look like in the variables.tf file:

```terraform
variable "credentials" {
	description = "My Credentials"
	default = "./keys/my-creds.json"
}
```

and then we can update our "google" provider options in the main.tf file as follows:

```terraform
provider "google" {
	credentials = var.credentials
	project = var.project
	region = var.region
}
```

# Homework

Output from terraform apply:

Terraform used the selected providers to generate the following execution plan. Resource
actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_bigquery_dataset.demo_dataset will be created
  + resource "google_bigquery_dataset" "demo_dataset" {
      + creation_time              = (known after apply)
      + dataset_id                 = "demo_dataset"
      + default_collation          = (known after apply)
      + delete_contents_on_destroy = false
      + effective_labels           = (known after apply)
      + etag                       = (known after apply)
      + id                         = (known after apply)
      + is_case_insensitive        = (known after apply)
      + last_modified_time         = (known after apply)
      + location                   = "US"
      + max_time_travel_hours      = (known after apply)
      + project                    = "iron-cycle-412122"
      + self_link                  = (known after apply)
      + storage_billing_model      = (known after apply)
      + terraform_labels           = (known after apply)
    }

  # google_storage_bucket.demo-bucket will be created
  + resource "google_storage_bucket" "demo-bucket" {
      + effective_labels            = (known after apply)
      + force_destroy               = true
      + id                          = (known after apply)
      + location                    = "US"
      + name                        = "iron-cycle-412122-terra-bucket"
      + project                     = (known after apply)
      + public_access_prevention    = (known after apply)
      + rpo                         = (known after apply)
      + self_link                   = (known after apply)
      + storage_class               = "STANDARD"
      + terraform_labels            = (known after apply)
      + uniform_bucket_level_access = (known after apply)
      + url                         = (known after apply)

      + lifecycle_rule {
          + action {
              + type = "AbortIncompleteMultipartUpload"
            }
          + condition {
              + age                   = 1
              + matches_prefix        = []
              + matches_storage_class = []
              + matches_suffix        = []
              + with_state            = (known after apply)
            }
        }
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_bigquery_dataset.demo_dataset: Creating...
google_storage_bucket.demo-bucket: Creating...
google_bigquery_dataset.demo_dataset: Creation complete after 1s [id=projects/iron-cycle-412122/datasets/demo_dataset]
google_storage_bucket.demo-bucket: Creation complete after 2s [id=iron-cycle-412122-terra-bucket]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

hi i'm jessica