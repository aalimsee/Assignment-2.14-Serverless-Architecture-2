# Assignment-2.14-Serverless-Architecture-2
Assignment 2.14 - Serverless Architecture 2 (SQS, SNS, Eventbridge)

Answer the following:
Does SNS guarantee exactly once delivery to subscribers?
What is the purpose of the Dead-letter Queue (DLQ)? This is a feature available to SQS/SNS/EventBridge.
How would you enable a notification to your email when messages are added to the DLQ?

1. Does SNS guarantee exactly-once delivery to subscribers?
No, Amazon SNS (Simple Notification Service) does not guarantee exactly-once delivery. SNS is an at-least-once delivery service, meaning that a message may be delivered more than once to subscribers. This is because SNS is designed for high availability and durability, and it retries message delivery in case of transient failures, potentially leading to duplicate deliveries.
To handle duplicates, you can:
Use deduplication logic in the subscriber application.
Use FIFO SNS and FIFO SQS (available in certain regions) for exactly-once message processing.
2. What is the purpose of a Dead-letter Queue (DLQ)?
A Dead-letter Queue (DLQ) is used to capture messages that fail to be processed after multiple delivery attempts. This feature helps prevent message loss and allows for debugging issues.
SNS DLQ: If SNS cannot successfully deliver a message to a subscribed endpoint (e.g., SQS, Lambda, HTTP), it moves the message to the DLQ.
SQS DLQ: If a message in an SQS queue fails processing multiple times (based on the maxReceiveCount in the redrive policy), it is sent to the DLQ.
EventBridge DLQ: If an EventBridge rule fails to deliver an event, it moves the event to the DLQ.
The DLQ allows you to inspect failed messages and take corrective actions.

3. How to enable email notifications when messages are added to the DLQ?
To get email notifications when messages are added to the DLQ, follow these steps:
Step 1: Create an SNS Topic for Notifications
Go to the AWS SNS Console.
Create a new SNS topic (e.g., DLQ-Alerts).
Create an email subscription to this topic.
Step 2: Set Up an Alarm on the DLQ (Using CloudWatch)
Go to the AWS CloudWatch Console.
Navigate to Alarms → Create Alarm.
Select SQS Metrics → ApproximateNumberOfMessagesVisible for the DLQ.
Set a threshold (e.g., ≥1 message).
Choose SNS topic (DLQ-Alerts) as the alarm action.
Confirm the email subscription.
Now, whenever messages are added to the DLQ, you'll receive an email alert.
