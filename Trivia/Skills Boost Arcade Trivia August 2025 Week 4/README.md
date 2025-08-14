## Modular Load Balancing with Terraform - Regional Load Balancer

```shell
git clone https://github.com/GoogleCloudPlatform/terraform-google-lb
cd ~/terraform-google-lb/examples/basic

export GOOGLE_PROJECT=$(gcloud config get-value project)

terraform init

export REGION=us-west1
sed -i 's/us-central1/'"$REGION"'/g' variables.tf

terraform plan

terraform apply

# EXTERNAL_IP=$(terraform output | grep load_balancer_default_ip | cut -d = -f2 | xargs echo -n)

# echo "http://${EXTERNAL_IP}"
```

## Create VPC Peering Connection between VPCs

```shell
gcloud compute networks peerings create peering-1 \
    --network=workspace-vpc \
    --peer-network=private-vpc

gcloud compute networks peerings create peering-2 \
    --network=private-vpc \
    --peer-network=workspace-vpc
```

## Modify VM Instance for Cost Optimization

```shell
INSTANCE_NAME=lab-vm

ZONE=$(gcloud compute instances list --filter=name:$INSTANCE_NAME --format='value(ZONE)')

gcloud compute instances stop $INSTANCE_NAME --zone=$ZONE

gcloud compute instances set-machine-type $INSTANCE_NAME --zone=$ZONE --machine-type=e2-medium

gcloud compute instances start $INSTANCE_NAME --zone=$ZONE
```

## Skills Boost Arcade Trivia August 2025 Week 4

1. Which Terraform command initializes a working directory containing Terraform configuration files?

terraform init

2. In Terraform, which command executes the actions proposed in a Terraform plan?

terraform apply

3. What gcloud command initiates a VPC peering connection?

gcloud compute networks peerings create

4. In Google Cloud, what must be unique between two VPC networks to enable peering (i.e., they cannot overlap)?

Subnet ranges

5. What Google Cloud feature allows you to automatically schedule VM instances to start and stop, reducing costs when not in use?

Instance Schedules

6. Which Google Cloud CLI command lets you change the machine type of a VM instance for cost optimization?

gcloud compute instances set-machine-type
