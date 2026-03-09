# AWS Lambda EBS Cleanup Automation

## Project Name

AWS Lambda Automated EBS Cleanup

## Overview

This project automatically identifies and deletes **unused (available) EBS volumes** in AWS using an AWS Lambda function.

Unused EBS volumes continue to incur charges even when they are not attached to EC2 instances. This automation helps organizations reduce unnecessary storage costs by periodically cleaning up unused volumes.

The Lambda function scans all EBS volumes in the region and deletes volumes that are:

* In **available** state (not attached to any instance)
* **Untagged volumes**

The script also supports a **DRY_RUN mode**, which allows testing the logic without actually deleting resources.

---

## Architecture

User / CloudWatch Event
↓
AWS Lambda Function
↓
AWS EC2 API
↓
Identify Unused EBS Volumes
↓
Delete Volumes (if DRY_RUN = False)

---

## Features

* Automatically detects unused EBS volumes
* Deletes untagged volumes
* Cost optimization
* DRY_RUN mode for safe testing
* Serverless automation using AWS Lambda
* Can be scheduled using CloudWatch Events / EventBridge

---

## Technologies Used

* AWS Lambda
* AWS EC2
* Python (Boto3)
* CloudWatch / EventBridge
* IAM

---

## Lambda Python Script

```python
import boto3

# Set to False for real deletion
DRY_RUN = False

# Use Lambda's current region automatically
ec2 = boto3.resource('ec2')

def lambda_handler(event, context):

    print("EBS Cleanup Lambda Started")

    for vol in ec2.volumes.all():

        print(f"Checking volume {vol.id} | State: {vol.state}")

        if vol.state == 'available':

            if not vol.tags:
                if DRY_RUN:
                    print(f"[DRY RUN] Would delete untagged volume {vol.id}")
                else:
                    print(f"Deleting untagged volume {vol.id}")
                    vol.delete()

            else:
                print(f"Volume {vol.id} has tags, skipping")
```

---

## IAM Role Required

Attach the following permissions to the Lambda execution role.

### IAM Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeVolumes",
        "ec2:DeleteVolume"
      ],
      "Resource": "*"
    }
  ]
}
```

---

## Deployment Steps

### 1. Create Lambda Function

* Go to AWS Console
* Navigate to **Lambda**
* Click **Create Function**
* Choose **Author from scratch**
* Runtime: **Python 3.x**

---

### 2. Upload the Python Script

Paste the provided script into the Lambda editor.

---

### 3. Configure IAM Role

Attach the policy allowing:

* `ec2:DescribeVolumes`
* `ec2:DeleteVolume`

---

### 4. Configure Timeout

Set timeout to **1–3 minutes** depending on number of volumes.

---

### 5. Test Lambda

Create a **test event** and run the function.

Check **CloudWatch Logs** to verify results.

---

## Scheduling Automation

You can automate the cleanup using **EventBridge (CloudWatch Events)**.

Example schedule:

Run every day:

```
cron(0 2 * * ? *)
```

This runs the cleanup **daily at 2 AM UTC**.

---

## DRY_RUN Mode

Before enabling deletion, test the script.

```
DRY_RUN = True
```

Output example:

```
[DRY RUN] Would delete untagged volume vol-12345
```

Once verified:

```
DRY_RUN = False
```

---

## Use Cases

* Cloud cost optimization
* Automated infrastructure housekeeping
* Dev/Test environment cleanup
* Prevent accumulation of unused EBS volumes

---

## Future Improvements

* Delete volumes older than specific days
* Skip volumes with specific tags
* Send alerts via SNS before deletion
* Generate cost reports
* Multi-region scanning

---

## Author

Pruthvi Mandeep
DevOps Engineer

---

## License

This project is open-source and can be used for learning and automation purposes.
