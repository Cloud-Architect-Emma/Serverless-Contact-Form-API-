# Serverless Contact Form API

A production-grade, secure, and scalable **serverless backend system** for handling website contact form submissions. This architecture is designed using core AWS services, ensuring high availability, low operational cost, and real-time notifications without server management.

---

## Project Goals

- Accept user submissions from a website contact form  
- Validate and store submissions in a persistent database  
- Notify the website owner via email in real-time  
- Ensure low-latency, high availability, and zero-maintenance backend

---

## GOFAT Analysis (Goal, Opportunity, Functional, Architecture, Technology)

| **Aspect**     | **Details** |
|----------------|-------------|
| **Goal**       | Enable website visitors to submit queries via a contact form with secure backend processing, real-time alerting, and long-term storage. |
| **Opportunity**| Serverless architecture reduces operational overhead, auto-scales, and eliminates the need for traditional web servers. |
| **Functional** | RESTful `POST /contact` endpoint for data intake, Lambda processing, DynamoDB persistence, SNS email notification. |
| **Architecture**| API Gateway → Lambda → DynamoDB (Store) + SNS (Notify). IAM Roles restrict and delegate access securely. |
| **Technology** | AWS Lambda, API Gateway (HTTP), DynamoDB, SNS, IAM. Written in Python 3. |

---

## Technologies & AWS Services

- **Amazon API Gateway** – RESTful interface to handle HTTP `POST /contact` requests  
- **AWS Lambda** – Stateless Python function to validate data, store records, and trigger alerts  
- **Amazon DynamoDB** – Serverless NoSQL database to persist contact form submissions  
- **Amazon SNS (Simple Notification Service)** – Real-time email notifications upon form submission  
- **IAM Roles & Policies** – Securely manage permissions across services

---

## API Endpoint

- **Base URL:** `https://<api-id>.execute-api.<region>.amazonaws.com/prod`  
- **Endpoint:** `POST /contact`  
- **Headers:** `Content-Type: application/json`

### Request Body Example

```json
{
  "name": "Jane Doe",
  "email": "jane@example.com",
  "message": "Hi! I'm interested in your services."
}
Success Response
json
Copy
Edit
{
  "message": "Submission received!"
}
Error Response Example
json
Copy
Edit
{
  "error": "Missing required fields"
}
Lambda Function Overview (Python)
python
Copy
Edit
import boto3, uuid, json
from datetime import datetime

dynamodb = boto3.resource('dynamodb')
sns = boto3.client('sns')

DDB_TABLE = 'ContactSubmissions'
SNS_TOPIC_ARN = 'arn:aws:sns:us-east-1:<your-account-id>:<your-topic-name>'

def lambda_handler(event, context):
    if 'body' not in event or not event['body']:
        return {"statusCode": 400, "body": json.dumps({"error": "Missing body in request"})}
    
    try:
        body = json.loads(event['body'])
    except json.JSONDecodeError:
        return {"statusCode": 400, "body": json.dumps({"error": "Invalid JSON"})}
    
    name, email, message = body.get("name"), body.get("email"), body.get("message")
    
    if not all([name, email, message]):
        return {"statusCode": 400, "body": json.dumps({"error": "Missing required fields"})}
    
    item = {
        "submissionId": str(uuid.uuid4()),
        "name": name,
        "email": email,
        "message": message,
        "submittedAt": datetime.utcnow().isoformat()
    }
    
    dynamodb.Table(DDB_TABLE).put_item(Item=item)
    
    sns.publish(
        TopicArn=SNS_TOPIC_ARN,
        Subject="New Contact Form Submission",
        Message=f"Name: {name}\nEmail: {email}\nMessage: {message}"
    )
    
    return {"statusCode": 200, "body": json.dumps({"message": "Submission received!"})}
System Architecture Diagram

Security & Permissions
Fine-grained IAM policies grant the Lambda function access only to:

Write to DynamoDB

Publish to SNS

Write logs to CloudWatch

No publicly exposed credentials or storage buckets

API Gateway exposes only a POST method to /contact route

Deployment Notes
Ensure Lambda has permission using lambda:InvokeFunction by API Gateway

Attach the following IAM managed or custom inline policies:

AmazonDynamoDBFullAccess

AmazonSNSFullAccess

CloudWatchLogsFullAccess

Use Postman or curl to test the POST /contact endpoint

Author
Cloud Architect Emma
GitHub Profile

License
This project is open-source and available under the MIT License
