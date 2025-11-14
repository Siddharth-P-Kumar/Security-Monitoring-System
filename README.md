<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Build a Security Monitoring System

**Project Link:** [View Project](http://learn.nextwork.org/projects/aws-security-monitoring)

**Author:** siddharthpk nextwork  
**Email:** siddharthpknextwork@gmail.com

---

![Image](http://learn.nextwork.org/mischievous_red_fierce_pony/uploads/aws-security-monitoring_reghtjy)

---

## Introducing Today's Project!

In this project, I will demonstrate how to monitor secret access in AWS using logging and alerting tools. I'm doing this project to learn how to improve cloud security and get notified about unauthorized or unexpected access.

Stage 1: Set up the secret & logging
🔑 Step #1: Create a secret in AWS Secrets Manager.
🏞️ Step #2: Enable CloudTrail to log account activity.
😈 Step #3: Test the CloudTrail setup by accessing the secret and verifying a log is created.

Stage 2: Set up the monitoring & alert system
🔎 Step #4: Create a CloudWatch filter to detect secret access events.
🔔 Step #5: Configure a CloudWatch Alarm and SNS to send email alerts.
💌 Step #6: Test and troubleshoot the full monitoring pipeline.

### Tools and concepts

Services I used were AWS Secrets Manager, AWS CloudTrail, Amazon CloudWatch, Amazon SNS, Amazon S3, and the AWS CLI. Key concepts I learnt include how to securely store and manage secrets using Secrets Manager, how to track and log user activity with CloudTrail, and how to create alarms and monitor events using CloudWatch. I also learned how to set up automated notifications through SNS, store logs and data in S3, and interact with AWS services efficiently using the AWS CLI.

### Project reflection

This project took me approximately 2 hours to complete. The most challenging part was troubleshooting the notification setup to ensure that the email alerts triggered correctly after accessing the secret. It was most rewarding to see the end-to-end system work as expected — from monitoring activity to receiving real-time alerts — which gave me confidence in understanding how different AWS services integrate.

---

## Create a Secret

Secrets Manager is a service by AWS that securely stores, manages, and retrieves sensitive information such as database credentials, API keys, and other secrets. You could use Secrets Manager to protect this data by encrypting it, controlling access through IAM policies, and automatically rotating secrets to reduce security risks.


To set up for my project, I created a secret called topSecretInfo that contains a key/value pair, with the secret is' as the key and 'rice is the best carb' as the value. This secret will be used to test access monitoring and alerting in the later steps of the project.


![Image](http://learn.nextwork.org/mischievous_red_fierce_pony/uploads/aws-security-monitoring_o5p6q7r8)

---

## Set Up CloudTrail

CloudTrail is an AWS service that records actions taken by users, roles, or services in your AWS account, helping you monitor activity for security and compliance. I set up a trail to automatically collect and store these logs in an S3 bucket so I can track specific events—like when someone accesses a secret—and use that data for alerting and auditing.


CloudTrail events include types like management events, data events, insight events, and network activity events.

Management events log control-plane actions such as creating, modifying, or deleting AWS resources (e.g. launching EC2, creating a secret).

Data events capture high-volume operations like reading from or writing to S3 buckets or accessing secrets in Secrets Manager.

Insight events help detect unusual API behavior, such as sudden spikes in activity or errors.

Network activity events log details about network-level actions, such as VPC Flow Logs or DNS queries, helping you monitor traffic and detect potential security issues.

### Read vs Write Activity

Read API activity involves actions that retrieve or view data, such as reading a secret from Secrets Manager or listing resources.
Write API activity involves actions that change or create resources, like creating a secret, updating a configuration, or deleting a resource.
For this project, we need to monitor Read API activity because we're specifically interested in detecting when someone accesses (reads) our secret.

---

## Verifying CloudTrail

I retrieved the secret in two ways:

First, through the AWS Console by opening Secrets Manager, clicking on my secret (TopSecretInfo), and then clicking the “Retrieve secret value” button.

Second, using the AWS CLI by typing the command:
aws secretsmanager get-secret-value --secret-id "TopSecretInfo" --region ap-south-1

These two methods help test if CloudTrail logs both console and programmatic access to the secret.

To analyze my CloudTrail events, I visited the Event history tab of CloudTrail. I found two events with the event name GetSecretValue, which matched the times I accessed my secret through the console and the AWS CLI. This tells me that CloudTrail successfully logs secret access events, allowing me to track when and how secrets are being retrieved—an important step for monitoring and securing sensitive data.

![Image](http://learn.nextwork.org/mischievous_red_fierce_pony/uploads/aws-security-monitoring_s8t9u0v1)

---

## CloudWatch Metrics

CloudWatch Logs is an AWS service that collects, stores, and manages log data from AWS resources, applications, and services like CloudTrail. It's important for monitoring because it allows you to search, filter, and analyze log events in near real-time, helping you detect unusual activity, troubleshoot issues, and set up automated alerts based on specific patterns—like someone accessing a secret.

CloudTrail's Event History is useful for viewing recent events (last 90 days) in a simple, searchable format, making it great for quick audits or investigations.
While CloudWatch Logs are better for long-term storage, detailed analysis, real-time filtering, and triggering alarms, making them essential for setting up automated monitoring and alerting systems.

A CloudWatch metric is a numerical representation of data points over time, used to track and analyze specific events or system performance.

When setting up a metric, the metric value represents the number you want to assign to a matching event—in this case, 1 is used to signal that a secret was accessed.

Default value is used when no matching event is found during a given period—here, 0 means no access occurred. This setup helps CloudWatch determine when to trigger an alarm based on changes in event activity.

![Image](http://learn.nextwork.org/mischievous_red_fierce_pony/uploads/aws-security-monitoring_a9b0c1d2)

---

## CloudWatch Alarm

A CloudWatch alarm is a monitoring tool that watches a specific metric and performs actions when that metric meets a defined condition. I set my CloudWatch alarm threshold to static, with the condition set to trigger whenever the SecretIsAccessed metric is greater than or equal to 1. So the alarm will trigger when someone accesses the secret, indicating a potential security event that requires attention

I created an SNS topic along the way. An SNS topic is an Amazon Simple Notification Service resource that acts as a communication channel for sending messages to multiple subscribers. My SNS topic is set up to send a notification to my email whenever the CloudWatch alarm is triggered, so I’m immediately alerted when someone accesses my secret.

AWS requires email confirmation because it ensures that the recipient consents to receive notifications from the SNS topic. This helps prevent unauthorized subscriptions, spam, and unwanted messages, making sure that only verified and intended recipients get alerted.

![Image](http://learn.nextwork.org/mischievous_red_fierce_pony/uploads/aws-security-monitoring_fsdghstt)

---

## Troubleshooting Notification Errors

To test my monitoring system, I retrieved the secret value again to simulate access and trigger the CloudWatch alarm. The results were that the alarm detected the access event, and I received an email notification from SNS, confirming that my logging, metric filter, alarm, and alert system are all working correctly.

When troubleshooting the notification issues I:

Checked CloudTrail to make sure it recorded the GetSecretValue event—if the event didn’t show up, the secret access wasn't tracked.

Verified CloudTrail log delivery to CloudWatch Logs, ensuring the logs were actually reaching the log group connected to my metric filter.

Reviewed the CloudWatch metric filter to confirm it was correctly matching the log pattern related to secret access.

Tested the CloudWatch alarm to see if it was triggering when the metric value was ≥ 1 and properly configured to take action.

Checked SNS delivery, making sure my email subscription was confirmed, and the alarm was set to notify the correct topic and recipient.

I initially didn't receive an email before because the CloudWatch alarm wasn't properly configured to trigger the SNS topic. The key solution was to go back into the alarm settings and explicitly add the SNS topic as the action for the "In Alarm" state, ensuring that notifications are sent when the secret is accessed.

---

## Success!

To validate that my monitoring system works, I checked the alarm system manually by pushing an email notification using AWS CloudShell. I received the email after confirming that the setup was correct. Once I verified the configuration, I accessed my secrets through AWS Secrets Manager. About five minutes later, I received an automated email notification indicating that the secret had been accessed, confirming that the alarm system was successfully triggered.

![Image](http://learn.nextwork.org/mischievous_red_fierce_pony/uploads/aws-security-monitoring_ageraergearge)

---

