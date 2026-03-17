# AWS API Gateway

## Table of Contents

1. [Introduction](#introduction)
2. [API Gateway Basics](#api-gateway-basics)
3. [API Methods and HTTP Status Codes](#api-methods-and-http-status-codes)
4. [Creating Your First API Gateway](#creating-your-first-api-gateway)
5. [HTTP API vs REST API](#http-api-vs-rest-api)
6. [Proxy and Non-Proxy Lambda Integration](#proxy-and-non-proxy-lambda-integration)
7. [Request Validators](#request-validators)
8. [CloudWatch Logging](#cloudwatch-logging)
9. [Resource Policies](#resource-policies)
10. [Authorizers](#authorizers)

---

## Introduction

### What Will You Learn?

This comprehensive guide covers AWS API Gateway from beginner to production-ready level:

- **Beginner Level**: Understanding basic concepts and creating simple APIs
- **Advanced Level**: Configuring production-ready APIs with security, logging, and validation

### Prerequisites

- AWS Account with appropriate permissions
- Postman (free tool for testing APIs)
- Basic understanding of AWS Lambda
- Familiarity with AWS Console

---

## API Gateway Basics

### What is an API?

**API** stands for **Application Programming Interface**.

An API is a messenger that allows secure and scalable communication between different software systems and enables data exchange.

#### Real-World Analogy

Think of an API like a waiter in a restaurant:

1. **Customer (Client)** places an order
2. **Waiter (API)** takes the order to the kitchen
3. **Chef (Backend Service)** prepares the food
4. **Waiter (API)** returns the food to the customer

Similarly:
- Client sends a request to the API
- API forwards the request to the backend service
- Backend service processes the request
- API returns the response back to the client

### What is AWS API Gateway?

AWS API Gateway is a fully managed service that enables you to:

- **Create** RESTful APIs and WebSocket APIs
- **Publish** your APIs securely
- **Maintain** and monitor API health
- **Scale** automatically based on demand
- **Secure** your APIs with authentication and authorization

### What Can You Access Through API Gateway?

API Gateway acts as a gateway to access various AWS services:

- **Data** - Access and update data in databases (RDS, DynamoDB)
- **Business Logic** - Run custom code through Lambda functions
- **Backend Services** - EC2, S3, containers, and more

### Where is API Gateway Used?

APIs are everywhere in modern applications:

- **Weather Applications** - Multiple apps share weather data through APIs
- **Healthcare Systems** - Hospitals share patient data securely
- **Government Systems** - Public data access
- **Ticket Booking** - Flights, trains, buses all use APIs for bookings

---

## API Methods and HTTP Status Codes

### HTTP Methods (API Types)

API Gateway supports the following HTTP methods:

#### 1. GET Request

**Purpose**: Fetch information from the backend system

- **Request**: Sends a request to retrieve data
- **Response**: Returns the requested data (usually in JSON format) with HTTP 200 status code
- **No Payload Required**: You don't send a body with GET requests

**Example Flow**:
```
Client → GET /users/123 → Backend → Returns user data → Client receives data
```

#### 2. POST Request

**Purpose**: Create/Store new information in the backend system

- **Request**: Sends data (payload) to be stored
- **Payload**: Contains the data you want to save
- **Response**: Returns HTTP 200 or 201 status code confirming storage
- **No Data Returned**: You don't get back the stored data

**Example Flow**:
```
Client → POST /users (with user data) → Backend → Stores data → Returns 200 OK
```

#### 3. PUT Request

**Purpose**: Replace/Update entire resource

- **Request**: Sends complete replacement data
- **Payload**: Must contain all fields of the resource
- **Response**: Returns HTTP 200 confirming update
- **Full Replacement**: Replaces the entire resource

#### 4. PATCH Request

**Purpose**: Partially update an existing resource

- **Request**: Sends only the fields to be updated
- **Payload**: Contains only changed fields
- **Response**: Returns HTTP 200 confirming partial update
- **Partial Update**: Only specified fields are modified

**Example**: If you want to change only the email of a user, use PATCH instead of PUT

#### 5. DELETE Request

**Purpose**: Remove/Delete information from the backend system

- **Request**: Specifies which resource to delete
- **Response**: Returns HTTP 200 confirming deletion
- **Data Removed**: The resource is permanently removed

**Example Flow**:
```
Client → DELETE /users/123 → Backend → Deletes user → Returns 200 OK
```

#### 6. HEAD Request

**Purpose**: Same as GET but without the response body

- **Use Case**: Check if a resource exists without downloading data

#### 7. OPTIONS Request

**Purpose**: Describe the communication options for the target resource

- **Use Case**: CORS (Cross-Origin Resource Sharing) preflight requests

### HTTP Status Codes

Status codes indicate the result of your API request:

#### 2xx - Success

- **200 OK**: Request processed successfully
- **201 Created**: Resource created successfully (usually with POST)
- **204 No Content**: Request successful but no content to return

#### 3xx - Redirection

- **301 Moved Permanently**: Resource permanently moved
- **302 Found**: Temporary redirect

#### 4xx - Client Error

- **400 Bad Request**: Invalid request format or parameters
- **401 Unauthorized**: Authentication required or failed
- **403 Forbidden**: Authenticated but not authorized to access
- **404 Not Found**: Resource doesn't exist
- **422 Unprocessable Entity**: Request validation failed

#### 5xx - Server Error

- **500 Internal Server Error**: Backend service error
- **503 Service Unavailable**: Service temporarily unavailable

---

## Creating Your First API Gateway

### Architecture

We'll create a simple flow:

```
Client → API Gateway (GET) → Lambda Function → Response (JSON, 200 OK) → Client
```

### Step 1: Create the Lambda Function

#### 1.1 Navigate to Lambda

1. Open AWS Console
2. Search for "Lambda" in the search box
3. Click on **Lambda service**

#### 1.2 Create Function

1. Click **Create function**
2. Enter function name: `api-gateway-lambda`
3. Select Runtime: **Python 3.10** (or your preferred language)
4. Click **Create function**

#### 1.3 Update Lambda Code

Replace the default code with:

```python
import json

def lambda_handler(event, context):
    """
    A simple Lambda function that handles a GET request from the API gateway.
    """
    # This is the data you want to return.
    name = "World"
    if event.get('queryStringParameters'):
        name = event.get('queryStringParameters').get('name', name)

    response_data = {
        "message": f"Hello, Mallick! Your request was successful.",
        "status": "success"
    }

    # This is the required response format for API Gateway.
    # The 'body' must be a string, so we use json.dumps to convert our response data to a JSON string.
    return {
        "statusCode": 200,
        "headers": {"Content-Type": "application/json"},
        "body": json.dumps(response_data)
    }
```

**Explanation**:
- `statusCode`: HTTP status code (200 = OK)
- `headers`: Response headers (Content-Type = JSON)
- `body`: The actual response data

#### 1.4 Deploy Lambda

1. Click **Deploy** button
2. Wait for deployment to complete

#### 1.5 Test Lambda Function

1. Click on the **Test** tab
2. Click **Create test event**
3. Event name: `test-event`
4. Keep the JSON payload empty: `{}`
5. Click **Save**
6. Click **Test**

**Expected Result**:
- Status code: 200
- Response body shows your message
- Execution status: Succeeded

---

### Step 2: Create API Gateway

#### 2.1 Navigate to API Gateway

1. Search for "API Gateway" in AWS Console
2. Click **API Gateway service**

#### 2.2 Create API

1. Click **Create API**
2. Choose **HTTP API** (for simplicity, we'll start with this)
3. Click **Build**

#### 2.3 Configure API

1. **API Name**: `first-api-gateway-with-get`
2. Click **Next**
3. Click **Next** again (keep default stage settings)
4. Click **Create**

Your API is now created!

---

### Step 3: Create Route and Integrate Lambda

#### 3.1 Create Route

1. In the API dashboard, click **Create**
2. **Request type**: Select **GET**
3. Click **Create**

Your GET route is now created.

#### 3.2 Attach Lambda Integration

1. Click on the **GET** route
2. Click **Attach integration**
3. Click **Create and attach an integration**
4. **Integration target**: Select **Lambda function**
5. **Lambda Function**: Select your `api-gateway-lambda` (if not visible, ensure you're in the same region)
6. Click **Create**

Your Lambda is now connected to the API!

---

### Step 4: Get and Test API URL

#### 4.1 Find Invoke URL

1. In the API Gateway dashboard, find the **Invoke URL**
2. Copy this URL

#### 4.2 Test via Browser

1. Open a new browser tab
2. Paste the Invoke URL
3. Press Enter

**Expected Result**: You should see the JSON response with your message

#### 4.3 Test via Postman (Better Approach)

1. Open Postman
2. Create new request
3. Method: **GET**
4. Paste the Invoke URL
5. Click **Send**

**Expected Result**: 
- Status: 200 OK
- Response body shows your JSON message

---

## HTTP API vs REST API

### Quick Comparison Table

| Feature | HTTP API | REST API |
|---------|----------|----------|
| **Performance** | Faster | Slightly slower |
| **Cost** | Cheaper | More expensive |
| **Simplicity** | Very simple | More complex |
| **Request/Response Mapping** | Not supported | Fully supported |
| **API Keys** | Not supported | Supported |
| **OAuth** | Supported | Not supported |
| **Stage Variables** | Not supported | Supported |
| **CORS** | Native support | Manual configuration |
| **Custom Domain** | Supported | Supported |
| **Caching** | Not supported | Supported |
| **Use Case** | Simple, modern APIs | Enterprise, complex APIs |

### When to Use HTTP API

- Modern serverless applications
- Simple microservices
- Cost-conscious projects
- OpenID Connect / OAuth2 authentication needed
- Quick development

### When to Use REST API

- Complex request/response transformation
- API Keys for authorization
- Caching requirements
- Stage variables needed
- Legacy integrations
- Enterprise requirements
- Full control over mapping templates

### Decision Tree

```
Is your API simple and modern?
├─ YES → Use HTTP API
└─ NO → Need request/response mapping?
    ├─ YES → Use REST API
    └─ NO → Use HTTP API
```

---

## Proxy and Non-Proxy Lambda Integration

### Understanding the Difference

#### Non-Proxy Integration

**Non-proxy** means API Gateway can modify the request before sending it to Lambda and modify the response before sending it to the client.

```
Client Request → [API Gateway Modifies Request] → Lambda → [API Gateway Modifies Response] → Client
```

**Advantages**:
- Full control over request/response
- Can add/modify headers
- Can transform request/response data
- Validation at API Gateway level

**Disadvantages**:
- More complex configuration
- Requires mapping templates

#### Proxy Integration

**Proxy** means Lambda receives the entire request as-is and returns the entire response as-is.

```
Client Request → Lambda → Client Response
```

**Advantages**:
- Simpler setup
- Lambda has full control
- All transformation logic in Lambda
- Flexible

**Disadvantages**:
- Lambda must handle all formatting
- Less control at API Gateway level
- Higher Lambda execution overhead

---

### Non-Proxy Integration Deep Dive

#### Scenario

We'll create an API that:
1. Accepts a `name` parameter
2. Adds a prefix to the name at API Gateway level
3. Sends modified data to Lambda
4. Returns the result

#### Step 1: Create Non-Proxy Lambda

##### 1.1 Create Function

1. Go to Lambda
2. Click **Create function**
3. Name: `non-proxy-lambda-processor`
4. Runtime: **Python 3.10**
5. Click **Create function**

##### 1.2 Add Code

```python
import json

def lambda_handler(event, context):
    # Safe extraction of query parameter
    name = (
        event.get('queryStringParameters', {}).get('name') or
        event.get('name', 'World')
    )

    return {
        "statusCode": 200,
        "headers": {"Content-Type": "application/json"},
        "body": json.dumps({"message": f"Hello, {name}!"})
    }
```

or

```python
import json

def lambda_handler(event, context):
    name = event.get('name')

    if not name:
        params = event.get('queryStringParameters', {})
        name = params.get('name')

    return {
        "statusCode": 200,
        "headers": {"Content-Type": "application/json"},
        "body": f"Hello, {name}!"
    }
```

##### 1.3 Deploy and Test

1. Click **Deploy**
2. Go to **Test** tab
3. Create test event with:
```json
{
    "name": "Rahul"
}
```
4. Click **Test**
5. Should return: `Hello Rahul`

---

#### Step 2: Create REST API with Non-Proxy Integration

##### 2.1 Create API

1. Go to API Gateway
2. Click **Create API**
3. Select **REST API**
4. Click **Build**
5. Name: `non-proxy-api-gateway`
6. Click **Create API**

##### 2.2 Create Resource

1. Click **Resources**
2. Click **Create resource**
3. Resource name: `nonproxy-demo-api`
4. Click **Create resource**

##### 2.3 Create POST Method

1. Click on the resource you just created
2. Click **Create method**
3. Select **POST**
4. Integration type: **Lambda function**
5. **DO NOT enable** "Lambda Proxy Integration" (this is key for non-proxy!)
6. Lambda Function: Select `non-proxy-lambda-processor`
7. Click **Create method**

##### 2.4 Configure Method Request

The method request is where we define what parameters we expect from the client.

1. Click on **POST** method
2. Click **Method Request** (at the bottom)
3. Click **Edit**
4. **Request Validator**: Select `Validate body, query string parameters, and headers`
5. Expand **URI Query String Parameters**
6. Add parameter: `name` (Required)
7. Click **Save**

##### 2.5 Configure Integration Request (This is where the magic happens!)

Integration request is where we map/transform the incoming request before sending to Lambda.

1. Go back to the **POST** method
2. Click **Integration Request** (scroll down)
3. Click **Edit**
4. Scroll down to **Mapping Templates**
5. Click **Add mapping template**
6. Content-Type: `application/json`
7. Click the checkmark
8. In the template body, add:

```json
{
    "name": "$input.params('name')",
    "prefix": "non-proxy-prefix-"
}
```

**Explanation**:
- `$input.params('name')`: Gets the `name` query parameter from the request
- We're adding a `prefix` field
- This data gets sent to Lambda

9. Click **Save**

##### 2.6 Test in API Gateway

1. Click back on the **POST** method
2. Click **Test**
3. In Query Strings, enter: `name=Rahul`
4. Click **Test**
5. Response should show: `Hello Rahul`

##### 2.7 Deploy API

1. Click **Deploy API**
2. Stage: Create new stage called `dev`
3. Click **Deploy**
4. Copy the Invoke URL

##### 2.8 Test with Postman

1. Open Postman
2. Create new request
3. Method: **POST**
4. URL: `{invoke-url}/nonproxy-demo-api?name=Rahul`
5. Click **Send**
6. Should see response with your name

**The Power of Non-Proxy Integration**: You modified the request at the API Gateway level without touching Lambda code!

---

### Proxy Integration Deep Dive

#### Scenario

Similar to non-proxy, but Lambda handles all the transformation.

#### Step 1: Create Proxy Lambda

##### 1.1 Create Function

1. Name: `proxy-lambda-processor`
2. Runtime: **Python 3.10**
3. Click **Create function**

##### 1.2 Add Code

```python
def lambda_handler(event, context):
    """
    Proxy integration - Lambda receives full request and returns full response
    """
    try:
        # In proxy integration, we receive the full request object
        # We need to parse query parameters ourselves
        query_params = event.get('queryStringParameters', {})
        name = query_params.get('name', 'Unknown') if query_params else 'Unknown'
        
        return {
            'statusCode': 200,
            'headers': {
                'Content-Type': 'application/json'
            },
            'body': {
                'message': f'Hello {name}',
                'status': 'success'
            }
        }
    except Exception as e:
        return {
            'statusCode': 500,
            'headers': {
                'Content-Type': 'application/json'
            },
            'body': {
                'error': str(e)
            }
        }
```

##### 1.3 Deploy

1. Click **Deploy**

---

#### Step 2: Create REST API with Proxy Integration

##### 2.1 Create API

1. Go to API Gateway
2. Click **Create API**
3. Select **REST API**
4. Name: `proxy-api-gateway`
5. Click **Create API**

##### 2.2 Create Resource

1. Click **Create resource**
2. Name: `proxy-demo-api`
3. Click **Create resource**

##### 2.3 Create POST Method with Proxy Integration

1. Click **Create method**
2. Select **POST**
3. Integration type: **Lambda function**
4. **ENABLE** "Lambda Proxy Integration" ✓ (This is the key difference!)
5. Lambda Function: `proxy-lambda-processor`
6. Click **Create method**

##### 2.4 Configure Method Request

1. Click **Method Request**
2. Click **Edit**
3. Request Validator: `Validate body, query string parameters, and headers`
4. Add parameter `name` (Required)
5. Click **Save**

##### 2.5 Deploy API

1. Click **Deploy API**
2. Stage: Create new `dev` stage
3. Click **Deploy**

##### 2.6 Test with Postman

1. Open Postman
2. Method: **POST**
3. URL: `{invoke-url}/proxy-demo-api?name=Rahul`
4. Click **Send**

**Notice**: You get the full response object including statusCode and headers.

---

### Key Differences in Practice

**Non-Proxy Request/Response Cycle**:
```
Client → API Gateway (maps data using template) → Lambda (receives clean data) → Lambda (returns just data) → API Gateway (formats response) → Client
```

**Proxy Request/Response Cycle**:
```
Client → API Gateway (passes through) → Lambda (receives full request, must handle all formatting) → Lambda (returns full response object) → Client
```

---

## Request Validators

### Why Use Validators?

Imagine you're receiving user registration data:

```json
{
    "name": "John Doe",
    "email": "john@example.com",
    "mobile": "9876543210"
}
```

**Without Validators**:
- Invalid emails get to Lambda
- Lambda wastes time processing bad data
- Lambda costs money to execute unnecessary functions
- Inconsistent error messages

**With Validators**:
- Invalid requests rejected at API Gateway
- Save Lambda execution costs
- Consistent validation rules
- Better security

---

### Creating a Validation Model

#### Model Schema Example

```json
{
    "type": "object",
    "properties": {
        "name": {
            "type": "string",
            "minLength": 1,
            "maxLength": 100,
            "description": "User's full name"
        },
        "email": {
            "type": "string",
            "format": "email",
            "description": "Valid email address"
        },
        "mobile": {
            "type": "string",
            "pattern": "^[0-9]{10}$",
            "description": "10-digit mobile number"
        }
    },
    "required": ["name", "email", "mobile"],
    "additionalProperties": false
}
```

**Explanation of Validators**:

- **type**: Data type (string, number, integer, boolean, object, array)
- **minLength**: Minimum string length
- **maxLength**: Maximum string length
- **format**: Special format validation (email, date-time, uuid, ipv4, etc.)
- **pattern**: Regular expression pattern
- **minimum**: Minimum numeric value
- **maximum**: Maximum numeric value
- **required**: Fields that must be present
- **additionalProperties**: Reject extra fields if false

---

### Implementation Steps

#### Step 1: Create the API

1. Go to API Gateway
2. Click **Create API**
3. Select **REST API**
4. Name: `request-validator-api`
5. Click **Create API**

#### Step 2: Create Resource and Method

1. Click **Create resource**
2. Name: `user-registration`
3. Click **Create resource**
4. Click **Create method** → **POST**
5. Integration type: **Lambda**
6. Use existing non-proxy lambda
7. Click **Create method**

#### Step 3: Create the Validation Model

1. In left sidebar, click **Models**
2. Click **Create model**
3. Model name: `demo-request-validator`
4. Content-Type: `application/json`
5. In Schema field, paste your model schema (shown above)
6. Click **Create**

#### Step 4: Enable Validation on Method

1. Click on your **POST** method
2. Click **Method Request**
3. Click **Edit**
4. **Request Validator**: Select `Validate body, query string parameters, and headers`
5. Expand **Request Body**
6. Click **Add model**
7. Content-Type: `application/json`
8. Model: Select `demo-request-validator`
9. Click **Save**

#### Step 5: Deploy

1. Click **Deploy API**
2. Stage: `dev` (create new)
3. Click **Deploy**

#### Step 6: Test Validation

**Test 1: Valid Request**

In Postman:
```
Method: POST
URL: {invoke-url}/user-registration
Body (raw JSON):
{
    "name": "John Doe",
    "email": "john@example.com",
    "mobile": "9876543210"
}
```

Click **Send** → Should succeed with 200 OK

**Test 2: Invalid Email**

Change body to:
```json
{
    "name": "John Doe",
    "email": "invalid-email",
    "mobile": "9876543210"
}
```

Click **Send** → Should get 400 Bad Request: "Invalid request body"

**Test 3: Missing Field**

```json
{
    "name": "John Doe",
    "email": "john@example.com"
}
```

Click **Send** → Should get validation error (mobile is missing)

**Test 4: Invalid Mobile Format**

```json
{
    "name": "John Doe",
    "email": "john@example.com",
    "mobile": "123"
}
```

Click **Send** → Should get validation error (doesn't match pattern)

---

### Cost Savings Example

Assume:
- Lambda cost: $0.20 per million requests
- Daily invalid requests: 10,000 (without validation)
- Invalid requests waste $0.002 per day
- Monthly waste: $0.06

**With validators**: All invalid requests blocked at API Gateway (pennies to reject)

---

## CloudWatch Logging

### Why Enable Logging?

**Production Issues**:
- API suddenly starts failing
- Slow response times
- Mysterious 500 errors
- Need to debug quickly

**Without Logging**:
- Can't see what's happening
- Blind troubleshooting
- Hours of debugging

**With Logging**:
- Full request/response visibility
- Error tracking
- Performance metrics
- Quick issue resolution

---

### Architecture

```
API Gateway → CloudWatch Logs Enabled → IAM Role → CloudWatch Log Groups → Log Streams
```

---

### Step-by-Step Implementation

#### Step 1: Create IAM Role for API Gateway

##### 1.1 Navigate to IAM

1. Go to **IAM** service in AWS
2. Click **Roles** in left sidebar
3. Click **Create role**

##### 1.2 Select Trusted Entity

1. Service: **API Gateway**
2. Click the checkbox for API Gateway
3. Click **Next**

##### 1.3 Add Permissions

1. A permission policy `AmazonAPIGatewayPushToCloudWatchLogs` should be selected by default
2. Click **Next**

##### 1.4 Name and Create

1. Role name: `APIGatewayCloudWatchLogsRole`
2. Click **Create role**

##### 1.5 Copy Role ARN

1. Search for the role you just created: `APIGatewayCloudWatchLogsRole`
2. Click on it
3. Copy the **ARN** value (looks like: `arn:aws:iam::123456789:role/APIGatewayCloudWatchLogsRole`)
4. Save this for next step

---

#### Step 2: Associate Role with API Gateway Account Settings

**Important**: This is done ONCE per AWS account, not per API.

##### 2.1 Navigate to API Gateway Settings

1. Go to **API Gateway** service
2. In left sidebar, find **Settings** (at the bottom, might need to scroll)
3. Click **Settings**

##### 2.2 Configure CloudWatch Logs

1. In the **CloudWatch Logs** section, click **Edit**
2. **CloudWatch Log Role ARN**: Paste the ARN you copied above
3. Click **Save**

---

#### Step 3: Enable Logging for Your Specific API

##### 3.1 Navigate to Your API

1. Go to API Gateway
2. Select your API from the list
3. Click **Stages** in left sidebar

##### 3.2 Enable Logging on Stage

1. Click on your stage (e.g., `dev`)
2. Click **Logs, Tracing, and Metrics** tab
3. Click **Edit**
4. Toggle **CloudWatch Settings** to enable
5. **Log Level**: Select one of:
   - `ERROR` - Only errors
   - `INFO` - Errors and info (recommended for development)
   - `ERROR, INFO and Logs` - Most detailed
6. Click the checkboxes for:
   - ✓ Log full request/response data
   - ✓ Log data transformation (optional)
7. For **Data Trace Enabled**: Toggle ON (shows request/response bodies)
8. Click **Save Changes**

---

#### Step 4: Access and View Logs

##### 4.1 Find Your API ID

1. Still on your API stage
2. At the top, you'll see your **API ID**
3. Copy this ID

##### 4.2 Navigate to CloudWatch

1. Go to **CloudWatch** service
2. Click **Log Groups** in left sidebar
3. Search for your API ID
4. You should see a log group named: `/aws/apigateway/{API-ID}`
5. Click on it

##### 4.3 View Log Streams

1. You'll see **Log Streams**
2. Each log stream contains logs from your API calls
3. Click on a log stream to see details

##### 4.4 Understanding Logs

Typical log entry includes:

```
Request ID: xyz123
Authorization Type: NONE
Status: 200
Response Time: 45ms
Request Body: {parameters...}
Response Body: {response...}
Error: (if any)
```

---

### Interpreting Common Logs

**Successful Request**:
```
Method: POST
Status: 200
Message: Successful request processing
Response Time: 23ms
```

**Validation Error**:
```
Status: 400
Error: Invalid request body
Message: Field 'email' must be valid email format
```

**Authorization Failed**:
```
Status: 403
Error: User is not authorized
Message: Authorization token invalid
```

**Lambda Error**:
```
Status: 500
Error: Lambda execution failed
Message: Timeout or internal Lambda error
```

---

### Cost Implications

CloudWatch Logs pricing:
- Ingestion: $0.50 per GB
- Storage: $0.03 per GB per month

**Estimate**: An API with 100,000 requests/day with full logging ≈ $20-30/month

---

## Resource Policies

### What is a Resource Policy?

A **Resource Policy** controls **who** can access your API Gateway based on:

- IP addresses
- AWS account IDs
- VPC IDs
- IP ranges (CIDR blocks)

It acts as a firewall at the API Gateway level.

```
Request from Client IP 192.168.1.1 → [Resource Policy Check] → If Allowed: Forward to API, If Denied: Reject
```

---

### Use Cases

1. **Restrict to Corporate Network**: Only allow requests from company IP range
2. **Block Specific Countries**: Deny certain IP ranges
3. **Multi-Tenant Access**: Allow only specific AWS accounts
4. **VPC-Only Access**: Restrict to requests from specific VPC
5. **Development Restriction**: Block production URLs from test IPs

---

### Finding Your Public IP

1. Go to `whatismyip.com` or similar service
2. Note your public IP address (e.g., `203.0.113.45`)

---

### Implementation Steps

#### Step 1: Navigate to Resource Policy

1. Go to API Gateway
2. Select your API
3. In left sidebar, find **Resource Policy**
4. Click **Create policy**
5. Choose template: **IP range deny** (for our example)

#### Step 2: Understand the Template

Default template:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "execute-api:Invoke",
            "Resource": "*"
        },
        {
            "Effect": "Deny",
            "Principal": "*",
            "Action": "execute-api:Invoke",
            "Resource": "execute-api:/*",
            "Condition": {
                "IpAddress": {
                    "aws:SourceIp": "203.0.113.45"
                }
            }
        }
    ]
}
```

**Explanation**:
- First statement: Allow everyone (default open)
- Second statement: Deny specific IP address
- **Important**: Deny always takes precedence over Allow

#### Step 3: Customize the Policy

**Example 1: Deny Single IP**

```json
{
    "Effect": "Deny",
    "Principal": "*",
    "Action": "execute-api:Invoke",
    "Resource": "execute-api:/*",
    "Condition": {
        "IpAddress": {
            "aws:SourceIp": "203.0.113.45"
        }
    }
}
```

**Example 2: Deny IP Range (CIDR)**

```json
{
    "Effect": "Deny",
    "Principal": "*",
    "Action": "execute-api:Invoke",
    "Resource": "execute-api:/*",
    "Condition": {
        "IpAddress": {
            "aws:SourceIp": "203.0.113.0/24"
        }
    }
}
```

**Example 3: Allow Only Specific IP**

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "execute-api:Invoke",
            "Resource": "execute-api:/*",
            "Condition": {
                "IpAddress": {
                    "aws:SourceIp": ["203.0.113.45", "203.0.113.46"]
                }
            }
        },
        {
            "Effect": "Deny",
            "Principal": "*",
            "Action": "execute-api:Invoke",
            "Resource": "execute-api:/*"
        }
    ]
}
```

**Example 4: Allow Only Specific AWS Account**

```json
{
    "Effect": "Allow",
    "Principal": {
        "AWS": "arn:aws:iam::123456789012:root"
    },
    "Action": "execute-api:Invoke",
    "Resource": "execute-api:/*"
}
```

---

#### Step 4: Apply Policy

1. Paste your policy in the editor
2. Click **Save changes**
3. Click **OK** to confirm

---

#### Step 5: Deploy API

**Critical**: After applying resource policy, you MUST redeploy:

1. Go back to **Resources**
2. Click **Deploy API**
3. Select your stage (e.g., `dev`)
4. Click **Deploy**
5. Wait a few seconds for policy to activate

---

#### Step 6: Test the Policy

**Before testing**: Wait 10-30 seconds for changes to propagate

**Test 1: Without Policy (Before Applying)**

In Postman:
```
URL: {invoke-url}/path
Send Request → Should get 200 OK
```

**Test 2: With Deny Policy**

After deploying:
```
URL: {invoke-url}/path
Send Request → Should get error: "User is not authorized to perform: execute-api:Invoke"
```

**Test 3: From Different Network**

If you have access to a different IP (different WiFi, VPN, etc.):
- Connect to different network
- Same request → Should work (because policy denies your specific IP, not all IPs)

---

### Advanced Scenarios

#### Scenario 1: Corporate Network + VPN

Allow office network and company VPN:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "execute-api:Invoke",
            "Resource": "execute-api:/*",
            "Condition": {
                "IpAddress": {
                    "aws:SourceIp": [
                        "203.0.113.0/24",    # Office network
                        "198.51.100.0/24"    # VPN network
                    ]
                }
            }
        },
        {
            "Effect": "Deny",
            "Principal": "*",
            "Action": "execute-api:Invoke",
            "Resource": "execute-api:/*"
        }
    ]
}
```

#### Scenario 2: Multi-Tenant Restriction

Allow only specific AWS accounts to invoke your API:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": [
                    "arn:aws:iam::111111111111:root",
                    "arn:aws:iam::222222222222:root"
                ]
            },
            "Action": "execute-api:Invoke",
            "Resource": "execute-api:/*"
        },
        {
            "Effect": "Deny",
            "Principal": "*",
            "Action": "execute-api:Invoke",
            "Resource": "execute-api:/*"
        }
    ]
}
```

---

## Authorizers

### What is an Authorizer?

An **Authorizer** is a mechanism to **authenticate and authorize** requests to your API before they reach your backend service.

**Without Authorizer**: API is public, anyone with URL can access
**With Authorizer**: Only users with valid credentials can access

---

### Flow

```
Client Request (with Authorization Token)
        ↓
[API Gateway Authorizer Lambda]
        ↓
    Validates Token
        ↓
        ├─ Valid Token → Generate ALLOW policy → Request proceeds to Lambda
        └─ Invalid Token → Generate DENY policy → Request rejected (403 Forbidden)
```

---

### Types of Authorizers

1. **Lambda Authorizer** (Custom) - Your custom validation logic
2. **Cognito Authorizer** - AWS Cognito user pools
3. **OpenID Connect** - Third-party identity provider
4. **OAuth 2.0** - OAuth flow

We'll focus on **Lambda Authorizer** as it's the most flexible.

---

### Creating a Lambda Authorizer

#### Step 1: Create Authorizer Lambda Function

##### 1.1 Create Function

1. Go to Lambda
2. Click **Create function**
3. Name: `api-authorizer`
4. Runtime: **Python 3.10**
5. Click **Create function**

##### 1.2 Add Authorization Logic

```python
import json

def lambda_handler(event, context):
    """
    Authorizer Lambda Function
    Validates authorization token from request header
    """
    
    # Get the authorization token from the request header
    # Event structure:
    # {
    #     'authorizationToken': 'Bearer YOUR_TOKEN',
    #     'methodArn': 'arn:aws:execute-api:...'
    # }
    
    token = event.get('authorizationToken', '')
    method_arn = event.get('methodArn', '')
    
    # Define valid token (in production, verify against database/service)
    valid_token = 'mySecretToken123'
    
    # Parse token (usually "Bearer TOKEN_VALUE")
    if token.startswith('Bearer '):
        token_value = token.split(' ')[1]
    else:
        token_value = token
    
    # Validate token
    if token_value == valid_token:
        effect = 'Allow'
        principal_id = 'user123'  # User ID from token
    else:
        effect = 'Deny'
        principal_id = 'user'
    
    # Generate the authorization policy
    policy = generate_policy(principal_id, effect, method_arn)
    
    return policy


def generate_policy(principal_id, effect, resource):
    """
    Generate IAM policy for API Gateway
    
    Args:
        principal_id: The principal user identification associated with the policy
        effect: 'Allow' or 'Deny'
        resource: The API Gateway ARN
    
    Returns:
        IAM Policy dict
    """
    
    policy = {
        'principalId': principal_id,
        'policyDocument': {
            'Version': '2012-10-17',
            'Statement': [
                {
                    'Action': 'execute-api:Invoke',
                    'Effect': effect,
                    'Resource': resource
                }
            ]
        }
    }
    
    # Optionally add context that can be passed to backend Lambda
    policy['context'] = {
        'userId': principal_id,
        'token': 'authorized'
    }
    
    return policy
```

**Explanation**:
- `authorizationToken`: Token sent by client in header
- `methodArn`: ARN of the API method being called
- **Valid Token Check**: In production, verify against database
- **Policy Generation**: If token valid → Allow, If invalid → Deny
- **Principal ID**: User identification from token

##### 1.3 Deploy Lambda

1. Click **Deploy**

---

#### Step 2: Test Authorizer Lambda

##### 2.1 Create Test Event

1. Click **Test** tab
2. Create test event:

```json
{
    "authorizationToken": "Bearer mySecretToken123",
    "methodArn": "arn:aws:execute-api:us-east-1:123456789012:api-id/stage/GET/path"
}
```

3. Click **Test**

**Expected Result**: Returns policy with `Effect: Allow`

##### 2.2 Test Invalid Token

```json
{
    "authorizationToken": "Bearer wrongToken",
    "methodArn": "arn:aws:execute-api:us-east-1:123456789012:api-id/stage/GET/path"
}
```

Click **Test** → Should return `Effect: Deny`

---

#### Step 3: Create Authorizer in API Gateway

##### 3.1 Navigate to Authorizers

1. Go to API Gateway
2. Select your API
3. In left sidebar, click **Authorizers**

##### 3.2 Create Authorizer

1. Click **Create authorizer**
2. Name: `MyLambdaAuthorizer`
3. Authorizer type: **Lambda**
4. Lambda function: Select `api-authorizer`
5. **Authorization Token Source**: 
   - Field name: `Authorization`
   - (This tells API Gateway where to find the token in the request)
6. Click **Create authorizer**

---

#### Step 4: Test Authorizer

##### 4.1 Test Valid Token

Before attaching to API, test the authorizer:

1. Click on your authorizer name
2. Click **Test authorizer**
3. Token: `Bearer mySecretToken123`
4. Click **Test**

**Expected**: Status 200, Shows `Allow` effect

##### 4.2 Test Invalid Token

Token: `Bearer wrongToken`
Click **Test**

**Expected**: Status 200, Shows `Deny` effect

---

#### Step 5: Attach Authorizer to API Method

##### 5.1 Configure Method

1. Go to **Resources**
2. Select your method (e.g., **POST**)
3. Click **Method Request**
4. Click **Edit**
5. Under **Authorization**, click dropdown
6. Select your authorizer: `MyLambdaAuthorizer`
7. Click **Save**

##### 5.2 Deploy API

1. Click **Deploy API**
2. Select stage: `dev` (or create new)
3. Click **Deploy**

**Important**: Must redeploy for authorizer to take effect

---

#### Step 6: Test Protected API

##### 6.1 Test Without Token

In Postman:
```
URL: {invoke-url}/path
Method: POST
NO HEADERS
Click Send
```

**Expected**: `401 Unauthorized`

##### 6.2 Test With Invalid Token

```
URL: {invoke-url}/path
Headers:
    Authorization: Bearer wrongToken
Click Send
```

**Expected**: `403 Forbidden` - "User is not authorized"

##### 6.3 Test With Valid Token

```
URL: {invoke-url}/path
Headers:
    Authorization: Bearer mySecretToken123
Click Send
```

**Expected**: `200 OK` - Your API response

---

### Real-World Example: Token Validation with Database

```python
import json
import boto3

dynamodb = boto3.resource('dynamodb')
tokens_table = dynamodb.Table('api_tokens')

def lambda_handler(event, context):
    """
    Production-grade authorizer
    Validates token against DynamoDB
    """
    
    token = event.get('authorizationToken', '')
    method_arn = event.get('methodArn', '')
    
    # Extract token value
    if token.startswith('Bearer '):
        token_value = token.split(' ')[1]
    else:
        token_value = token
    
    try:
        # Look up token in DynamoDB
        response = tokens_table.get_item(Key={'token': token_value})
        
        if 'Item' in response:
            token_data = response['Item']
            
            # Check if token is still valid (not expired)
            if token_data.get('active') == True:
                effect = 'Allow'
                principal_id = token_data.get('user_id', 'user')
            else:
                effect = 'Deny'
                principal_id = 'user'
        else:
            effect = 'Deny'
            principal_id = 'user'
        
        policy = generate_policy(principal_id, effect, method_arn)
        return policy
        
    except Exception as e:
        print(f"Error: {str(e)}")
        # Fail closed - deny access on error
        return generate_policy('user', 'Deny', method_arn)


def generate_policy(principal_id, effect, resource):
    policy = {
        'principalId': principal_id,
        'policyDocument': {
            'Version': '2012-10-17',
            'Statement': [
                {
                    'Action': 'execute-api:Invoke',
                    'Effect': effect,
                    'Resource': resource
                }
            ]
        }
    }
    
    return policy
```

---

### API Key Alternative (For Simpler Cases)

If you just need basic key validation:

1. Go to API Gateway → Your API
2. Click **API Keys**
3. Create API Key
4. On method, set **API Key Required** to true
5. Deploy

Clients must send: `x-api-key: {key-value}`

**Pros**: Simpler setup
**Cons**: Less flexible than custom authorizer

---

## Complete Example: Building a Production-Ready API

### Scenario

Build a secure user registration API that:
1. Validates request format
2. Requires authorization token
3. Logs all requests
4. Only allows specific IPs
5. Transforms data before passing to Lambda

---

### Complete Architecture

```
┌─────────────┐
│   Client    │
└──────┬──────┘
       │ POST /users (with auth token)
       │ JSON: {name, email, mobile}
       ↓
┌─────────────────────────────────┐
│  API Gateway (REST API)         │
├─────────────────────────────────┤
│ 1. Resource Policy (IP check)   │
│ 2. Authorizer (Token validation)│
│ 3. Validator (Format check)     │
│ 4. Mapping Template (Transform) │
│ 5. CloudWatch Logs (Record)     │
└──────────┬──────────────────────┘
           │ Clean data
           ↓
┌─────────────────────────────────┐
│     Lambda Function             │
│  (Non-Proxy Integration)        │
│  - Process registration         │
│  - Store in database            │
└──────────┬──────────────────────┘
           │ Processed response
           ↓
┌─────────────┐
│  CloudWatch │
│  Logs       │
└─────────────┘
```

---

### Implementation

#### Step 1: Create IAM Role for CloudWatch Logging

1. IAM → Roles → Create role
2. Service: API Gateway
3. Policy: AmazonAPIGatewayPushToCloudWatchLogs
4. Name: `APIGatewayCloudWatchRole`
5. Copy ARN

#### Step 2: Create Authorizer Lambda

```python
def lambda_handler(event, context):
    token = event.get('authorizationToken', '')
    method_arn = event.get('methodArn', '')
    
    if token.startswith('Bearer '):
        token_value = token.split(' ')[1]
    else:
        token_value = token
    
    valid_tokens = ['user-token-12345', 'admin-token-67890']
    
    if token_value in valid_tokens:
        effect = 'Allow'
        user_id = 'user-123'
    else:
        effect = 'Deny'
        user_id = 'unknown'
    
    policy = {
        'principalId': user_id,
        'policyDocument': {
            'Version': '2012-10-17',
            'Statement': [{
                'Action': 'execute-api:Invoke',
                'Effect': effect,
                'Resource': method_arn
            }]
        }
    }
    
    return policy
```

#### Step 3: Create Registration Lambda

```python
def lambda_handler(event, context):
    try:
        # Get data from request mapping
        data = event
        
        # Simulate storing in database
        user_id = 'user-' + str(hash(data.get('email', '')))
        
        return {
            'statusCode': 201,
            'user_id': user_id,
            'message': f'User {data.get("name")} registered successfully',
            'email': data.get('email')
        }
    except Exception as e:
        return {
            'statusCode': 500,
            'error': str(e)
        }
```

#### Step 4: Create API Gateway

1. REST API
2. Name: `production-user-api`
3. Create resource: `users`
4. Create method: POST

#### Step 5: Attach Authorizer

1. Method Request → Authorization → Select your authorizer
2. Save

#### Step 6: Create Validation Model

```json
{
    "type": "object",
    "properties": {
        "name": {
            "type": "string",
            "minLength": 2,
            "maxLength": 100
        },
        "email": {
            "type": "string",
            "format": "email"
        },
        "mobile": {
            "type": "string",
            "pattern": "^[0-9]{10}$"
        }
    },
    "required": ["name", "email", "mobile"],
    "additionalProperties": false
}
```

#### Step 7: Attach Validator

Method Request → Request Body → Add model

#### Step 8: Configure Request Mapping

Integration Request → Mapping Template → Content-Type: application/json

```velocity
{
    "name": "$input.json('$.name')",
    "email": "$input.json('$.email')",
    "mobile": "$input.json('$.mobile')",
    "timestamp": "$context.requestTime",
    "userId": "$context.authorizer.principalId"
}
```

#### Step 9: Configure Resource Policy

Resource Policy:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "execute-api:Invoke",
            "Resource": "execute-api:/*"
        },
        {
            "Effect": "Deny",
            "Principal": "*",
            "Action": "execute-api:Invoke",
            "Resource": "execute-api:/*",
            "Condition": {
                "IpAddress": {
                    "aws:SourceIp": "192.168.1.100"
                }
            }
        }
    ]
}
```

#### Step 10: Enable CloudWatch Logging

1. API Gateway Settings
2. CloudWatch Logs Role ARN: Paste your saved ARN
3. On API Stage → Enable CloudWatch Logs
4. Log Level: INFO

#### Step 11: Deploy

Deploy API to `prod` stage

#### Step 12: Test Complete Flow

**Test with Postman**:

```
URL: {invoke-url}/users
Method: POST
Headers:
    Authorization: Bearer user-token-12345
    Content-Type: application/json
Body:
{
    "name": "John Doe",
    "email": "john@example.com",
    "mobile": "9876543210"
}

Click Send
```

**Expected**:
- Status: 201 Created (or 200 OK)
- Response: User registered successfully
- CloudWatch Logs: Full request/response recorded

---

## Best Practices Summary

### Security

✅ Always use HTTPS (API Gateway handles this)
✅ Implement Authorizers for protected endpoints
✅ Use Resource Policies to restrict IP access
✅ Enable request validation
✅ Rotate API Keys and tokens regularly
✅ Never log sensitive data (passwords, tokens)

### Performance

✅ Use HTTP API for simple, modern applications
✅ Use REST API only when needed
✅ Enable caching where appropriate
✅ Optimize Lambda execution time
✅ Use CloudFront for static content

### Monitoring

✅ Always enable CloudWatch Logs
✅ Set up alarms for error rates
✅ Monitor API Gateway metrics
✅ Regular log analysis
✅ Track cost metrics

### Cost Optimization

✅ Use validators to block invalid requests early
✅ Optimize Lambda execution
✅ Clean up unused stages
✅ Monitor CloudWatch log storage
✅ Use HTTP API when possible (cheaper)

---

## Troubleshooting Guide

### Common Issues

#### Issue 1: 401 Unauthorized

**Causes**:
- Authorization token missing
- Token format wrong (should be "Bearer TOKEN")
- Authorizer lambda returning Deny

**Solution**:
1. Check if token is in header
2. Verify token format
3. Test authorizer lambda separately
4. Check CloudWatch logs

#### Issue 2: 400 Bad Request

**Causes**:
- Request body doesn't match validation model
- Missing required fields
- Invalid data format

**Solution**:
1. Verify request body against model schema
2. Check data types (email should be email format)
3. Check required fields are present
4. Review CloudWatch logs for detailed error

#### Issue 3: 403 Forbidden (Resource Policy)

**Causes**:
- Your IP is in the deny list
- Not from allowed IP range
- AWS account not in allow list

**Solution**:
1. Verify your current IP (`whatismyip.com`)
2. Check resource policy rules
3. Ensure you're connecting from correct network

#### Issue 4: 500 Internal Server Error

**Causes**:
- Lambda execution error
- Timeout
- Insufficient permissions
- Invalid mapping template

**Solution**:
1. Check CloudWatch Logs for Lambda error
2. Increase Lambda timeout
3. Check Lambda IAM role permissions
4. Validate mapping template syntax

#### Issue 5: Lambda Not Responding

**Causes**:
- Lambda not integrated properly
- Integration request/response configuration wrong
- Lambda returning wrong format

**Solution**:
1. Test Lambda separately
2. Check Lambda response format
3. Verify integration type (proxy vs non-proxy)
4. Check mapping templates

---

## Final Checklist for Production Deployment

Before deploying to production:

- [ ] All APIs have authorizers
- [ ] CloudWatch logging enabled
- [ ] Resource policies configured
- [ ] Request validators enabled
- [ ] Error handling implemented
- [ ] All endpoints tested with Postman
- [ ] Load testing completed
- [ ] SSL/TLS enabled (default)
- [ ] API Keys rotated
- [ ] Alarms configured
- [ ] Documentation complete
- [ ] Team trained on API usage
- [ ] Cost monitoring setup
- [ ] Disaster recovery plan ready
- [ ] API versioning strategy defined

---

## Additional Resources

### Tools Needed

1. **Postman** - API testing: `https://postman.com`
2. **AWS Console** - Management: `https://console.aws.amazon.com`
3. **VS Code** - Code editing: `https://code.visualstudio.com`
4. **Git** - Version control: `https://git-scm.com`

### AWS Documentation

- API Gateway Documentation: https://docs.aws.amazon.com/apigateway/
- Lambda Documentation: https://docs.aws.amazon.com/lambda/
- CloudWatch Logs: https://docs.aws.amazon.com/AmazonCloudWatch/
- IAM Policies: https://docs.aws.amazon.com/IAM/

### Next Steps

1. Create your first simple API
2. Add authorization to an existing API
3. Implement request validation
4. Enable CloudWatch logging
5. Set up resource policies
6. Build a complete production-ready API

---

## Conclusion

You now have a complete understanding of AWS API Gateway from basics to production deployment. Start with simple APIs and gradually add complexity as you become more comfortable with each feature.

Remember: Security and monitoring are not optional in production - they are essential!

Good luck with your API development journey! 🚀
