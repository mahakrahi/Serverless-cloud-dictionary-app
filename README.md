<h1>Serverless Cloud Dictionary Application</h1>

<h2>Project Overview:</h2>

In this project, we'll be developing a serverless cloud dictionary application where users can:

-Search for terms related to cloud technologies.

-View the definitions of cloud terms.

-Utilize a serverless architecture using AWS services.

-The application will use Lambda for backend processing, API Gateway for managing the API endpoints, and DynamoDB for storing the dictionary terms and their definitions.

The frontend, a React application, will be hosted on AWS Amplify, and API requests will be made to interact with the database.


<h2>Services Used: </h2>

-AWS Amplify: Host the frontend React application.

-AWS Lambda: Handle API requests for retrieving terms and adding new ones.

-AWS API Gateway: Manage API endpoints to allow communication between frontend and Lambda functions.

-AWS DynamoDB: Store dictionary terms and definitions.

-IAM Roles & Policies: Secure access to AWS resources like Lambda, DynamoDB, and API Gateway.

<h2>Project Architecture: </h2>

<img width="1381" height="541" alt="531621442-56a6c9b2-dece-49dc-ad62-29848fa59418" src="https://github.com/user-attachments/assets/9937ea7c-69ab-4b3b-a4fd-40ce9b1c12d9" />


<h2>Steps to be performed:</h2>

In the next few lessons, we'll be going through the following steps.

-Setup frontend and host it on AWS Amplify

-Configure DynamoDB to store Cloud Definitions

-Create Lambda function for fetching terms

-Setup API Gateway for API management

<h2>Lambda Function Code: </h2>


```python
import json
import boto3

# Create a DynamoDB client
dynamodb = boto3.client('dynamodb')

# Your DynamoDB table name
table_name = 'CloudDefinitions'

def lambda_handler(event, context):
    try:
        # Debug: Log the entire event
        print("Event received:", json.dumps(event))
        
        # Try to get term from multiple sources
        term = ''
        
        # Method 1: Query parameters (for GET requests)
        query_params = event.get('queryStringParameters') or {}
        if query_params and query_params.get('term'):
            term = query_params.get('term')
        
        # Method 2: Path parameters
        path_params = event.get('pathParameters') or {}
        if not term and path_params and path_params.get('term'):
            term = path_params.get('term')
        
        # Method 3: Body (for POST requests)
        if not term and event.get('body'):
            try:
                body = json.loads(event['body'])
                term = body.get('term', '')
            except:
                pass
        
        # Method 4: Direct event (for test events)
        if not term:
            term = event.get('term', '')
        
        print("Term extracted:", term)
        
        # If no term provided, return error with debug info
        if not term:
            return {
                'statusCode': 400,
                'headers': {
                    'Content-Type': 'application/json',
                    'Access-Control-Allow-Origin': '*',
                    'Access-Control-Allow-Methods': 'OPTIONS,GET,POST',
                    'Access-Control-Allow-Headers': 'Content-Type',
                },
                'body': json.dumps({
                    'message': 'Term parameter is required',
                    'debug': {
                        'queryStringParameters': event.get('queryStringParameters'),
                        'pathParameters': event.get('pathParameters'),
                        'body': event.get('body'),
                        'httpMethod': event.get('httpMethod'),
                        'event_keys': list(event.keys())
                    }
                })
            }
        
        # Query the DynamoDB table for the term
        response = dynamodb.get_item(
            TableName=table_name,
            Key={'term': {'S': term}}
        )
        
        # Check if the term exists in the table
        if 'Item' in response:
            definition = response['Item']['definition']['S']
            return {
                'statusCode': 200,
                'headers': {
                    'Content-Type': 'application/json',
                    'Access-Control-Allow-Origin': '*',
                    'Access-Control-Allow-Methods': 'OPTIONS,GET,POST',
                    'Access-Control-Allow-Headers': 'Content-Type',
                },
                'body': json.dumps({
                    'term': term,
                    'definition': definition
                })
            }
        else:
            return {
                'statusCode': 404,
                'headers': {
                    'Content-Type': 'application/json',
                    'Access-Control-Allow-Origin': '*',
                    'Access-Control-Allow-Methods': 'OPTIONS,GET,POST',
                    'Access-Control-Allow-Headers': 'Content-Type',
                },
                'body': json.dumps({'message': f'Term "{term}" not found'})
            }
            
    except Exception as e:
        return {
            'statusCode': 500,
            'headers': {
                'Content-Type': 'application/json',
                'Access-Control-Allow-Origin': '*',
                'Access-Control-Allow-Methods': 'OPTIONS,GET,POST',
                'Access-Control-Allow-Headers': 'Content-Type',
            },
            'body': json.dumps({
                'error': 'Internal server error',
                'message': str(e)
            })
        }
```

<h2>TEST Event:</h2>

```
{
  "queryStringParameters": {
    "term": "AWS KMS"
  }
}
```

<h2>DynamoDB AWS CLI Commands:</h2>

```
aws dynamodb batch-write-item --request-items file://records/records-1.json
aws dynamodb batch-write-item --request-items file://records/records-2.json
aws dynamodb batch-write-item --request-items file://records/records-3.json
aws dynamodb batch-write-item --request-items file://records/records-4.json
```
