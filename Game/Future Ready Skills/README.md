## Introduction to Convolutions with TensorFlow

1. Vertex AI > Workbench > cloudlearningservices (select) > Reset
2. Open JupyterLab (cloudlearningservices)
3. Notebook >Python 3
4. CLS_Vertex_AI_Intro_to_CNN-v1.0.0.ipynb (double click to open)
5. Run (menu) > Run All Cells

## Filtering Explores with LookML

- Develop > Development mode (toggle)
- Develop >  qwiklabs-ecommerce (project) > training_ecommerce.model
    - Save Changes
    - Validate LookML
    - Commit Changes & Push
    - Deploy to Production
 
```
...

explore: order_items {
  conditionally_filter: {
    filters: [created_date: "3 years"]
    unless: [users.id, users.state]
  }
  
  join: users {
    type: left_outer
    sql_on: ${order_items.user_id} = ${users.id} ;;
    relationship: many_to_one
  }

  ...
}

...
```

## VPC Networking: Cloud HA-VPN

```shell
ZONE_1=us-central1-a
ZONE_2=europe-west4-c
ZONE_3=us-central1-c

REGION_1="${ZONE_1%-*}"
REGION_2="${ZONE_2%-*}"

# Task 1. Cloud VPC setup

gcloud compute networks create vpc-demo --subnet-mode custom

gcloud beta compute networks subnets create vpc-demo-subnet1 \
	--network vpc-demo --range 10.1.1.0/24 --region $REGION_1
	
gcloud beta compute networks subnets create vpc-demo-subnet2 \
	--network vpc-demo --range 10.2.1.0/24 --region $REGION_2
	
gcloud compute firewall-rules create vpc-demo-allow-internal \
	--network vpc-demo \
	--allow tcp:0-65535,udp:0-65535,icmp \
	--source-ranges 10.0.0.0/8

gcloud compute firewall-rules create vpc-demo-allow-ssh-icmp \
    --network vpc-demo \
    --allow tcp:22,icmp

gcloud compute instances create vpc-demo-instance1 --zone $ZONE_1 --subnet vpc-demo-subnet1 --machine-type e2-medium

gcloud compute instances create vpc-demo-instance2 --zone $ZONE_2 --subnet vpc-demo-subnet2 --machine-type e2-medium

# Task 2. Simulate on-premises setup

gcloud compute networks create on-prem --subnet-mode custom

gcloud beta compute networks subnets create on-prem-subnet1 \
	--network on-prem --range 192.168.1.0/24 --region $REGION_1

gcloud compute firewall-rules create on-prem-allow-internal \
	--network on-prem \
	--allow tcp:0-65535,udp:0-65535,icmp \
	--source-ranges 192.168.0.0/16

gcloud compute firewall-rules create on-prem-allow-ssh-icmp \
    --network on-prem \
    --allow tcp:22,icmp

gcloud compute instances create on-prem-instance1 --zone $ZONE_3 --subnet on-prem-subnet1 --machine-type e2-medium

# Task 3. HA-VPN setup

gcloud beta compute vpn-gateways create vpc-demo-vpn-gw1 --network vpc-demo --region $REGION_1

gcloud beta compute vpn-gateways create on-prem-vpn-gw1 --network on-prem --region $REGION_1

gcloud compute routers create vpc-demo-router1 \
    --region $REGION_1 \
    --network vpc-demo \
    --asn 65001

gcloud compute routers create on-prem-router1 \
    --region $REGION_1 \
    --network on-prem \
    --asn 65002

## Create two VPN tunnels

gcloud beta compute vpn-tunnels create vpc-demo-tunnel0 \
    --peer-gcp-gateway on-prem-vpn-gw1 \
    --region $REGION_1 \
    --ike-version 2 \
    --shared-secret [SHARED_SECRET] \
    --router vpc-demo-router1 \
    --vpn-gateway vpc-demo-vpn-gw1 \
    --interface 0

gcloud beta compute vpn-tunnels create vpc-demo-tunnel1 \
    --peer-gcp-gateway on-prem-vpn-gw1 \
    --region $REGION_1 \
    --ike-version 2 \
    --shared-secret [SHARED_SECRET] \
    --router vpc-demo-router1 \
    --vpn-gateway vpc-demo-vpn-gw1 \
    --interface 1

gcloud beta compute vpn-tunnels create on-prem-tunnel0 \
    --peer-gcp-gateway vpc-demo-vpn-gw1 \
    --region $REGION_1 \
    --ike-version 2 \
    --shared-secret [SHARED_SECRET] \
    --router on-prem-router1 \
    --vpn-gateway on-prem-vpn-gw1 \
    --interface 0

gcloud beta compute vpn-tunnels create on-prem-tunnel1 \
    --peer-gcp-gateway vpc-demo-vpn-gw1 \
    --region $REGION_1 \
    --ike-version 2 \
    --shared-secret [SHARED_SECRET] \
    --router on-prem-router1 \
    --vpn-gateway on-prem-vpn-gw1 \
    --interface 1

## Create bgp peering for each tunnel

gcloud compute routers add-interface vpc-demo-router1 \
    --interface-name if-tunnel0-to-on-prem \
    --ip-address 169.254.0.1 \
    --mask-length 30 \
    --vpn-tunnel vpc-demo-tunnel0 \
    --region $REGION_1

gcloud compute routers add-bgp-peer vpc-demo-router1 \
    --peer-name bgp-on-prem-tunnel0 \
    --interface if-tunnel0-to-on-prem \
    --peer-ip-address 169.254.0.2 \
    --peer-asn 65002 \
    --region $REGION_1

gcloud compute routers add-interface vpc-demo-router1 \
    --interface-name if-tunnel1-to-on-prem \
    --ip-address 169.254.1.1 \
    --mask-length 30 \
    --vpn-tunnel vpc-demo-tunnel1 \
    --region $REGION_1

gcloud compute routers add-bgp-peer vpc-demo-router1 \
    --peer-name bgp-on-prem-tunnel1 \
    --interface if-tunnel1-to-on-prem \
    --peer-ip-address 169.254.1.2 \
    --peer-asn 65002 \
    --region $REGION_1

gcloud compute routers add-interface on-prem-router1 \
    --interface-name if-tunnel0-to-vpc-demo \
    --ip-address 169.254.0.2 \
    --mask-length 30 \
    --vpn-tunnel on-prem-tunnel0 \
    --region $REGION_1

gcloud compute routers add-bgp-peer on-prem-router1 \
    --peer-name bgp-vpc-demo-tunnel0 \
    --interface if-tunnel0-to-vpc-demo \
    --peer-ip-address 169.254.0.1 \
    --peer-asn 65001 \
    --region $REGION_1

gcloud compute routers add-interface on-prem-router1 \
    --interface-name if-tunnel1-to-vpc-demo \
    --ip-address 169.254.1.2 \
    --mask-length 30 \
    --vpn-tunnel on-prem-tunnel1 \
    --region $REGION_1

gcloud compute routers add-bgp-peer on-prem-router1 \
    --peer-name bgp-vpc-demo-tunnel1 \
    --interface if-tunnel1-to-vpc-demo \
    --peer-ip-address 169.254.1.1 \
    --peer-asn 65001 \
    --region $REGION_1

## Configure Firewall rules to allow traffic from the remote VPC

gcloud compute firewall-rules create vpc-demo-allow-subnets-from-on-prem \
    --network vpc-demo \
    --allow tcp,udp,icmp \
    --source-ranges 192.168.1.0/24

gcloud compute firewall-rules create on-prem-allow-subnets-from-vpc-demo \
    --network on-prem \
    --allow tcp,udp,icmp \
    --source-ranges 10.1.1.0/24,10.2.1.0/24

## Global routing with VPN

gcloud compute networks update vpc-demo --bgp-routing-mode GLOBAL
```

## Importing Data to a Firestore Database

```
REGION=us-east1

gcloud firestore databases create --location=$REGION

git clone https://github.com/rosera/pet-theory

cd pet-theory/lab01

npm install @google-cloud/firestore

npm install @google-cloud/logging

edit importTestData.js

npm install faker@5.5.3

edit createTestData.js

node createTestData 1000

# npm install csv-parse

node importTestData customers_1000.csv
```

importTestData.js

```
const csv = require('csv-parse');
const fs  = require('fs');
const { Firestore } = require("@google-cloud/firestore");
const { Logging } = require('@google-cloud/logging');

const logName = "pet-theory-logs-importTestData";

// Creates a Logging client
const logging = new Logging();
const log = logging.log(logName);

const resource = {
  type: "global",
};

function writeToDatabase(records) {
  ...
}

async function importCsv(csvFilename) {
  const parser = csv.parse({ columns: true, delimiter: ',' }, async function (err, records) {
    if (err) {
      console.error('Error parsing CSV:', err);
      return;
    }
    try {
      console.log(`Call write to Firestore`);
      await writeToFirestore(records);
      // await writeToDatabase(records);
      console.log(`Wrote ${records.length} records`);
	  // A text log entry
	  success_message = `Success: importTestData - Wrote ${records.length} records`;
	  const entry = log.entry(
		  { resource: resource },
		  { message: `${success_message}` }
	  );
	  log.write([entry]);
    } catch (e) {
      console.error(e);
      process.exit(1);
    }
  });

  await fs.createReadStream(csvFilename).pipe(parser);
}

if (process.argv.length < 3) {
  ...
}

async function writeToFirestore(records) {
  const db = new Firestore({  
    // projectId: projectId
  });
  const batch = db.batch()

  records.forEach((record)=>{
    console.log(`Write: ${record}`)
    const docRef = db.collection("customers").doc(record.email);
    batch.set(docRef, record, { merge: true })
  })

  batch.commit()
    .then(() => {
       console.log('Batch executed')
    })
    .catch(err => {
       console.log(`Batch error: ${err}`)
    })
  return
}

importCsv(process.argv[2]).catch(e => console.error(e));
```

createTestData.js

```
const fs = require('fs');
const faker = require('faker');
const { Logging } = require("@google-cloud/logging");

const logName = "pet-theory-logs-createTestData";

// Creates a Logging client
const logging = new Logging();
const log = logging.log(logName);

const resource = {
	// This example targets the "global" resource for simplicity
	type: "global",
};

function getRandomCustomerEmail(firstName, lastName) {
  ...
}

async function createTestData(recordCount) {
  const fileName = `customers_${recordCount}.csv`;
  var f = fs.createWriteStream(fileName);
  f.write('id,name,email,phone\n')
  for (let i=0; i<recordCount; i++) {
    const id = faker.datatype.number();
    const firstName = faker.name.firstName();
    const lastName = faker.name.lastName();
    const name = `${firstName} ${lastName}`;
    const email = getRandomCustomerEmail(firstName, lastName);
    const phone = faker.phone.phoneNumber();
    f.write(`${id},${name},${email},${phone}\n`);
  }
  console.log(`Created file ${fileName} containing ${recordCount} records.`);
  // A text log entry
  const success_message = `Success: createTestData - Created file ${fileName} containing ${recordCount} records.`;
  const entry = log.entry(
	  { resource: resource },
	  {
		  name: `${fileName}`,
		  recordCount: `${recordCount}`,
		  message: `${success_message}`,
	  }
  );
  log.write([entry]);
}

recordCount = parseInt(process.argv[2]);
if (process.argv.length != 3 || recordCount < 1 || isNaN(recordCount)) {
  ...
}

createTestData(recordCount);
```

## APIs Explorer: Cloud SQL

```
REGION=us-central1

gcloud sql instances create my-instance \
	--region=$REGION \
	--tier=db-n1-standard-1

gcloud sql databases create mysql-db \
	--instance=my-instance

cat > employee_info.csv << EOF
"Sean", 23, "Content Creator"
"Emily", 34, "Cloud Engineer"
"Rocky", 40, "Event coordinator"
"Kate", 28, "Data Analyst"
"Juan", 51, "Program Manager"
"Jennifer", 32, "Web Developer"
EOF

BUCKET_NAME=$DEVSHELL_PROJECT_ID-bucket

gsutil mb gs://$BUCKET_NAME

gsutil cp employee_info.csv gs://$BUCKET_NAME/

SERVICE_ACCOUNT=$(gcloud sql instances describe my-instance --format="value(serviceAccountEmailAddress)")

gcloud storage buckets add-iam-policy-binding gs://$BUCKET_NAME \
    --member=serviceAccount:$SERVICE_ACCOUNT \
    --role=roles/storage.admin
```

## Configuring Private Google Access and Cloud NAT

```
ZONE=us-west1-b
REGION="${ZONE%-*}"

gcloud config set compute/zone $ZONE
gcloud config set compute/region $REGION

gcloud compute networks create privatenet --subnet-mode=custom

gcloud compute networks subnets create privatenet-us --network=privatenet --region=$REGION --range=10.130.0.0/20

gcloud compute firewall-rules create privatenet-allow-ssh --network=privatenet --allow=tcp:22 --source-ranges=0.0.0.0/0

gcloud compute instances create vm-internal --zone=$ZONE --machine-type=e2-medium --network-interface=subnet=privatenet-us,no-address

gcloud compute instances create vm-bastion --zone=$ZONE --machine-type=e2-micro --network-interface=subnet=privatenet-us,address="" --scopes=https://www.googleapis.com/auth/compute,https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/trace.append

gsutil mb gs://$DEVSHELL_PROJECT_ID-bucket

gsutil cp gs://cloud-training/gcpnet/private/access.png gs://$DEVSHELL_PROJECT_ID-bucket

gcloud compute networks subnets update privatenet-us --enable-private-ip-google-access

gcloud compute routers create nat-router --network=privatenet --region=$REGION

gcloud compute routers nats create nat-config --router=nat-router --auto-allocate-nat-external-ips --nat-all-subnet-ip-ranges
```

## Building and Debugging Cloud Functions for Node.js

IDE

```
Functions Framework for Node.js.

gcloud auth login

mkdir ff-app && cd $_ 

npm init --y

npm install @google-cloud/functions-framework

cat > index.js << EOF
exports.validateTemperature = async (req, res) => {
 try {
   if (req.body.temp < 100) {
     res.status(200).send("Temperature OK \n");
   } else {
     res.status(200).send("Too hot \n");
   }
 } catch (error) {
   //return an error
   console.log("got error: ", error);
   res.status(500).send(error);
 }
};
EOF

# npx @google-cloud/functions-framework --target=validateTemperature

# curl -X POST http://localhost:8080 -H "Content-Type:application/json"  -d '{"temp":"50"}'

rm index.js

cat > index.js << EOF
exports.validateTemperature = async (req, res) => {
 try {
   if (!req.body.temp) {
     throw "Temperature is undefined \n";
   }
   if (req.body.temp < 100) {
     res.status(200).send("Temperature OK \n");
   } else {
     res.status(200).send("Too hot \n");
   }
 } catch (error) {
   //return an error
   console.log("got error: ", error);
   res.status(500).send(error);
 }
};
EOF

# npx --node-options=--inspect @google-cloud/functions-framework --target=validateTemperature

# Debug: Attach to Node Process

# curl -X POST http://localhost:8080 

PROJECT_ID=qwiklabs-gcp-03-100225e92ad7
REGION=us-central1
SERVICE_ACCOUNT=developer-sa@$PROJECT_ID.iam.gserviceaccount.com

gcloud config set project $PROJECT_ID

gcloud functions deploy validateTemperature \
  --trigger-http \
  --runtime nodejs20 \
  --gen2 \
  --allow-unauthenticated \
  --region $REGION \
  --service-account $SERVICE_ACCOUNT

# curl -X POST https://$REGION-$PROJECT_ID.cloudfunctions.net/validateTemperature -H "Content-Type:application/json"  -d '{"temp":"50"}'
```

## Getting started with Flutter Development

IDE

```
flutter create startup_namer

cd startup_namer

sed -i 's/Flutter Demo Home Page/Flutter is awesome\!'/g' lib/main.dart

# fwr
```
