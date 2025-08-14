## Creating Cross-region Load Balancing

```
ZONE_1=us-east1-c
ZONE_2=us-west1-c

REGION_1="${ZONE_1%-*}"
REGION_2="${ZONE_2%-*}"



gcloud compute instances create www-1 \
    --image-family debian-11 \
    --image-project debian-cloud \
    --machine-type e2-micro \
    --zone $ZONE_1 \
    --tags http-tag \
    --metadata startup-script="#! /bin/bash
      apt-get update
      apt-get install apache2 -y
      service apache2 restart
      Code"

gcloud compute instances create www-2 \
    --image-family debian-11 \
    --image-project debian-cloud \
    --machine-type e2-micro \
    --zone $ZONE_1 \
    --tags http-tag \
    --metadata startup-script="#! /bin/bash
      apt-get update
      apt-get install apache2 -y
      service apache2 restart
      Code"

gcloud compute instances create www-3 \
    --image-family debian-11 \
    --image-project debian-cloud \
    --machine-type e2-micro \
    --zone $ZONE_2 \
    --tags http-tag \
    --metadata startup-script="#! /bin/bash
      apt-get update
      apt-get install apache2 -y
      service apache2 restart
      Code"

gcloud compute instances create www-4 \
    --image-family debian-11 \
    --image-project debian-cloud \
    --machine-type e2-micro \
    --zone $ZONE_2 \
    --tags http-tag \
    --metadata startup-script="#! /bin/bash
      apt-get update
      apt-get install apache2 -y
      service apache2 restart
      Code"

gcloud compute firewall-rules create www-firewall \
    --target-tags http-tag --allow tcp:80

# gcloud compute instances list



gcloud compute addresses create lb-ip-cr \
    --ip-version=IPV4 \
    --global

gcloud compute instance-groups unmanaged create $REGION_1-resources-w --zone $ZONE_1

gcloud compute instance-groups unmanaged create $REGION_2-resources-w --zone $ZONE_2

gcloud compute instance-groups unmanaged add-instances $REGION_1-resources-w \
    --instances www-1,www-2 \
    --zone $ZONE_1

gcloud compute instance-groups unmanaged add-instances $REGION_2-resources-w \
    --instances www-3,www-4 \
    --zone $ZONE_2

gcloud compute health-checks create http http-basic-check



gcloud compute instance-groups unmanaged set-named-ports $REGION_1-resources-w \
    --named-ports http:80 \
    --zone $ZONE_1

gcloud compute instance-groups unmanaged set-named-ports $REGION_2-resources-w \
    --named-ports http:80 \
    --zone $ZONE_2

gcloud compute backend-services create web-map-backend-service \
    --protocol HTTP \
    --health-checks http-basic-check \
    --global

gcloud compute backend-services add-backend web-map-backend-service \
    --balancing-mode UTILIZATION \
    --max-utilization 0.8 \
    --capacity-scaler 1 \
    --instance-group $REGION_1-resources-w \
    --instance-group-zone $ZONE_1 \
    --global

gcloud compute backend-services add-backend web-map-backend-service \
    --balancing-mode UTILIZATION \
    --max-utilization 0.8 \
    --capacity-scaler 1 \
    --instance-group $REGION_2-resources-w \
    --instance-group-zone $ZONE_2 \
    --global

gcloud compute url-maps create web-map \
    --default-service web-map-backend-service

gcloud compute target-http-proxies create http-lb-proxy \
    --url-map web-map

LB_IP_ADDRESS=$(gcloud compute addresses list --format="get(ADDRESS)")

gcloud compute forwarding-rules create http-cr-rule \
    --address $LB_IP_ADDRESS \
    --global \
    --target-http-proxy http-lb-proxy \
    --ports 80
```

## Distributed Load Testing Using Kubernetes

```shell
PROJECT=$(gcloud config get-value project)
ZONE=us-central1-a
REGION="${ZONE%-*}"
CLUSTER=gke-load-test
TARGET=${PROJECT}.appspot.com
gcloud config set compute/region $REGION
gcloud config set compute/zone $ZONE

gsutil -m cp -r gs://spls/gsp182/distributed-load-testing-using-kubernetes .
cd distributed-load-testing-using-kubernetes/sample-webapp/
sed -i "s/python37/python39/g" app.yaml
cd ..
gcloud builds submit --tag gcr.io/$PROJECT/locust-tasks:latest docker-image/.

gcloud app create --region=$REGION
gcloud app deploy sample-webapp/app.yaml

gcloud container clusters create $CLUSTER \
  --zone $ZONE \
  --num-nodes=5

sed -i -e "s/\[TARGET_HOST\]/$TARGET/g" kubernetes-config/locust-master-controller.yaml
sed -i -e "s/\[TARGET_HOST\]/$TARGET/g" kubernetes-config/locust-worker-controller.yaml
sed -i -e "s/\[PROJECT_ID\]/$PROJECT/g" kubernetes-config/locust-master-controller.yaml
sed -i -e "s/\[PROJECT_ID\]/$PROJECT/g" kubernetes-config/locust-worker-controller.yaml

kubectl apply -f kubernetes-config/locust-master-controller.yaml
# kubectl get pods -l app=locust-master
kubectl apply -f kubernetes-config/locust-master-service.yaml
# kubectl get svc locust-master

kubectl apply -f kubernetes-config/locust-worker-controller.yaml
# kubectl get pods -l app=locust-worker
kubectl scale deployment/locust-worker --replicas=20
# kubectl get pods -l app=locust-worker

# EXTERNAL_IP=$(kubectl get svc locust-master -o yaml | grep ip: | awk -F": " '{print $NF}')
# echo http://$EXTERNAL_IP:8089
```

## App Dev: Adding User Authentication to your Application - Python

```shell
git clone https://github.com/GoogleCloudPlatform/training-data-analyst

ln -s ~/training-data-analyst/courses/developingapps/v1.3/python/firebase ~/firebase

cd ~/firebase/start

export REGION=us-west1
sed -i "s/us-central/$REGION/g" prepare_environment.sh

. prepare_environment.sh

python run_server.py
```

Identity Platform

Add a provider:

- Sign-in method: Email / Password
- Enabled
- Authorized domains (Cloud Shell Web Preview): 8080-cs-3bd64301-b3b5-48db-9f9b-1b7d34e9acd0.ql-asia-southeast1-dsfl.cloudshell.dev

Add user:

- Email: user1@example.com
- Password: abc123!

#### Reference

```
#!/bin/bash

# Copyright 2017 Google Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

echo "Creating Datastore/App Engine instance"
gcloud app create --region "us-west1"

echo "Creating bucket: gs://$DEVSHELL_PROJECT_ID-media"
gsutil mb gs://$DEVSHELL_PROJECT_ID-media

echo "Exporting GCLOUD_PROJECT and GCLOUD_BUCKET"
export GCLOUD_PROJECT=$DEVSHELL_PROJECT_ID
export GCLOUD_BUCKET=$DEVSHELL_PROJECT_ID-media

echo "Creating virtual environment"
mkdir ~/venvs
virtualenv -p python3 ~/venvs/developingapps
source ~/venvs/developingapps/bin/activate

echo "Installing Python libraries"
pip install --upgrade pip
pip install -r requirements.txt

echo "Creating Datastore entities"
python add_entities.py

echo "Project ID: $DEVSHELL_PROJECT_ID"
```

## Skills Boost Arcade Trivia August 2025 Week 3

1. Which component of a Google Cloud cross-region load balancer is used to check the health of the instances?

Backend Service

2. Which component directs incoming user requests from the load balancer's IP address to the correct target proxy?

Global Forwarding Rule

3. Which command from the following will you use to deploy applications to Google App Engine?

gcloud app deploy

4. In Kubernetes, what is the smallest, most fundamental deployable unit that encapsulates one or more containers, along with shared storage and network resources?

Pod

5. What Google Cloud service is commonly used to add secure user authentication to applications, including support for various sign-in methods?

Firebase Authentication

6. What process is involved in confirming a user's identity through their provided credentials, such as an ID token, to ensure they are who they claim to be?

Authentication
