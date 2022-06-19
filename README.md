# CloudWatch – How to send Lyve Cloud Audit Logs to AWS CloudWatch


## Introduction
This integration solution sends Lyve Cloud API Audit Log events to be consumed and displayed in AWS CloudWatch.
The procedure will continuously send Lyve Cloud API Audit Logs to AWS CloudWatch. AWS CloudWatch collects monitoring and operational data in the form of logs, metrics, and events and provides you with actionable insights to improve your IT operations using S3 storage.

### How it Works
The integration runs each round hour only. Whenever the integration is initialized, a cron job will run it in the closest round hour.
Each iteration it pulls from Lyve Cloud the necessary logs, and sends them in groups of 100 logs to the destination.
If the first iteration pulled either all existing logs or logs from a specific date, the next iteration will pull new logs only from the latest hour.


### Intergration Modes
You will have to [specify the mode](#step-4-set-up-your-environment) in which the intergation will run. You can choose one of the folliwing modes:
1. Pull and upload all existing logs. At the end of the process, every hour the integration will collect and upload new logs only.
2. Pull and upload existing logs from a specific date. At the end of the process, every hour the integration will collect and upload new logs only.
3. Every hour, scan for new logs only and upload them to CloudWatch. (Without uploading logs created before the running time of the integration)

### Logging
In order to monitor this integrations activity, it will create and post logs to a bucket in lyve cloud as specified by user in [config.json](#step-4-set-up-your-environment). On a successful pull and upload attempt, it will log a message "Done sending x logs". In case of a failure, it will log the failure cause. 

## Requirements
Before you start, please make sure you have these requirements and information in place:
* [`Python 3`](https://www.python.org/downloads/) and above is required.
* Docker.
* Lyve Cloud access and secret keys. These can be obtained from the Lyve Cloud Console by creating a new Service Account to be used by this solution.
* Bucket which containts audit logs and a bucket to which the integration will do logging. Both must be under the same access policy.
* IAM user with CloudWatchAgentServerPolicy to AWS CloudWatch.

## Known Limitations
This repository provides sample code to show how API Audit Logs can be pulled from Lyve Cloud into AWS CloudWatch, but it’s by no means a complete solution. 
There are limitations and functionality gaps to handle before this sample code can be used in a production environment:
* This example requires full access permissions to AWS CloudWatch.
* This integration solution sequentially processes API audit logs from a single tenant.
* In case of a crash, there's no implemented mechanism that recovers lost logs that weren't uploaded during the crash period.

## Running Steps
### Step 1: Get your Lyve Cloud bucket credentials for use with CloudWatch.
Here's what you'll need:
* Access Key
* Secret key
* Region

### Step 2: Generate AWS keys for CloudWatch
1. Login to AWS and go over to [IAM](https://console.aws.amazon.com/iamv2/home).
2. Click on the "Users" tab.
3. Click "Add Users".
4. Enter any username you would like. Under the "Select AWS access type" select "Access key - Programmatic access".
5. In the permissions page, select "Attach existing policies directly".
6. Search for "CloudWatchFullAccess" and select it. **Warning: this policy will provide access to all CloudWatch. To limit access, you can read more about IAM policies for CloudWatch [here](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/iam-identity-based-access-control-cw.html)**.
7. Click "Next: Tags".
8. Click "Next: Review".
9. Click "Create user".
10. Your keys are now generated, copy the "Access Key ID" and "Secret access key". These will be used to upload the logs into CloudWatch.

### Step 3: Create CloudWatch log stream
1. In the AWS console go over to CloudWatch. **Important: notice the region you are on and using. It is critical for configuration.**
2. Select "Log groups".
3. Click on "Create log group".
4. Provide a name for your log group.
5. Click on "Create". **Save this name! You will need it for configuration**.

### Step 4: Set up your environment
1. Edit config.json file
    * bucket_name - The name of the bucket from which the logs will be taken.
    * output_bucket - The name of the bucket to which the integration logs will be written.
    * log_type - The type of logs which should be taken (S3/IAM/Console).
    * mode - The mode in which the integration will run (lasthour/dd/mm/yyyy).
    * log_group - Name of the log group in CloudWatch

2. Run `docker build -f Dockerfile -t logscollector .`
3. Run
      ```bash
   docker run -d \
	-e LYVE_ACCESS=<lyve cloud access key> \
	-e LYVE_SECRET=<lyve cloud secret key> \
	-e LYVE_REGION=<lyve cloud region> \
	-e AWS_ACCESS=<aws access key> \
	-e AWS_SECRET=<aws secret key> \
	-e AWS_REGION=<aws region> \
 	logscollector
      ```

### Step 5: Set up CloudWatch dashboard
1. Go to AWS CloudWatch.
2. Select "Dashboards".
3. Click "Create Dashboard".
4. Name your dashboard and click "Create dashboard".
5. A popup will appear, if you would like to create your own dashboard, you can do so from here. If you would like to use a template, choose from the `dashboards` and continue from here.
6. Click "Cancel".
7. open the dashboard file you would like to use under and replace the fields written with `<>`.
8. Click on "Actions" -> "View/Edit source".
9. Delete anything in the textboxt and copy the source from the dashboard file.

## Tested by:
* December 12, 2021: Avi Wolicki on Linux
* December 12, 2021: Yinnon Hadad on Linux
* March 23, 2022: Bari Arviv (bari.arviv@seagate.com) on CentOS

### Project Structure

This section will describe the representation of each of the folders or files in the structure.
```
.
├── README.md
├── dashboards
│   └── requests_count.json
├── documentation
│   └── demo.mp4
│   └── introduction.pptx
├── services
│   └── pull_logs_service.py
│   └── upload_cloudwatch_logs_service.py
├── utils
│    └── json_extractor.py
├── requirements.txt
├── engine.py
├── config.json
└── .gitignore
```

### `/dashboards`
This folder contains examples of dashboards that can be imported to CloudWatch.

### `/services`
This folder contains the scripts that are used for fetching and uploading the logs.

### `/utils`
This folder contains scripts that are used as helpers for the scripts.

### `/images`
Contains images for the documentation.
