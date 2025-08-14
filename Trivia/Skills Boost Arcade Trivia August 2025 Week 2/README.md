## Build an LLM and RAG-based Chat Application using AlloyDB and LangChain

```shell
# Task 5. Deploy the Retrieval Service to Cloud Run

export PROJECT_ID=$(gcloud config get-value project)
gcloud iam service-accounts create retrieval-identity
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:retrieval-identity@$PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/aiplatform.user"

# Task 1. Initialize the Environment

gcloud compute ssh instance-1 --zone=us-central1-c

sudo apt-get update
sudo apt-get install --yes postgresql-client 

export PGPASSWORD=alloydbworkshop2023

export PROJECT_ID=$(gcloud config get-value project)
export REGION=us-central1
export ADBCLUSTER=alloydb-aip-01
export INSTANCE_IP=$(gcloud alloydb instances describe $ADBCLUSTER-pr --cluster=$ADBCLUSTER --region=$REGION --format="value(ipAddress)")
psql "host=$INSTANCE_IP user=postgres sslmode=require" 

exit 

# Task 2. Initialize the database

psql "host=$INSTANCE_IP user=postgres" -c "CREATE DATABASE assistantdemo" 

psql "host=$INSTANCE_IP user=postgres dbname=assistantdemo" -c "CREATE EXTENSION vector"   

# Task 3. Install Python

sudo apt install -y python3.11-venv git
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip

# python -V

# Task 4. Populate Database

git clone https://github.com/GoogleCloudPlatform/genai-databases-retrieval-app.git

cd genai-databases-retrieval-app/retrieval_service
cp example-config.yml config.yml
sed -i s/127.0.0.1/$INSTANCE_IP/g config.yml
sed -i s/my-password/$PGPASSWORD/g config.yml
sed -i s/my_database/assistantdemo/g config.yml
sed -i s/my-user/postgres/g config.yml
# cat config.yml

pip install -r requirements.txt
python run_database_init.py

# Task 6. Deploy the Retrieval Service

cd ~/genai-databases-retrieval-app
gcloud alpha run deploy retrieval-service \
    --source=./retrieval_service/\
    --no-allow-unauthenticated \
    --service-account retrieval-identity \
    --region us-central1 \
    --network=default \
    --quiet

# curl -H "Authorization: Bearer $(gcloud auth print-identity-token)" $(gcloud  run services list --filter="(retrieval-service)" --format="value(URL)")

# Task 7. Deploy sample application

export BASE_URL=$(gcloud  run services list --filter="(retrieval-service)" --format="value(URL)")
echo $BASE_URL
```

Google Auth Platform

Create branding

- App name: Cymbal Air
- User support email: student-xx-xxxxxxxxxxxx@qwiklabs.net
- Audience: Internal
- Contact information: student-xx-xxxxxxxxxxxx@qwiklabs.net

Create client

- Application type: Web application
- Name: Web client 1
- Authorized JavaScript origins:
	- $BASE_URL
	- $BASE_URL:8081
- Authorized redirect URLs: $BASE_URL:8081/login/google

## Dataflow: Qwik Start - Python

```shell
REGION=us-east1

gcloud config set compute/region $REGION

gcloud services disable dataflow.googleapis.com

gcloud services enable dataflow.googleapis.com

# Task 1. Create a Cloud Storage bucket

gsutil mb gs://$DEVSHELL_PROJECT_ID-bucket/

# Task 2. Install the Apache Beam SDK for Python

docker run -it -e DEVSHELL_PROJECT_ID=$DEVSHELL_PROJECT_ID -e REGION=$REGION python:3.9 /bin/bash

pip install "apache-beam[gcp]"==2.42.0

# python -m apache_beam.examples.wordcount --output OUTPUT_FILE

# Task 3. Run an example Dataflow pipeline remotely

BUCKET=gs://$DEVSHELL_PROJECT_ID-bucket

python -m apache_beam.examples.wordcount --project $DEVSHELL_PROJECT_ID \
  --runner DataflowRunner \
  --staging_location $BUCKET/staging \
  --temp_location $BUCKET/temp \
  --output $BUCKET/results/output \
  --region $REGION
```

## Terraform Fundamentals

```
ZONE=us-central1-a

cat > instance.tf << EOF
resource "google_compute_instance" "terraform" {
  project      = "$DEVSHELL_PROJECT_ID"
  name         = "terraform"
  machine_type = "e2-medium"
  zone         = "$ZONE"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }

  network_interface {
    network = "default"
    access_config {
    }
  }
}
EOF

terraform init

terraform plan

terraform apply
```

## Skills Boost Arcade Trivia August 2025 Week 2

1. In a RAG-based chat application, what type of model is responsible for generating the final human-like response after relevant information has been retrieved?

A Large Language Model (LLM)

2. Which popular open-source framework acts as the orchestrator, connecting the LLM, vector store, and other components in this RAG application?

LangChain

3. Which command will you use to pull a Docker container with the latest stable version of Python 3.9?

docker run

4. In the Apache Beam SDK for Dataflow, what is the top-level object that encapsulates the entire data processing workflow, from input to output?

Pipeline

5. What open-source tool allows you to define and provision infrastructure using a declarative configuration language?

Terraform

6. In Terraform, what command is used to create an execution plan that shows what actions Terraform will take to achieve the desired state?

terraform plan
