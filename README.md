# Assignment-2.14-Serverless-Architecture-2
Assignment 2.14 - Serverless Architecture 2 (SQS, SNS, Eventbridge)

Answer the following:
- 1. Does SNS guarantee exactly once delivery to subscribers?
- 2. What is the purpose of the Dead-letter Queue (DLQ)? This is a feature available to SQS/SNS/EventBridge.
- 3. How would you enable a notification to your email when messages are added to the DLQ?

# Reference
https://aws.amazon.com/sns/faqs/

# 1. Does SNS guarantee exactly-once delivery to subscribers?
- ANS: No, Amazon SNS (Simple Notification Service) does not guarantee exactly-once delivery. SNS is an at-least-once delivery service, meaning that a message may be delivered more than once to subscribers. This is because SNS is designed for high availability and durability, and it retries message delivery in case of transient failures, potentially leading to duplicate deliveries.

To handle duplicates, you can:
Use deduplication logic in the subscriber application.
Use FIFO SNS and FIFO SQS (available in certain regions) for exactly-once message processing.

# 2. What is the purpose of a Dead-letter Queue (DLQ)?
- ANS: A Dead-letter Queue (DLQ) is used to capture messages that fail to be processed after multiple delivery attempts. This feature helps prevent message loss and allows for debugging issues.

  - SNS DLQ: If SNS cannot successfully deliver a message to a subscribed endpoint (e.g., SQS, Lambda, HTTP), it moves the message to the DLQ.
  - SQS DLQ: If a message in an SQS queue fails processing multiple times (based on the maxReceiveCount in the redrive policy), it is sent to the DLQ.
  - EventBridge DLQ: If an EventBridge rule fails to deliver an event, it moves the event to the DLQ.

The DLQ allows you to inspect failed messages and take corrective actions.

# 3. How to enable email notifications when messages are added to the DLQ?
- ANS: To get email notifications when messages are added to the DLQ, follow these steps:

    **Step 1:**
    - Create an SNS Topic for Notifications
    - Go to the AWS SNS Console.
    - Create a new SNS topic (e.g., DLQ-Alerts).
    - Create an email subscription to this topic.

    **Step 2:**
    - Set Up an Alarm on the DLQ (Using CloudWatch)
    - Go to the AWS CloudWatch Console.
    - Navigate to Alarms → Create Alarm.
    - Select SQS Metrics → ApproximateNumberOfMessagesVisible for the DLQ.
    - Set a threshold (e.g., ≥1 message).
    - Choose SNS topic (DLQ-Alerts) as the alarm action.
    - Confirm the email subscription.

Now, whenever messages are added to the DLQ, you'll receive an email alert.

# Additional Information
# Dead Letter Queue (DLQ) in SNS, SQS, and EventBridge

Dead Letter Queues (DLQs) are used in **Amazon SNS, Amazon SQS, and Amazon EventBridge** to capture messages that fail processing. However, each service implements DLQs differently based on their architecture and use cases.

## Table of Contents
- [DLQ in Amazon SQS](#dlq-in-amazon-sqs)
- [DLQ in Amazon SNS](#dlq-in-amazon-sns)
- [DLQ in Amazon EventBridge](#dlq-in-amazon-eventbridge)
- [Comparison Table](#comparison-table)
- [Summary](#summary)

---

## DLQ in Amazon SQS

### **Purpose**
Captures messages that fail to be processed after multiple retry attempts.

### **How it Works**
- You configure a **redrive policy** for an SQS queue.
- Messages are moved to the DLQ when they exceed the **MaximumReceives** count.
- The DLQ is just another SQS queue.

### **Best Use Case**
For decoupled architectures where a consumer polls messages, ensuring no message is lost after failures.

### **Key Points**
- ✅ Messages are moved to DLQ after exceeding `maxReceiveCount`.
- ✅ Consumers can reprocess messages later.
- ✅ Useful for background job processing.

---

## DLQ in Amazon SNS

### **Purpose**
Captures failed message deliveries to SNS **subscription endpoints** (SQS, Lambda, HTTP, etc.).

### **How it Works**
- You specify an **SQS DLQ** in the SNS topic's delivery policy.
- Messages are moved to the DLQ if delivery to the primary endpoint fails.

### **Best Use Case**
When SNS fan-out messages need a backup mechanism for failed deliveries.

### **Key Points**
- ✅ Used only when the **SNS subscriber** (e.g., SQS, Lambda, HTTP) fails to process the message.
- ✅ Helps debug undelivered messages in pub-sub models.
- ✅ Works at the **subscription level**, meaning each subscriber can have its own DLQ.

---

## DLQ in Amazon EventBridge

### **Purpose**
Captures events that fail to be delivered to **EventBridge targets** (e.g., Lambda, SQS, Kinesis).

### **How it Works**
- You configure an SQS queue as a **dead-letter queue for an EventBridge rule**.
- If an event **fails all retry attempts**, it is sent to the DLQ.

### **Best Use Case**
When EventBridge rules need a fallback mechanism for undeliverable events.

### **Key Points**
- ✅ Captures failed event deliveries (e.g., due to permission errors, target unavailability).
- ✅ Useful for debugging and ensuring event-driven architectures don’t lose data.
- ✅ Helps in debugging **why EventBridge rules fail**.

---

## Comparison Table

| Feature         | SNS DLQ | SQS DLQ | EventBridge DLQ |
|---------------|--------|--------|----------------|
| **Purpose** | Capture failed **message delivery** to subscribers | Capture failed **message processing** | Capture failed **event delivery** |
| **Triggers Failure** | Subscriber (Lambda, SQS, HTTP) is unreachable | Consumer fails multiple times (`maxReceiveCount`) | Target (e.g., Lambda, SQS, API Gateway) fails |
| **DLQ Type** | SQS queue | SQS queue | SQS queue |
| **Use Case** | Fan-out messages, notify multiple systems | Decoupled architectures, workers processing jobs | Event-driven architectures, rule-based event delivery |
| **Retries** | Retry based on delivery policy | `maxReceiveCount` before sending to DLQ | Exponential backoff, then move to DLQ |

---

## Summary
- **Use SQS DLQ** if your consumers fail to process messages multiple times.
- **Use SNS DLQ** if an SNS subscriber (SQS, Lambda, HTTP, etc.) fails to receive messages.
- **Use EventBridge DLQ** if an event rule fails to deliver events to a target.

Each service handles retries and failures differently, so choose the right DLQ implementation based on your architecture. 🚀



