# My Notes: Full-Stack Developer

Followed the [totorial](https://aws.amazon.com/getting-started/learning-path-full-stack-developer/?e=gs2020&p=gsrc) to deploy a web app with serverless backend and DynamoDB.

- [My Notes: Full-Stack Developer](#my-notes-full-stack-developer)
  - [Create a Web App](#create-a-web-app)
    - [Create a Web App with Amplify Console](#create-a-web-app-with-amplify-console)
    - [Test The Web App](#test-the-web-app)
  - [Build a Serverless Function](#build-a-serverless-function)
    - [Create and Configure The Lambda Function](#create-and-configure-the-lambda-function)
    - [Test The Lambda Function](#test-the-lambda-function)
  - [Link a Serverless Function to a Web App](#link-a-serverless-function-to-a-web-app)
    - [Create a New REST API](#create-a-new-rest-api)
    - [Create a New Resource and Method](#create-a-new-resource-and-method)
    - [Deploy API](#deploy-api)
    - [Validate API](#validate-api)
  - [Create a Data Table](#create-a-data-table)
    - [Create a DynamoDB Table](#create-a-dynamodb-table)
    - [Create and Add IAM Policy to Lambda Function](#create-and-add-iam-policy-to-lambda-function)
    - [Modify Lambda Function to Write to DynamoDB table](#modify-lambda-function-to-write-to-dynamodb-table)
    - [Test the Changes](#test-the-changes)
  - [Add Interactivity to Your Web App](#add-interactivity-to-your-web-app)
    - [Update Web App with Amplify Console](#update-web-app-with-amplify-console)
    - [Test Update Web App](#test-update-web-app)

## Create a Web App

### Create a Web App with Amplify Console

- Select **Host web app** in Amplify Console.
- **Deploy with Git provider**
- App name
- Environment name: dev
- Select **Drag and drop** method
- Select the ZIP file created.
- Save and deploy

### Test The Web App

Click on the link under Domain.

## Build a Serverless Function

- Compute Service
- Serverless function
- Lambda Trigger

### Create and Configure The Lambda Function

- Log into the **AWS Lamdba Console**
- Click "**Create Function**" button
- Give a Function name
- Select **Python 3.8** from the runtime drop-down
- Click on the "**Create Function**" button
- Replace the code under "Function Code"

  ```python
  import json
  def lambda_handler(event, context):
      name = event['firstName'] +' '+ event['lastName']
      return {
      'statusCode': 200,
      'body': json.dumps('Hello from Lambda, ' + name)
      }
  ```

- Click the orange "**Deploy**" button
- Let's test. Click on "Select a test event" at the top of screen
- From that drop-down menu click on "**Configure test events**"
- Event name: HelloWorldTestEvent
- paste the JSON object

  ```json
  {
    "firstName": "Ada",
    "lastName": "Lovelace"
  }
  ```

- Click the orange "**Create**"

### Test The Lambda Function

Click the grey "**Test**" button at the top of the page.

## Link a Serverless Function to a Web App

### Create a New REST API

- Log into the **API Gateway Console**
- Click the orange "**Create API**" button
- Find the **REST API** box and click the orange "**Build**" button in it
- Under "Choose the protocol" select **REST**
- Under "Create new API" select New API
- Give a API name
- Select "**Edge optimized**" in the "Endpoint Type" drop-down
- Click the blue "**Create** API" button

### Create a New Resource and Method

- In the left nav, click on "**Resources**" under the API
- With the "/" resource selected, click "**Create Method**" from the Action drop-down menu
- Select **POST** from the new drop-down that appers, then click on the checkmark
- Select Lambda Function for the integration type
- Type in HelloWorldFunction into the "Function" field
- Click the blue "**Save**" button
- With the newly created POST method selected, selecte "**Enable CORS**" from the Action drop-down menu
- Leave the POST checkbox selected and click the blue "**Enable CORS and replace existing CORS headers**" button
- Click the blue "**Yes, replace existing values**" button

### Deploy API

- In the "**Actions**" drop-down list select "**Deploy API**"
- Select "**[New Stage]**" in the "**Deployment stage**" drop-down list
- Enter _dev_ for the "**Stage Name**".
- Choose "**Deploy**"
- Copy the URL next to "**Invoke URL**"

### Validate API

- On the left nav, click on "**Resources**"
- The methods for our API will now be listed on the right. Click on "**POST**"
- Click on the **small blue light bolt**
- Paste the following into the "Request Body" field

  ```js
  {
      "firstName":"Grace",
      "lastName":"Hopper"
  }
  ```

- Click the blue "**Test**" button
- On the right side, we should see a response with **Code 200**

## Create a Data Table

### Create a DynamoDB Table

- Log into the DynamoDB Console
- Click the blue "**Create table**" button
- Next to "Table name" type in HelloWorldDatabase
- In the "Primary Key" field type in ID
- Click the blue "**Create**" button
- Copy the table's "**Amazon Resource Name (ARN)**" from the right hand panel

### Create and Add IAM Policy to Lambda Function

- Open the **AWS Lambda Console**
- Click on the function we created
- We'll be adding permissions to our function so it can use the DynamoDB service
- Click on the "**Permissions**" tab
- In the "Execution role" box **click on the role**. A new browser tab will open
- Click on "Add inline policy" on the right of the "Permissions policies" box
- Click on the "JSON" tab
- Paste the following policy in the text area, taking care to replace ARN in the "Resource" field on line 15

  ```json
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Sid": "VisualEditor0",
        "Effect": "Allow",
        "Action": [
          "dynamodb:PutItem",
          "dynamodb:DeleteItem",
          "dynamodb:GetItem",
          "dynamodb:Scan",
          "dynamodb:Query",
          "dynamodb:UpdateItem"
        ],
        "Resource": "YOUR-TABLE-ARN"
      }
    ]
  }
  ```

- Click the blue "**Review Policy**" button
- Next to name type in _HelloWorldDynamoPolicy_
- Click the blue "**Create Policy**" button

### Modify Lambda Function to Write to DynamoDB table

- Click the "**Configuration**" tab in Lambda function
- Replace the code

  ```py
  import json
  import boto3
  from time import gmtime, strftime

  dynamodb = boto3.resource('dynamodb')
  table = dynamodb.Table('HelloWorldDatabase')
  now = strftime("%a, %d %b %Y %H:%M:%S +0000", gmtime())

  def lambda_handler(event, context):
      name = event['firstName'] +' '+ event['lastName']
      response = table.put_item(
          Item={
              'ID': name,
              'LatestGreetingTime':now
              })
      return {
          'statusCode': 200,
          'body': json.dumps('Hello from Lambda, ' + name)
      }
  ```

- Click the orange "**Save**" button at the top of the page

### Test the Changes

- Click the white "**Test**" button
- We should see an "Execution result: succeeded" message with a green background
- Open the **DynamoDB Console**
- Click on "**Tables**" on the left hand nav bar
- Click on HelloWorldDatabase
- Click on the "Items" tab on the right
- Items matching our test event should appear here

## Add Interactivity to Your Web App

Call an API Gateway API from an HTML page

### Update Web App with Amplify Console

Modify the file. Make sure add the **API Invoke URL**.

```js
var callAPI = (firstName, lastName) => {
  var myHeaders = new Headers();
  myHeaders.append("Content-Type", "application/json");
  var raw = JSON.stringify({ firstName: firstName, lastName: lastName });
  var requestOptions = {
    method: "POST",
    headers: myHeaders,
    body: raw,
    redirect: "follow",
  };
  // NOTE: replace URL here
  fetch("YOUR-API-INVOKE-URL", requestOptions)
    .then((response) => response.text())
    .then((result) => alert(JSON.parse(result).body))
    .catch((error) => console.log("error", error));
};
```

Upload the ZIP file to Amplify Console.

### Test Update Web App

We shold see a message that starts with "Hello from Lambda" followed by the text you filled in when click the "Call API" button.
