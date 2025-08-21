## Terraform Essentials: Cloud Firestore Database

```
REGION=us-west1

gcloud services enable firestore.googleapis.com cloudbuild.googleapis.com

gcloud storage buckets create gs://$DEVSHELL_PROJECT_ID-tf-state --location=us

cat > main.tf << EOF
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 4.0"
    }
  }
  backend "gcs" {
    bucket = "$DEVSHELL_PROJECT_ID-tf-state"
    prefix = "terraform/state"
  }
}

provider "google" {
  project = "$DEVSHELL_PROJECT_ID"
  region  = "$REGION"
}

resource "google_firestore_database" "default" {
  name     = "default"
  project  = "$DEVSHELL_PROJECT_ID"
  location_id = "nam5"
  type     = "FIRESTORE_NATIVE"
}

output "firestore_database_name" {
  value       = google_firestore_database.default.name
  description = "The name of the Cloud Firestore database."
}
EOF

cat > variables.tf << EOF
variable "project_id" {
  type        = string
  description = "The ID of the Google Cloud project."
  default     = "$DEVSHELL_PROJECT_ID"
}

variable "bucket_name" {
  type        = string
  description = "Bucket name for terraform state"
  default     = "$DEVSHELL_PROJECT_ID-tf-state"
}
EOF

cat > outputs.tf << EOF
output "project_id" {
  value       = var.project_id
  description = "The ID of the Google Cloud project."
}

output "bucket_name" {
  value       = var.bucket_name
  description = "The name of the bucket to store terraform state."
}
EOF

terraform init

terraform plan

terraform apply
```

## Terraform Essentials: Google Compute Engine Instance

```
PROJECT_ID=$DEVSHELL_PROJECT_ID
ZONE=us-east1-b
REGION="${ZONE%-*}"

gsutil mb -l $REGION gs://$PROJECT_ID-tf-state

gsutil versioning set on gs://$PROJECT_ID-tf-state

cat > main.tf << EOF
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 4.0"
    }
  }
  backend "gcs" {
    bucket = "$PROJECT_ID-tf-state"
    prefix = "terraform/state"
  }
}

provider "google" {
  project = var.project_id
  region  = var.region
}

resource "google_compute_instance" "default" {
  name         = "terraform-instance"
  machine_type = "e2-micro"
  zone         = var.zone

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-12"
    }
  }

  network_interface {
    subnetwork = "default"

    access_config {
    }
  }
}
EOF

cat > variables.tf << EOF
variable "project_id" {
  type        = string
  description = "The ID of the Google Cloud project"
  default = "$PROJECT_ID"
}

variable "region" {
  type        = string
  description = "The region to deploy resources in"
  default     = "$REGION"
}

variable "zone" {
  type        = string
  description = "The zone to deploy resources in"
  default     = "$ZONE"
}
EOF

terraform init

terraform plan

terraform apply -auto-approve
```

## Terraform Essentials: Google Cloud Storage Bucket

```
PROJECT_ID=$DEVSHELL_PROJECT_ID
ZONE=us-central1-c
REGION="${ZONE%-*}"

gcloud config set compute/region $REGION
gcloud config set compute/zone $ZONE

gcloud storage buckets create gs://$PROJECT_ID-tf-state --location=$REGION --uniform-bucket-level-access

gsutil versioning set on gs://$PROJECT_ID-tf-state

mkdir terraform-gcs && cd $_

cat > main.tf << EOF
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 4.0"
    }
  }

  backend "gcs" {
    bucket = "$PROJECT_ID-tf-state"
    prefix = "terraform/state"
  }
}

provider "google" {
  project = "$PROJECT_ID"
  region  = "$REGION"
}

resource "google_storage_bucket" "default" {
  name          = "$PROJECT_ID-my-terraform-bucket"
  location      = "$REGION"
  force_destroy = true

  storage_class = "STANDARD"
  versioning {
    enabled = true
  }
}
EOF

terraform init

terraform plan

terraform apply -auto-approve
```

## Terraform Essentials: Service Account

```
PROJECT_ID=$DEVSHELL_PROJECT_ID
ZONE=us-west1-b
REGION="${ZONE%-*}"

gcloud config set compute/region $REGION
gcloud config set compute/zone $ZONE

gcloud services enable iam.googleapis.com

gcloud storage buckets create gs://$PROJECT_ID-tf-state --location=$REGION --uniform-bucket-level-access

gsutil versioning set on gs://$PROJECT_ID-tf-state

mkdir terraform-service-account && cd $_

cat > main.tf << EOF
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 4.0"
    }
  }
  backend "gcs" {
    bucket = "$PROJECT_ID-tf-state"
    prefix = "terraform/state"
  }
}

provider "google" {
  project = var.project_id
  region  = var.region 
}

resource "google_service_account" "default" {
  account_id   = "terraform-sa"
  display_name = "Terraform Service Account"
}
EOF

cat > variables.tf << EOF
variable "project_id" {
  type = string
  description = "The GCP project ID"
  default = "$PROJECT_ID"
}

variable "region" {
  type = string
  description = "The GCP region"
  default = "$REGION"
}
EOF

terraform init

terraform plan

terraform apply -auto-approve
```

## Terraform Essentials: Firewall Policy

```
PROJECT_ID=$DEVSHELL_PROJECT_ID
ZONE=us-east1-d
REGION="${ZONE%-*}"

gcloud config set compute/region $REGION
gcloud config set compute/zone $ZONE

gcloud storage buckets create gs://$PROJECT_ID-tf-state --location=$REGION --uniform-bucket-level-access

gsutil versioning set on gs://$PROJECT_ID-tf-state

cat > firewall.tf << EOF
resource "google_compute_firewall" "allow_ssh" {
  name    = "allow-ssh-from-anywhere"
  network = "default"
  project = "$PROJECT_ID"

  allow {
    protocol = "tcp"
    ports    = ["22"]
  }

  source_ranges = ["0.0.0.0/0"]
  target_tags   = ["ssh-allowed"]
}
EOF

cat > variables.tf << EOF
variable "project_id" {
  type = string
  default = "$PROJECT_ID"
}

variable "bucket_name" {
  type = string
  default = "$PROJECT_ID-tf-state"
}

variable "region" {
  type = string
  default = "$REGION"
}
EOF

cat > outputs.tf << EOF
output "firewall_name" {
  value = google_compute_firewall.allow_ssh.name
}
EOF

terraform init

terraform plan

terraform apply -auto-approve
```

## Terraform Essentials: VPC and Subnet

```
PROJECT_ID=$DEVSHELL_PROJECT_ID
ZONE=us-central1-a
REGION="${ZONE%-*}"

gcloud config set compute/region $REGION
gcloud config set compute/zone $ZONE

gcloud storage buckets create gs://$PROJECT_ID-terraform-state --location=us

gcloud services enable cloudresourcemanager.googleapis.com

cat > main.tf << EOF
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 4.0"
    }
  }
  backend "gcs" {
    bucket = "$PROJECT_ID-terraform-state"
    prefix = "terraform/state"
  }
}

provider "google" {
  project = "$PROJECT_ID"
  region  = "$REGION"
}

resource "google_compute_network" "vpc_network" {
  name                    = "custom-vpc-network"
  auto_create_subnetworks = false
}

resource "google_compute_subnetwork" "subnet_us" {
  name            = "subnet-us"
  ip_cidr_range   = "10.10.1.0/24"
  region          = "$REGION"
  network         = google_compute_network.vpc_network.id
}

resource "google_compute_firewall" "allow_ssh" {
  name    = "allow-ssh"
  network = google_compute_network.vpc_network.name
  allow {
    protocol = "tcp"
    ports    = ["22"]
  }
  source_ranges = ["0.0.0.0/0"]
}

resource "google_compute_firewall" "allow_icmp" {
  name    = "allow-icmp"
  network = google_compute_network.vpc_network.name

  allow {
    protocol = "icmp"
  }
  source_ranges = ["0.0.0.0/0"]
}
EOF

cat > variables.tf << EOF
variable "project_id" {
  type        = string
  description = "The ID of the Google Cloud project"
  default     = "$PROJECT_ID"
}

variable "region" {
  type        = string
  description = "The region to deploy resources in"
  default     = "$REGION"
}
EOF

cat > outputs.tf << EOF
output "network_name" {
  value       = google_compute_network.vpc_network.name
  description = "The name of the VPC network"
}

output "subnet_name" {
  value       = google_compute_subnetwork.subnet_us.name
  description = "The name of the subnetwork"
}
EOF

terraform init

terraform plan

terraform apply -auto-approve
```

## Developer Essentials: Application Development with Secret Manager

```
PROJECT_ID=$DEVSHELL_PROJECT_ID
REGION=us-central1

gcloud config set run/region $REGION

gcloud services enable secretmanager.googleapis.com run.googleapis.com artifactregistry.googleapis.com

gcloud secrets create arcade-secret --replication-policy=automatic

echo -n "t0ps3cr3t!" | gcloud secrets versions add arcade-secret --data-file=-

cat > app.py << EOF
import os
from flask import Flask, jsonify, request
from google.cloud import secretmanager
import logging

app = Flask(__name__)

# Configure logging
logging.basicConfig(level=logging.INFO)

# Initialize Secret Manager client
# The client will automatically use the service account credentials of the Cloud Run service
secret_manager_client = secretmanager.SecretManagerServiceClient()

# Hardcoded Project ID and Secret ID as per your request
PROJECT_ID = "$PROJECT_ID" # Project ID
SECRET_ID = "arcade-secret"   # Secret Identifier

@app.route('/')
def get_secret():
    """
    Retrieves the specified secret from Secret Manager and returns its payload.
    The SECRET_ID and PROJECT_ID are now hardcoded in the application.
    """
    if not SECRET_ID or not PROJECT_ID:
        logging.error("SECRET_ID or PROJECT_ID not configured (should be hardcoded).")
        return jsonify({"error": "Secret ID or Project ID not configured."}), 500

    secret_version_name = f"projects/{PROJECT_ID}/secrets/{SECRET_ID}/versions/latest"

    try:
        logging.info(f"Accessing secret: {secret_version_name}")
        # Access the secret version
        response = secret_manager_client.access_secret_version(request={"name": secret_version_name})
        secret_payload = response.payload.data.decode("UTF-8")

        # IMPORTANT: In a real application, you would process or use the secret
        # here, not return it directly in an HTTP response, especially if the
        # secret is sensitive. This example is for demonstration purposes only.
        return jsonify({"secret_id": SECRET_ID, "secret_value": secret_payload})

    except Exception as e:
        logging.error(f"Failed to retrieve secret '{SECRET_ID}': {e}")
        return jsonify({"error": f"Failed to retrieve secret: {str(e)}"}), 500

if __name__ == '__main__':
    # When running locally, Flask will use the hardcoded values directly.
    # In Cloud Run, these values are used without needing environment variables.
    app.run(host='0.0.0.0', port=int(os.environ.get('PORT', 8080)))
EOF

cat > requirements.txt << EOF
Flask==3.*
google-cloud-secret-manager==2.*
EOF

cat > Dockerfile << EOF
FROM python:3.9-slim-buster

WORKDIR /app

COPY requirements.txt .
RUN pip3 install -r requirements.txt

COPY . .

CMD ["python3", "app.py"]
EOF

gcloud artifacts repositories create arcade-images --repository-format=docker --location=$REGION --description="Docker repository"

docker build -t $REGION-docker.pkg.dev/$PROJECT_ID/arcade-images/arcade-secret:latest .

# docker run --rm -p 8080:8080 $REGION-docker.pkg.dev/$PROJECT_ID/arcade-images/arcade-secret:latest

docker push $REGION-docker.pkg.dev/$PROJECT_ID/arcade-images/arcade-secret:latest

gcloud iam service-accounts create arcade-service \
	--display-name="Arcade Service Account" \
	--description="Service account for Cloud Run application"

gcloud secrets add-iam-policy-binding arcade-secret \
	--member="serviceAccount:arcade-service@$PROJECT_ID.iam.gserviceaccount.com" \
	--role="roles/secretmanager.secretAccessor"

gcloud run deploy arcade-service \
	--image=$REGION-docker.pkg.dev/$PROJECT_ID/arcade-images/arcade-secret:latest \
	--region=$REGION \
	--set-secrets SECRET_ENV_VAR=arcade-secret:latest \
	--service-account arcade-service@$PROJECT_ID.iam.gserviceaccount.com \
	--allow-unauthenticated
```

## Developer Essentials: Creating Secrets with Secret Manager

```
gcloud services enable secretmanager.googleapis.com

gcloud secrets create my-secret

echo -n "super-secret-password" | gcloud secrets versions add my-secret --data-file=-

# echo "$(gcloud secrets versions access latest --secret=my-secret)"
```
