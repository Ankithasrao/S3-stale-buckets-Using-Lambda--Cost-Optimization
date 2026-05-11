# S3-stale-buckets-Using-Lambda--Cost-Optimization
This project demonstrates cost optimization by deleteing the stale s3 buckets by using a serverless service of AWS called AWS Lambda.

<img width="1536" height="1024" alt="S3-stale-bucket-using-lambda" src="https://github.com/user-attachments/assets/a47dd600-7450-47dd-87ef-e61ccc00ba2d" />

### The AWS services used for the projects are :
#### 1. S3 Buckets 
#### 2. AWS Lambda functions
#### 3. IAM User and Roles
#### 4. Cloud Watch
#### 5. SNS Topic
#### 6. Event scheduler

## Step 1 : Create a IAM user with required permisions and attach required inline policies.

## IAM ROLE inline policy
#### * "s3:ListAllMyBuckets",
#### * "s3:ListBucketVersions",
#### * "s3:ListBucket",
#### * "s3:DeleteBucket"
#### * "sns:Publish"

## IAM ROLE permissions :
#### * CloudWatchFullAccess
#### * CloudWatchFullAccessV2

## Step 2 : Create s3 buckets with and without uploading the objects into it. (Optional : in diffrent locations)

## Step 3 : Create AWS LAmbda function with the python code :
#### Lambda time out : 4 mins
#### Runtime python3.14

<img width="840" height="432" alt="s3-stale-bucket6" src="https://github.com/user-attachments/assets/35407626-7804-4110-8387-418cdbba9711" />

## Note : Explanation of python code 

### 🚀 1. Imports & Setup
```
import boto3   //AWS SDK (used to interact with S3, SNS, CloudWatch)
import json    //for dashboard creation
import time    //timestamps
from datetime import datetime, timedelta   //calculate age of buckets/objects
```

### ⚙️ 2. Configuration Section 
```
SNS_TOPIC_ARN = '...'  //Where alerts are sent
REGION = 'us-east-1'   //AWS region
DRY_RUN = False        //If True → simulate deletion
NOTIFY_ONLY = False    //If True → only alert, no deletion
```

#### S3 Filtering Logic
```
STALE_DAYS_THRESHOLD = 0  
EMPTY_BUCKETS_ONLY = True
CHECK_OBJECT_LAST_MODIFIED = True
```

### ☁️  3. AWS Clients
```
cloudwatch_main = boto3.client('cloudwatch', region_name=REGION)    //Send metrics → CloudWatch
sns = boto3.client('sns', region_name=REGION)                       //Send alerts → SNS
```

### 🧠 4. Function: is_bucket_stale() : This is the core logic of your script

#### 1. Get bucket objects
```
response = s3_client.list_objects_v2(Bucket=bucket_name, MaxKeys=1)     //Checks if bucket has any objects
```

#### 2. Check if empty
```
is_empty = 'Contents' not in response     //If no objects → bucket is empty
```

#### 3. If EMPTY_BUCKETS_ONLY = True
```
if is_empty:
    days_old = (datetime.now() - creation_date).days   //If bucket is empty, checks how old it is , if older than threshold it is stale
```

#### 4. If checking object age
```
last_modified = obj['LastModified']
days_old = (datetime.now() - last_modified).days   //Check last modified date, If recent - Not stale, If old - stale
```

### Error Handling
```
except Exception as e:    //Prevents crash if bucket access fails
```

### 🔁 5. Main Function: lambda_handler() : This is what AWS Lambda executes
```
s3 = boto3.client('s3')      //Create S3 client

total_buckets = 0
stale_buckets_global = 0     //Initialize counters

response = s3.list_buckets()  //List all buckets

for bucket in all_buckets:    //Loop through buckets

if is_bucket_stale(...):     //If bucket is stale

bucket_resource.objects.all().delete()
bucket_resource.object_versions.all().delete()
s3.delete_bucket(Bucket=bucket_name)             // Deleting Buckets (Important)
```

### 📊 6. Send Metrics to CloudWatch
```
cloudwatch_main.put_metric_data(...)     //Used for dashboards & monitoring
```

### 📈 7. Create Dashboard
```
widgets = [...]                 //Builds: Total bucket count widget,Stale bucket count widget, Text section (names + results)
```

### 📧  8. Send SNS Notification
```
sns.publish(...)                //Sends email/message
```

### 📊  9. Push Dashboard
```
cloudwatch_main.put_dashboard(...)     //Creates CloudWatch dashboard:Global-S3BucketDashboard
```

<img width="1027" height="680" alt="s3-stale-bucket4" src="https://github.com/user-attachments/assets/70d4351d-3a3b-41cb-ab2e-727d39bee71b" />


### ✅ 10. Final Response
```
return {
    'statusCode': 200,
    'body': ...
}                                 //Returns summary:Total buckets,stale buckets, and mode
```

## Step 4 : Create SNS topic 

<img width="937" height="551" alt="s3-stale-bucket2" src="https://github.com/user-attachments/assets/97b1f2dd-60cb-4268-83dd-9d1ccb33a672" />

<img width="872" height="507" alt="s3-stale-bucket3" src="https://github.com/user-attachments/assets/2c68afaa-af3d-43ab-bae9-0685d0ea2451" />

## Step 5 : Create Event scheduler : 00 10 ? JAN-DEC SUN *

<img width="1232" height="580" alt="s3-stale-bucket5" src="https://github.com/user-attachments/assets/daf8bedf-1a24-41f0-a910-a2b2bf863987" />

## Delete the resources after the project completion
#### Delete the lambda function
#### Delete the iam role
#### Delete the event bridge scheduler
#### Delete the sns topic
#### Delete the s3 buckets





