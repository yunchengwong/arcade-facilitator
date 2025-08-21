## Data Publishing on BigQuery for Data Sharing Partners

https://cloud.google.com/bigquery/docs/control-access-to-resources-iam

```shell
USER_1=student-01-90af51d979d7@qwiklabs.net
USER_2=student-01-90df41fda7c1@qwiklabs.net

# Task 1. Grant permissions via IAM for data access

gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID \
	--member=user:$USER_1 \
	--role=roles/bigquery.user

# Task 2. Create a new dataset within an existing project

bq --location=US mk -d demo_dataset

# Task 3. Copy an existing table to newly created dataset

bq cp bigquery-public-data:google_trends.top_terms $DEVSHELL_PROJECT_ID:demo_dataset.top_terms

# Task 4. Grant permission to the users to access the table

bq add-iam-policy-binding \
	--member=user:$USER_1 \
	--role=roles/bigquery.dataViewer \
	--table=true $DEVSHELL_PROJECT_ID:demo_dataset.top_terms

# Task 5. Authorize a dataset and grant permission to the users

bq show --format=prettyjson \
    $DEVSHELL_PROJECT_ID:demo_dataset > schema.json

edit schema.json

bq update --source schema.json \
	$DEVSHELL_PROJECT_ID:demo_dataset

# Task 6. Verify dataset sharing for customer projects 

# USER 1:

bq --location=US mk -d customer_1_dataset

bq mk --use_legacy_sql=false --view \
'SELECT * FROM `qwiklabs-gcp-03-5765b50979fb:demo_dataset.top_terms`' \
$DEVSHELL_PROJECT_ID:customer_1_dataset.customer_1_table

# USER 2:

bq --location=US mk -d customer_2_dataset

bq mk --use_legacy_sql=false --view \
'SELECT * FROM `qwiklabs-gcp-03-5765b50979fb:demo_dataset.top_terms`' \
$DEVSHELL_PROJECT_ID:customer_2_dataset.customer_2_table
```

```json
{
  "access": [
    {
      "role": "WRITER",
      "specialGroup": "projectWriters"
    },
    {
      "role": "OWNER",
      "specialGroup": "projectOwners"
    },
    {
      "role": "OWNER",
      "userByEmail": "student-01-8ab5971a93ca@qwiklabs.net"
    },
    {
      "role": "READER",
      "specialGroup": "projectReaders"
    },
    {
	  "dataset" : {
	    "dataset": {
	      "project_id": "qwiklabs-gcp-03-5765b50979fb",
	      "dataset_id": "demo_dataset"
	    },
	    "target_types": "VIEWS"
	  }
    },
    {
      "role": "roles/bigquery.user",
      "userByEmail": "student-01-90af51d979d7@qwiklabs.net"
    },
    {
      "role": "roles/bigquery.user",
      "userByEmail": "student-01-90df41fda7c1@qwiklabs.net"
    }
  ],
  ...
}
```

## Data Publishing on BigQuery using Authorized Views for Data Sharing Partners

```shell
# Task 1. Create authorized views

bq mk --use_legacy_sql=false --view \
"SELECT * FROM \`bigquery-public-data.geo_us_boundaries.zip_codes\`
WHERE state_code='TX'
LIMIT 4000" \
$DEVSHELL_PROJECT_ID:demo_dataset.authorized_view_a

bq mk --use_legacy_sql=false --view \
"SELECT * FROM \`bigquery-public-data.geo_us_boundaries.zip_codes\`
WHERE state_code='CA'
LIMIT 4000" \
$DEVSHELL_PROJECT_ID:demo_dataset.authorized_view_b

# Task 2. Assign IAM permissions to both the views

bq show --format=prettyjson $DEVSHELL_PROJECT_ID:demo_dataset > dataset_metadata.json

jq --arg project_id "$DEVSHELL_PROJECT_ID" '.access += [
    {
      "view": {
        "datasetId": "demo_dataset",
        "projectId": $project_id,
        "tableId": "authorized_view_a"
      }
    },
    {
      "view": {
        "datasetId": "demo_dataset",
        "projectId": $project_id,
        "tableId": "authorized_view_b"
      }
    },
    {
      "dataset": {
        "dataset": {
          "datasetId": "demo_dataset",
          "projectId": $project_id
        },
        "targetTypes": [
          "VIEWS"
        ]
      }
    }
  ]' dataset_metadata.json > updated_dataset_metadata.json

bq update --source updated_dataset_metadata.json $DEVSHELL_PROJECT_ID:demo_dataset

# Task 3. Grant permissions to the users to access the views

USER_A=student-01-05b472d4615f@qwiklabs.net
USER_B=student-01-6bad55e83be6@qwiklabs.net

bq add-iam-policy-binding \
	--member=user:$USER_A \
	--role=roles/bigquery.dataViewer \
	--table=true $DEVSHELL_PROJECT_ID:demo_dataset.authorized_view_a

bq add-iam-policy-binding \
	--member=user:$USER_B \
	--role=roles/bigquery.dataViewer \
	--table=true $DEVSHELL_PROJECT_ID:demo_dataset.authorized_view_b

# Task 4. Verify shared authorized views in customer projects

# USER 1:

bq --location=US mk -d customer_a_dataset

bq mk --use_legacy_sql=false --view \
'SELECT * FROM `qwiklabs-gcp-01-dade780aa35e:demo_dataset.authorized_view_a`' \
$DEVSHELL_PROJECT_ID:customer_a_dataset.customer_a_table

# USER 2:

bq --location=US mk -d customer_b_dataset

bq mk --use_legacy_sql=false --view \
'SELECT * FROM `qwiklabs-gcp-01-dade780aa35e:demo_dataset.authorized_view_b`' \
$DEVSHELL_PROJECT_ID:customer_b_dataset.customer_b_table
```

## Analytics as a Service for Data Sharing Partners

```shell
# Task 1. Create authorized views

bq mk --use_legacy_sql=false --view \
"SELECT * FROM \`bigquery-public-data.geo_us_boundaries.zip_codes\`
WHERE state_code='TX'
LIMIT 4000" \
$DEVSHELL_PROJECT_ID:demo_dataset.authorized_view_a

bq mk --use_legacy_sql=false --view \
"SELECT * FROM \`bigquery-public-data.geo_us_boundaries.zip_codes\`
WHERE state_code='CA'
LIMIT 4000" \
$DEVSHELL_PROJECT_ID:demo_dataset.authorized_view_b

# Task 2. Assign IAM permissions to both the views

bq show --format=prettyjson $DEVSHELL_PROJECT_ID:demo_dataset > dataset_metadata.json

jq --arg project_id "$DEVSHELL_PROJECT_ID" '.access += [
    {
      "view": {
        "datasetId": "demo_dataset",
        "projectId": $project_id,
        "tableId": "authorized_view_a"
      }
    },
    {
      "view": {
        "datasetId": "demo_dataset",
        "projectId": $project_id,
        "tableId": "authorized_view_b"
      }
    },
    {
      "dataset": {
        "dataset": {
          "datasetId": "demo_dataset",
          "projectId": $project_id
        },
        "targetTypes": [
          "VIEWS"
        ]
      }
    }
  ]' dataset_metadata.json > updated_dataset_metadata.json

bq update --source updated_dataset_metadata.json $DEVSHELL_PROJECT_ID:demo_dataset

# Task 3. Grant permissions to the users to access the views

USER_A=student-01-aca1c0171e9b@qwiklabs.net
USER_B=student-01-ad7d6e7fd6de@qwiklabs.net

bq add-iam-policy-binding \
	--member=user:$USER_A \
	--role=roles/bigquery.dataViewer \
	--table=true $DEVSHELL_PROJECT_ID:demo_dataset.authorized_view_a

bq add-iam-policy-binding \
	--member=user:$USER_B \
	--role=roles/bigquery.dataViewer \
	--table=true $DEVSHELL_PROJECT_ID:demo_dataset.authorized_view_b

# Task 4. Display insights for View A

bq --location=US mk -d customer_a_dataset

bq mk --use_legacy_sql=false --view \
"SELECT geos.zip_code, geos.city, cust.last_name, cust.first_name
FROM \`$DEVSHELL_PROJECT_ID.customer_a_dataset.customer_info\` as cust
JOIN \`qwiklabs-gcp-00-70a92ad7fcdb.demo_dataset.authorized_view_a\` as geos
ON geos.zip_code = cust.postal_code;" \
$DEVSHELL_PROJECT_ID:customer_a_dataset.customer_a_table

# https://lookerstudio.google.com/
# Create > Report (Customer A Visualization) > BigQuery > Authorize > PROJECT_ID:customer_a_dataset.customer_a_table
# Insert > Pie chart (city, Record Count)

# Task 5. Display insights for View B

bq --location=US mk -d customer_b_dataset

bq mk --use_legacy_sql=false --view \
"SELECT geos.zip_code, geos.city, cust.last_name, cust.first_name
FROM \`$DEVSHELL_PROJECT_ID.customer_b_dataset.customer_info\` as cust
JOIN \`qwiklabs-gcp-00-70a92ad7fcdb.demo_dataset.authorized_view_b\` as geos
ON geos.zip_code = cust.postal_code;" \
$DEVSHELL_PROJECT_ID:customer_b_dataset.customer_b_table

# https://lookerstudio.google.com/
# Create > Report (Customer B Visualization) > BigQuery > Authorize > PROJECT_ID:customer_a_dataset.customer_a_table
# Insert > Pie chart (city, Record Count)
```

## Consuming Customer Specific Datasets from Data Sharing Partners using BigQuery

https://cloud.google.com/bigquery/docs/tables

```shell
# Task 1. Created an authorized table

bq query \
--destination_table $DEVSHELL_PROJECT_ID:demo_dataset.authorized_table \
--use_legacy_sql=false \
"SELECT * FROM (
SELECT *, ROW_NUMBER() OVER (PARTITION BY state_code ORDER BY area_land_meters DESC) AS cities_by_area
FROM \`bigquery-public-data.geo_us_boundaries.zip_codes\`) cities
WHERE cities_by_area <= 10 ORDER BY cities.state_code
LIMIT 1000;"

bq show --format=prettyjson \
$DEVSHELL_PROJECT_ID:demo_dataset > dataset_metadata.json

jq --arg project_id "$DEVSHELL_PROJECT_ID" '.access += [
    {
      "dataset": {
        "dataset": {
          "datasetId": "demo_dataset",
          "projectId": $project_id
        },
        "targetTypes": [
          "VIEWS"
        ]
      }
    }
]' dataset_metadata.json > updated_dataset_metadata.json

bq update --source updated_dataset_metadata.json \
$DEVSHELL_PROJECT_ID:demo_dataset

PUBLISHER=student-01-c34b50a664f3@qwiklabs.net
CUSTOMER=student-01-c35930ae1458@qwiklabs.net

bq add-iam-policy-binding \
	--member=user:$PUBLISHER \
	--role=roles/bigquery.dataViewer \
	--table=true $DEVSHELL_PROJECT_ID:demo_dataset.authorized_table

bq add-iam-policy-binding \
	--member=user:$CUSTOMER \
	--role=roles/bigquery.dataViewer \
	--table=true $DEVSHELL_PROJECT_ID:demo_dataset.authorized_table

# Task 2. Create an authorized view in the Data Publishing project

# PUBLISHER:

SHARING_PROJECT_ID=qwiklabs-gcp-01-a9d3096bd92f

bq mk --use_legacy_sql=false --view \
'SELECT *
FROM `qwiklabs-gcp-01-a9d3096bd92f.demo_dataset.authorized_table`
WHERE state_code="NY"
LIMIT 1000' \
$DEVSHELL_PROJECT_ID:data_publisher_dataset.authorized_view

bq show --format=prettyjson \
$DEVSHELL_PROJECT_ID:data_publisher_dataset > view_metadata.json

jq --arg project_id "$DEVSHELL_PROJECT_ID" '.access += [
    {
      "view": {
        "datasetId": "data_publisher_dataset",
        "projectId": $project_id,
        "tableId": "authorized_view"
      }
    },
    {
      "dataset": {
        "dataset": {
          "datasetId": "data_publisher_dataset",
          "projectId": $project_id
        },
        "targetTypes": [
          "VIEWS"
        ]
      }
    }
]' view_metadata.json > updated_view_metadata.json

bq update --source updated_view_metadata.json \
$DEVSHELL_PROJECT_ID:data_publisher_dataset

CUSTOMER=student-01-c35930ae1458@qwiklabs.net

bq add-iam-policy-binding \
	--member=user:$CUSTOMER \
	--role=roles/bigquery.dataViewer \
	--table=true $DEVSHELL_PROJECT_ID:data_publisher_dataset.authorized_view

# Task 3. Access the authorized view as a Data Twin

# CUSTOMER:

PUBLISHER_PROJECT_ID=qwiklabs-gcp-01-c2611a3af0d3

bq mk --use_legacy_sql=false --view \
"SELECT cities.zip_code, cities.city, cities.state_code, customers.last_name, customers.first_name
FROM \`$DEVSHELL_PROJECT_ID.customer_dataset.customer_info\` as customers
JOIN \`$PUBLISHER_PROJECT_ID.data_publisher_dataset.authorized_view\` as cities
ON cities.state_code = customers.state;" \
$DEVSHELL_PROJECT_ID:customer_dataset.customer_table
```

## Share Data using Google Data Cloud: Challenge Lab

```shell
# Task 1. Create the partner authorized view (PARTNER)

PARTNER_VIEW=authorized_view_covh

bq mk --use_legacy_sql=false --view \
"SELECT * FROM \`bigquery-public-data.geo_us_boundaries.zip_codes\`;" \
$DEVSHELL_PROJECT_ID:demo_dataset.$PARTNER_VIEW

bq show --format=prettyjson \
$DEVSHELL_PROJECT_ID:demo_dataset > view_metadata.json

jq --arg project_id "$DEVSHELL_PROJECT_ID" '.access += [
    {
      "view": {
        "datasetId": "demo_dataset",
        "projectId": $project_id,
        "tableId": "$partner_view"
      }
    },
    {
      "dataset": {
        "dataset": {
          "datasetId": "demo_dataset",
          "projectId": $project_id
        },
        "targetTypes": [
          "VIEWS"
        ]
      }
    }
]' view_metadata.json > updated_view_metadata.json

bq update --source updated_view_metadata.json \
$DEVSHELL_PROJECT_ID:demo_dataset

CUSTOMER=student-01-b3b5677e351a@qwiklabs.net

bq add-iam-policy-binding \
	--member=user:$CUSTOMER \
	--role=roles/bigquery.dataViewer \
	--table=true $DEVSHELL_PROJECT_ID:demo_dataset.$PARTNER_VIEW

# Task 2. Update the customer data table (CUSTOMER)

PARTNER_VIEW=authorized_view_covh
PARTNER_PROJECT_ID=qwiklabs-gcp-03-402ef268dd64

bq query --use_legacy_sql=false \
"UPDATE
 \`$DEVSHELL_PROJECT_ID.customer_dataset.customer_info\` cust
SET
cust.county=vw.county
FROM
\`$PARTNER_PROJECT_ID.demo_dataset.$PARTNER_VIEW\` vw
WHERE
vw.zip_code=cust.postal_code;"

# Task 3. Create the customer authorized view (CUSTOMER)

CUSTOMER_VIEW=customer_authorized_view_244j

bq mk --use_legacy_sql=false --view \
"SELECT
  county,
COUNT(1) AS Count
FROM
 \`$DEVSHELL_PROJECT_ID.customer_dataset.customer_info\` cust
GROUP BY
 county
HAVING county is not null" \
$DEVSHELL_PROJECT_ID:customer_dataset.$CUSTOMER_VIEW

bq show --format=prettyjson \
$DEVSHELL_PROJECT_ID:customer_dataset > schema.json

jq --arg project_id "$DEVSHELL_PROJECT_ID" '.access += [
    {
      "view": {
        "datasetId": "customer_dataset",
        "projectId": $project_id,
        "tableId": "customer_authorized_view_244j"
      }
    },
    {
      "dataset": {
        "dataset": {
          "datasetId": "customer_dataset",
          "projectId": $project_id
        },
        "targetTypes": [
          "VIEWS"
        ]
      }
    }
]' schema.json > updated_schema.json

bq update --source updated_schema.json \
$DEVSHELL_PROJECT_ID:customer_dataset

PARTNER=student-01-5b664325699e@qwiklabs.net

bq add-iam-policy-binding \
	--member=user:$PARTNER \
	--role=roles/bigquery.dataViewer \
	--table=true $DEVSHELL_PROJECT_ID:customer_dataset.$CUSTOMER_VIEW

# Task 4. Use the customer authorized view to create a visualization (PARTNER)

# https://lookerstudio.google.com/
# Create > Report > BigQuery > Authorize > CUSTOMER_PROJECT_ID:customer_dataset.CUSTOMER_VIEW (MY PROJECTS)
```
