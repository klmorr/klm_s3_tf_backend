# klm_s3_tf_backend

## Description

Creates AWS infrastrure for managing terraform state

resources created

| Name | Purpose |
|------|---------|
| s3 | bucket to store terraform state files |
| dynamodb table | table for managing terraform state locks |
| iam | role and policy for utilizing infrastructure |

### Getting started

clone this repo

```bash
https://github.com/klmorr/klm_s3_tf_backend.git
```

navigate to the src directory

```bash
cd src
```

run invoke_tf.py

```bash
python invoke_tf.py --action apply --client test --profile default --region us-east-1
```

### invoke_tf.py

invoke_tf.py is a python script with logic to manage executing terraform to create the backend infrastructure and manage its state. Imports class from python files in the py_modules directory

arguments

| name | switch | description |
|------|--------|-------------|
| action | -a | terraform action to perform, accepts **apply** or **destroy** |
| client | -c | owner for the terraform created resources |
| profile | -p | aws credential profile name |
| region | -r | aws account region |

Todo:

Add step to migrate state to local on destroy to remove error related to releasing state lock

Init process

- Sets environment variables from profile and region to set environment variables AWS_PROFILE and AWS_REGION for terraform auth- 

- Sets variable **resource_name** used for naming terraform resources from the client and region arguments, ex **client-us-east-1-tf-state**
  
- Checks if an s3 bucket and a dynamodb table named **resource_name** exists, creates the resources if they do not exists using boto3
  
- Generates a backend config file named s3.tfbackend using the **resource_name** and **region** arguments. ex:

```bash
bucket          = "client-us-east-1-tf-state"
dynamodb_table  = "client-us-east-1-tf-state"
key             = "state/terraform.tfstate"
region          = "us-east-1"
encrypt         = true
```

- Generates terraform.tfvars file using the **client** argument to be used by terraform, ex:

```bash
client = "client"
```

- If an existing terraform state was found, runs **terraform init** using the backend config file. If not state is found, runs **terraform init** using the backend, then imports the terraform module resources for the s3 bucket and the dynamodb table into the state

- Executes terrform using the **action** argument (apply or destroy)