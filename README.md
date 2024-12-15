
# Power of Math Project with AWS Amplify, Lambda, API Gateway, and DynamoDB
This project demonstrates how to build a web app that performs exponentiation calculations using AWS services including Amplify, Lambda, API Gateway, and DynamoDB.

![cloud architecture](images/cloud-architecture.png)

Steps to Deploy
1. Create the Initial HTML Page
Create a local index.html file with the following content:
html
```
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>To the Power of Math!</title>
</head>

<body>
    To the Power of Math!
</body>
</html>
```
Zip the index.html file.

 ![Step 1](images/1.png)
 
2. Set Up AWS Amplify
- Navigate to the AWS Amplify Console.
 ![Step 2](images/2.png)

- Select Create new app and choose Deploy without Git.
  
 ![Step 3](images/3.png)
  
- Provide the app name (e.g., PowerOfMath) and environment (e.g., dev).
  
 ![Step 4](images/4.png)
  
- Drag and drop the index.html zip file you created.
  
 ![Step 5](images/5.png)
  
- Click Save and Deploy.
  
- After deployment, click on the generated link to view the result.

 ![Step 6](images/6.png)
  
4. Create the Lambda Function for Math Operation
- Navigate to the AWS Lambda Console.
  
  ![Step 7](images/7.png)
   
- Click Create function, give it a name (e.g., ExponentiationFunction), and choose Python as the runtime.
  
  ![Step 8](images/8.png)
  
In the Lambda function, replace the lambda_function.py content with the following Python code:
Python
```
import json
import math

def lambda_handler(event, context):
    mathResult = math.pow(int(event['base']), int(event['exponent']))
    return {
        'statusCode': 200,
        'body': json.dumps('Your result is ' + str(mathResult))
    }

```
   ![Step 9](images/9.png)
   
- Save (Ctrl+S) and click on the Deploy button.
  
   ![Step 10](images/10.png)

- Then test the function with an event:
json
```

{
    "base": 2,
    "exponent": 3
}

```
![Step 11](images/11.png)

![Step 12](images/12.png)
4. Set Up API Gateway
- Navigate to the API Gateway Console.

![Step 13](images/13.png)

- Create a new REST API. Click on Build.

![Step 14](images/14.png)
  
-  Give it a name (e.g., PowerOfMathAPI).
- ![Step 15](images/15.png)
- Create a new resource (e.g., /pow-of-math), and enable CORS.
 ![Step 16](images/16.png)
- Create a new POST method for this resource,
 ![Step 17](images/17.png)

-   Integrating it with the Lambda function and click on Create method.
  
   ![Step 18](images/18.png)

- For enabling CORS on this method. Click on /pow-of-math and select Enable CORS.

   ![Step 19](images/19.png)

- Select Access-Control-Allow-Methods as POST and save
    ![Step 20](images/20.png)
- Deploy the API to a new stage (e.g., dev).
    ![Step 22](images/22.png)
- Copy the Invoke URL generated after deployment.
 
   ![Step 23](images/23.png)

- Test the integration by going to the Test tab.

  ![Step 24](images/24.png)
  ![Step 25](images/25.png)
  
5. Update Lambda Permissions for DynamoDB
- Navigate to the DynamoDB Console and create a new table with ID as the partition key.
   ![Step 26](images/26.png)
   ![Step 27](images/27.png)
- Copy the ARN of the table.

  ![Step 28](images/28.png)
  
- Go to the Execution Role of your Lambda function, and create an inline policy to allow write permissions to the DynamoDB table.

  ![Step 29](images/29.png)
  ![Step 30](images/30.png)
  ![Step 31](images/31.png)
  
Here’s the policy you can use:

json
Copy code
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

![Step 32](images/32.png)
![Step 33](images/33.png)
![Step 34](images/34.png)
![Step 35](images/35.png)

6. Update Lambda Function to Write to DynamoDB
Modify your Lambda function to save the result to DynamoDB:
python
```
import json
import math
import boto3
from time import gmtime, strftime

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('PowerOfMathDatabase')
now = strftime("%a, %d %b %Y %H:%M:%S +0000", gmtime())

def lambda_handler(event, context):
    mathResult = math.pow(int(event['base']), int(event['exponent']))
    response = table.put_item(
        Item={
            'ID': str(mathResult),
            'LatestGreetingTime': now
        }
    )
    return {
        'statusCode': 200,
        'body': json.dumps('Your result is ' + str(mathResult))
    }
```
![Step 36](images/36.png)

- Ensure the DynamoDB table name in the Lambda function matches your table.
![Step 37](images/37.png)

- Click Deploy and test the function again.

![Step 38](images/38.png)

7. Update HTML to Call API Gateway
Update the index.html page to integrate with API Gateway. Replace the fetch URL with your Invoke URL from API Gateway:
html
Copy code
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Exponentiation Calculator</title>
    <style>
        /* Styling as described earlier */
    </style>
    <script>
        function calculateExponentiation(base, exponent) {
            const headers = new Headers();
            headers.append("Content-Type", "application/json");

            const bodyData = JSON.stringify({ base, exponent });
            const requestOptions = {
                method: 'POST',
                headers: headers,
                body: bodyData,
                redirect: 'follow'
            };

            fetch("https://YOUR-INVOKE-URL-HERE", requestOptions)
                .then(response => response.json())
                .then(data => alert(`Result: ${data.body}`))
                .catch(error => console.error('Error:', error));
        }

        function handleCalculation() {
            const baseValue = document.getElementById('base-input').value;
            const exponentValue = document.getElementById('exponent-input').value;
            calculateExponentiation(baseValue, exponentValue);
        }
    </script>
</head>
<body>
    <div class="container">
        <h1>Exponentiation Calculator</h1>
        <form>
            <label for="base-input">Enter Base:</label>
            <input type="number" id="base-input" placeholder="Base" required>

            <label for="exponent-input">Enter Exponent:</label>
            <input type="number" id="exponent-input" placeholder="Exponent" required>

            <button type="button" onclick="handleCalculation()">Calculate</button>
        </form>
    </div>
</body>
</html>
8. Re-deploy HTML Page Using AWS Amplify
Zip the updated index.html file.
Go back to AWS Amplify, upload the new zip file, and redeploy the app to the dev environment.
9. Test the Application
Click on the generated Amplify domain URL to see your exponentiation calculator in action.
Enter a base and exponent to get the result, which will also be saved in DynamoDB.
Technologies Used:
AWS Amplify: For hosting the web app.
AWS Lambda: To perform the exponentiation math operation.
AWS API Gateway: To expose the Lambda function as an API.
AWS DynamoDB: To store the calculation results.
