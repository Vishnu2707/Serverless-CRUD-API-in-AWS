# **Building a Serverless CRUD API to Simplify IT Infrastructure**

### **Overview**

In the modern IT infrastructure landscape, serverless architectures have emerged as a powerful paradigm. This guide demonstrates how to build a **serverless CRUD API** using **AWS Lambda**, **API Gateway**, and **DynamoDB**—eliminating the need for server management and simplifying infrastructure setup.

By the end of this guide, you'll have a fully functional CRUD API to perform **Create**, **Read**, **Update**, and **Delete** operations on a DynamoDB table. This solution showcases how serverless computing can streamline IT infrastructure for scalable and reliable applications.

---

### **Architecture**

![Architecture](https://github.com/user-attachments/assets/500b1e1f-1f41-4ea8-a401-ba778fbe6204)

1. **API Gateway**: Handles HTTP requests and routes them to AWS Lambda.
2. **AWS Lambda**: Executes the application logic for CRUD operations, interacting with DynamoDB.
3. **DynamoDB**: Serves as the storage backend for the CRUD operations.

---

### **Prerequisites**

1. **AWS Account** with permissions to create:
   - Lambda functions
   - API Gateway
   - DynamoDB tables
2. Familiarity with terminal commands (`curl`) to test HTTP requests.

---

### **Steps to Implement**

#### **Step 1: Set Up DynamoDB Table**

a. Open the **DynamoDB Console** and click **Create Table**.  
b. Configure the table as follows:
   - **Table Name**: `http-crud-api-items`
   - **Partition Key**: `id` (Data type: String)
c. Leave other settings as default and click **Create Table**.

---

#### **Step 2: Create a Lambda Function**

a. Open the **AWS Lambda Console** and click **Create Function**.  
b. Configure the function:
   - **Name**: `http-crud-api-function`
   - **Runtime**: Python 3.x
   - **Execution Role**: Create a new role with basic Lambda permissions.
c. Click **Create Function**.

d. Replace the default code with the following Python script:

```python
import json
import boto3
from decimal import Decimal

# Initialize DynamoDB
dynamo = boto3.resource('dynamodb').Table('http-crud-api-items')

class DecimalEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, Decimal):
            return float(obj)
        return super(DecimalEncoder, self).default(obj)

def lambda_handler(event, context):
    status_code = 200
    headers = {'Content-Type': 'application/json'}
    body = ""

    try:
        route_key = event['routeKey']
        if route_key == "DELETE /items/{id}":
            dynamo.delete_item(
                Key={'id': event['pathParameters']['id']}
            )
            body = f"Deleted item {event['pathParameters']['id']}"
        elif route_key == "GET /items/{id}":
            response = dynamo.get_item(
                Key={'id': event['pathParameters']['id']}
            )
            body = response.get('Item', {})
        elif route_key == "GET /items":
            response = dynamo.scan()
            body = response.get('Items', [])
        elif route_key == "PUT /items":
            request_json = json.loads(event['body'])
            dynamo.put_item(
                Item={
                    'id': request_json['id'],
                    'price': request_json['price'],
                    'name': request_json['name']
                }
            )
            body = f"Put item {request_json['id']}"
        else:
            raise ValueError(f"Unsupported route: {route_key}")
    except Exception as e:
        status_code = 400
        body = str(e)
    finally:
        body = json.dumps(body, cls=DecimalEncoder)

    return {
        'statusCode': status_code,
        'body': body,
        'headers': headers
    }
```

e. Click **Deploy** to save the changes.

---

#### **Step 3: Grant DynamoDB Access to Lambda**

a. Open the **IAM Console** and locate the role assigned to your Lambda function.  
b. Attach the following inline policy:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "dynamodb:DescribeTable",
                "dynamodb:PutItem",
                "dynamodb:GetItem",
                "dynamodb:DeleteItem",
                "dynamodb:Scan"
            ],
            "Resource": "arn:aws:dynamodb:<region>:<account-id>:table/http-crud-api-items"
        }
    ]
}
```

c. Replace `<region>` and `<account-id>` with your AWS region and account ID.

---

#### **Step 4: Create an HTTP API in API Gateway**

a. Open the **API Gateway Console** and click **Create API**.  
b. Choose **HTTP API** and click **Build**.  
c. Add the following routes:
   - `PUT /items`
   - `GET /items`
   - `GET /items/{id}`
   - `DELETE /items/{id}`
d. For each route:
   - Select **Integration Target** as **Lambda Function**.
   - Choose the function `http-crud-api-function`.
e. Deploy the API:
   - Click **Deploy API** and name the stage `$default`.
   - Note the **Invoke URL** (e.g., `https://abc123xyz.execute-api.<region>.amazonaws.com`).

---

#### **Step 5: Test the API**

1. Export the API URL:
   ```bash
   export INVOKE_URL="https://<invoke-url>"
   ```

2. Perform CRUD operations:

- **Create an Item (PUT)**:
  ```bash
  curl -X "PUT" -H "Content-Type: application/json" -d '{
      "id": "item123",
      "price": 100,
      "name": "Test Item"
  }' $INVOKE_URL/items
  ```

- **Retrieve All Items (GET)**:
  ```bash
  curl -X "GET" $INVOKE_URL/items
  ```

- **Retrieve a Specific Item (GET)**:
  ```bash
  curl -X "GET" $INVOKE_URL/items/item123
  ```

- **Delete an Item (DELETE)**:
  ```bash
  curl -X "DELETE" $INVOKE_URL/items/item123
  ```

---

### **Key Benefits for IT Infrastructure**

1. **No Server Management**:
   - Completely serverless. No need to provision, manage, or maintain servers.
2. **Cost Efficiency**:
   - Pay only for the compute time (Lambda) and data storage/queries (DynamoDB).
3. **Scalability**:
   - Automatically handles high traffic without manual intervention.
4. **Simplicity**:
   - API Gateway, Lambda, and DynamoDB provide an intuitive stack for modern IT applications.

---

### **Optional Enhancements**

1. Add **CORS headers** to allow cross-origin requests.
2. Implement authentication with **API Gateway Authorizers** or **AWS Cognito**.
3. Use **CloudWatch Logs** for debugging and monitoring usage patterns.

---

This guide provides a practical approach to leveraging serverless architectures in IT infrastructure, offering a scalable and cost-efficient solution for CRUD operations.
