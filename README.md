# Project: Setting Up a Monitoring and Alert System in AWS

## Overview  
In this project, I demonstrate how to build a basic monitoring and alert system on AWS using **AWS CloudTrail**, **CloudWatch**, and **Simple Notification Service (SNS)**. The goal is to track access to sensitive secrets stored in **AWS Secrets Manager** and receive real-time email notifications when those secrets are accessed. This project also highlights how AWS security and monitoring services work together to improve visibility and incident response.

---

## 1. Set Up Secrets and Logging  
I began by creating a secret named `TopSecret` in **AWS Secrets Manager**. This secret stores a sample password in string/integer format.  
Secrets Manager is a secure AWS service for storing sensitive credentials like API keys, account IDs, and tokens—protecting them from unauthorized access.
![Top Secret Creation](cs/topsecret-creation.png)
---

## 2. Configure Monitoring with CloudTrail 
Next, I set up **AWS CloudTrail** to monitor activity in my AWS account. CloudTrail tracks API calls and records events related to resource access, which is essential for security audits, incident response, and troubleshooting.

For this project:
- I created a CloudTrail trail to **track management events**, which includes access to secrets.
- Both **read** and **write** API actions were enabled. Although accessing a secret is technically a read action, it is considered a **write API event** in CloudTrail due to its security implications.
- Management events are free to monitor and provide detailed logs about who accessed what, when, and how.
![CloudTrail Created](cs/cloudtrail-created.png)
![Event Type - Read & Write](cs/event-type-r-w.png)

---

## 3. Verifying CloudTrail Logs  
To verify that secret access is logged correctly, I accessed the `TopSecret` secret in two ways:
- Via the **Secrets Manager Console** (by clicking “Retrieve secret value”)
- Via the **AWS CLI** using the command:  
  ```bash
  aws secretsmanager get-secret-value --secret-id "TopSecret" --region us-east-1
  ```
![CloudShell CLI](cs/cloudshell-cli.png)

Then, I reviewed the **CloudTrail Event History**, where I confirmed the **GetSecretValue** event was recorded in both cases. This validated that CloudTrail is accurately monitoring secret access.
![CloudTrail Event History](cs/cloudtrail-event-history.png)
---

## 4. Analyzing Logs with CloudWatch  
With CloudTrail confirmed, I integrated **Amazon CloudWatch Logs** for deeper analysis. CloudWatch is ideal for storing logs long-term, creating custom metrics, and enabling advanced filtering.

Key steps:
- Sent CloudTrail logs to a CloudWatch Log Group  
- Created a **CloudWatch metric filter** to detect every `GetSecretValue` event  
- Configured the metric to increment by `1` each time the secret is accessed  
- Set the default value to `0` when the event is not found, for proper alarm evaluation
![CloudWatch Log Group](cs/cloudwatch-log-group.png)
![Metric Created](cs/cw-metric-created.png)
![CloudWatch Metric](cs/cloudwatch-metric.png)
---

## 5. Creating Alarms and Notifications with SNS  
I then created a **CloudWatch Alarm** based on the metric above. The alarm triggers if the secret is accessed **once or more within 5 minutes**.
![CloudWatch Alarm](cs/alarm-created.png)

To notify when this happens:
- I created an **SNS Topic** and subscribed my email address  
- Configured the alarm to send a notification to the SNS topic  
- Verified my email subscription (required by AWS for security)  
Once confirmed, I successfully received email alerts when the secret was accessed.
![Email Alert](cs/email-alert.png)
![SNS Subscription](cs/subscription-confirmed.png)

---

## 6. Enabling SNS Notifications in CloudTrail  
Finally, I enabled **SNS delivery from CloudTrail** as an additional notification method. This allows CloudTrail to directly publish events to SNS in real time. I confirmed its effectiveness by receiving alerts from both CloudTrail and CloudWatch when the secret was accessed.

---

## 🔍 Final Reflection  
This project used the following AWS services:  
- **Secrets Manager** (to store credentials)  
- **CloudTrail** (to log access events)  
- **CloudWatch** (to analyze logs and create alarms)  
- **SNS** (to send email alerts)  
- **IAM Roles & Policies** (for access control)  
- **S3 Bucket** (for CloudTrail log storage)  

Key concepts explored:  
- Secure credential storage  
- CloudTrail vs. CloudWatch use cases  
- Real-time notifications (email alerts)  
- Monitoring API activity and auditing access patterns  

**Time spent:** ~3 hours
