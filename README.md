# Serverless-Contact-Form-API# Serverless Contact Form API  
A production-grade, fully serverless backend for processing contact form submissions. Built using AWS Lambda, API Gateway, DynamoDB, and SNS, this project securely stores user messages and sends real-time email notifications.  
## Technologies & Services  
- **API Gateway** – REST endpoint to receive contact form data  
- **AWS Lambda** – Stateless compute to validate, store, and trigger alerts  
- **DynamoDB** – NoSQL table to store form submissions with timestamps  
- **SNS (Simple Notification Service)** – Sends email alerts on new form submissions  
- **IAM Roles** – Secure permission delegation for Lambda access to other services  
## API Endpoint  
**Base URL:** `https://<api-id>.execute-api.<region>.amazonaws.com/prod`  
**Endpoint:** `POST /contact`  
**Headers:** `Content-Type: application/json`  
**Request Body:** `{"name": "Jane Doe", "email": "jane@example.com", "message": "Hi! I'm interested in your services."}`  
**Success Response:** `{"message": "Submission received!"}`  
**Error Response Example:** `{"error": "Missing required fields"}`  
## Lambda Function Overview  
```python  
import boto3, uuid, json  
from datetime import datetime  
dynamodb = boto3.resource('dynamodb')  
sns = boto3.client('sns')  
DDB_TABLE = 'ContactSubmissions'  
SNS_TOPIC_ARN = 'arn:aws:sns:us-east-1:<your-account-id>:<your-topic-name>'  
def lambda_handler(event, context):  
    if 'body' not in event or not event['body']:  
        return {"statusCode": 400, "body": json.dumps({"error": "Missing body in request"})}  
    try: body = json.loads(event['body'])  
    except json.JSONDecodeError:  
        return {"statusCode": 400, "body": json.dumps({"error": "Invalid JSON"})}  
    name, email, message = body.get("name"), body.get("email"), body.get("message")  
    if not all([name, email, message]):  
        return {"statusCode": 400, "body": json.dumps({"error": "Missing required fields"})}  
    item = {"submissionId": str(uuid.uuid4()), "name": name, "email": email, "message": message, "submittedAt": datetime.utcnow().isoformat()}  
    dynamodb.Table(DDB_TABLE).put_item(Item=item)  
    sns.publish(TopicArn=SNS_TOPIC_ARN, Subject="New Contact Form Submission", Message=f"Name: {name}\nEmail: {email}\nMessage: {message}")  
    return {"statusCode": 200, "body": json.dumps({"message": "Submission received!"})}  
