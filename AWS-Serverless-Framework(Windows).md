**Assumptions**: This tutorial assumes that the local development environment is Windows 7+ and is a traditional "locked down" corporate environment.
# Serverless Framework Quickstart
## Local Development Environment Pre-requisites
### 1.  Obtain administrator credentials / elevated privileges
Obtain elevated priveleges / firecall credentials for your machine.
### 2.  Install / Upgrade NodeJS  and npm
Note: This step could be skipped if NodeJS / npm are already present and meet minimum version requirements (see next section)
  1. Download and run the most current LTS NodeJS Windows Installation msi @ https://nodejs.org/en/download/
  1. Accept UELA, and continue through the installation with the default settings
  1. Enter administrator credentials when prompted during final install step
  1. Wait until completes with confirmation of successful installation.
### 3.  Validate Node.JS / npm installation and version requirements
  1. Launch Powershell (Ctrl + Win key -> type 'Powershell' -> Enter)  
  1. execute `node -v`
  1. Ensure version of node is **at least** v6.5.0 or later.  If not, re-install latest LTS distribution.
### 4.  Install the Serverless CLI
  1. Set npm proxy if on corporate network machine.  Execute the following commands (be sure to substitute $PROXY_HOST as necessary):
      * `npm config set https-proxy http://$PROXY_HOST:8080/`
      * `npm config set proxy http://$PROXY_HOST:8080/`
  1. Install serverless cli module.  This may take quite some time depending on the level of corporate spyware present on your machine:
      * `npm install -g serverless`
### 5.  AWS Account setup
  If you do not already have an AWS Account, you can sign up for a [free trial](https://aws.amazon.com/s/dm/optimization/server-side-test/free-tier/free_np/).  AWS permits a generous 1 Million Lambda function invocations / month for free **indefinitely**.  This offer is available to both existing and new customers.  

### 6.  Set-up Cloud Provider Credentials.
There are a number of ways this can be achieved, but in general, AWS best practices recommend that you create a new IAM Users specifically for granting access to AWS resources, and follow security best practices of granting the least privledges necessary for the task.  If you are unfamiliar with these concepts, the Serverless framework documentation has a [step-by-step guide](https://serverless.com/framework/docs/providers/aws/guide/credentials/) and a [video on setting up credentials](https://www.youtube.com/watch?v=HSd9uYj2LJA)

For quick reference, set AWS credentials temporarily that live only for the length of the Powershell session:
```powershell
>  $env:AWS_ACCESS_KEY_ID = 'abcd'
>  $env:AWS_SECRET_ACCESS_KEY = 'wyxz'
```


#### Security Concerns and Analysis
In order to follow industry security best practices, AWS Security best practices, and the AWS Well Architected Framework, one must provision a new IAM user specifically for the task of granting access to the Serverless Framework.   This user should only be granted the **bare minimum** level of access control, permissions, and policies within the AWS account.  This is generally regarded as the [Principal of least privilege](https://en.wikipedia.org/wiki/Principle_of_least_privilege) and is a fundamental concept in designing secure systems.

Unfortunately, the official Serverless Framework documentation does not publish the minimum AWS IAM service permissions and policies required by the framework.  In fact, the official documentation and guides gloss over this detail, and instruct you to grant a **full AdminstratorAccess** access policy.  This is a huge red flag and results in a critical security risk.  

I spent the better part of a week, starting off with a IAM user with zero access policies, iterating thru serverless framework deploy -> debug access control failure -> add IAM permission policy loops.  Ultimately I was unable to successfully deploy even the most simple Serverless Framework app without comprimising security standards.  This finding, in and of iteself, raises concerns over the viability of the Serverless Framework and usage within secure, enterprise applications.

## Creating and Deploying a simple API
First create a new directory and init npm (or hand craft a `package.json`):

```powershell
> mkdir express-app ; cd express-app
> npm init -f
```

Install a few dependencies.  The `express` framework and [`serverless-http`](https://github.com/dougmoscrop/serverless-http).  This combination of modules will provide a very similar experience to hosted Node.js + express apps:

```powershell
> npm install --save express serverless-http
```
`serverless-http` is http middleware, and handles the communication layer between our application code and the AWS API Gateway.  

Next, create an `index.js` file and expose a basic Express Route :

```javascript
// index.js

const serverless = require('serverless-http');
const express = require('express')
const app = express()

app.get('/', function (req, res) {
  res.send('Hello World')
})

module.exports.handler = serverless(app);
```

Express routing behaves similar to non serverless environments and builds on relative path exposed by AWS API Gateway.

Last but not least, create a `serverless.yml` file.  This serves as the configuration file to drive the Serverless CLI.  The configuration details within the file represent your high-level **Infrastructure as Code**.  More on this later, but simple, concise config parameters here drive the creation and of an AWS CloudFormation Stack file.  CloudFormation serve as your lower-level AWS **Infrastructure as Code** and can be rather esoteric.

```yml
# serverless.yml

service: express-app

provider:
  name: aws
  runtime: nodejs6.10
  stage: dev
  region: us-east-1

functions:
  app:
    handler: index.handler
    events:
      - http: ANY /
      - http: 'ANY {proxy+}'
```

At this point, the configuration defines a single function `app` and a Javascript function defined and exported in `index.js`.  This is the main entry point for Lambda triggers.  The function will be triggerd by HTTP Events (AWS API Gateway).  The broad url path matching here essentially delegates all routing to the express app.  However, complex confuration and API creation can be described directly within the `serverless.yml` config file.

Now, Deploy time:
```powershell
> serverless deploy
```
```diff
- ADD Command Output from confluence notes!
```

You can now invoke your Lambda endpoint by navigating to the URL (or using cURL, Postman, etc) displayed in your deploy output:
```diff
- ADD screen shots from confluence
```

Tada - You have gone serverless!

## Redeploying
You can iteratively develop, deploy, and test your application simply by making changes in your local environment and executing a deploy:
```powershell
>  serverless deploy
```

## Cleanup / Teardown
You can delete your serverless application and all resources provisioned by the Severless Framework by executing:
```powershell
>  serverless remove
```
## Extra Credit - DynamoDB and RESTful API 
Enhancing functionality of the 'hello-world' express serverless app by adding a "RESTful" API will be demonistrated here.  To provision a DynamoDB table, the `serverless.yml` configuration will require three additions:
 1. Provision the Dynamo in the `resources` section.  This section allows you to include raw CloudFormation YAML template syntax. 
 2. Add the requisite IAM permissions to the Lambda IAM Role.  This permits the Lambda function to access DynamoDB.
 3. Pass the DynamoDB table name as an environment variable to Lambda.  This tactic allows reusability and modularity.
 
The resulting `serverless.yml` becomes:
```yml
# serverless.yml

service: express-app

custom:
  tableName: 'users-table-${self:provider.stage}'

provider:
  name: aws
  runtime: nodejs6.10
  stage: dev
  region: us-east-1
  iamRoleStatements:
    - Effect: Allow
      Action:
        - dynamodb:Query
        - dynamodb:Scan
        - dynamodb:GetItem
        - dynamodb:PutItem
        - dynamodb:UpdateItem
        - dynamodb:DeleteItem
      Resource:
        - { "Fn::GetAtt": ["UsersDynamoDBTable", "Arn" ] }
  environment:
    USERS_TABLE: ${self:custom.tableName}

functions:
  app:
    handler: index.handler
    events:
      - http: ANY /
      - http: 'ANY {proxy+}'

resources:
  Resources:
    UsersDynamoDBTable:
      Type: 'AWS::DynamoDB::Table'
      Properties:
        AttributeDefinitions:
          -
            AttributeName: userId
            AttributeType: S
        KeySchema:
          -
            AttributeName: userId
            KeyType: HASH
        ProvisionedThroughput:
          ReadCapacityUnits: 1
          WriteCapacityUnits: 1
        TableName: ${self:custom.tableName}
``` 
 
Install a few helper libraries via npm.  `aws-sdk` client sdk will be used to access DynamoDB, and `body-parser` will facilitate HTTP request body processing:
```powershell
> npm install --save aws-sdk body-parser
```

Add a GET and POST route to `index.js`:
```javascript
// index.js

const serverless = require('serverless-http');
const bodyParser = require('body-parser');
const express = require('express')
const app = express()
const AWS = require('aws-sdk');


const USERS_TABLE = process.env.USERS_TABLE; // This is the variable we setup in the serverless.yml.  Lambda env vars are in process.env
const dynamoDb = new AWS.DynamoDB.DocumentClient();

app.use(bodyParser.json({ strict: false }));

app.get('/', function (req, res) {
  res.send('Hello World!')
})

// Get User endpoint
app.get('/users/:userId', function (req, res) {
  const params = {
    TableName: USERS_TABLE,
    Key: {
      userId: req.params.userId,
    },
  }

  dynamoDb.get(params, (error, result) => {
    if (error) {
      console.log(error);
      res.status(400).json({ error: 'Could not get user' });
    }
    if (result.Item) {
      const {userId, name} = result.Item;
      res.json({ userId, name });
    } else {
      res.status(404).json({ error: "User not found" });
    }
  });
})

// Create User endpoint
app.post('/users', function (req, res) {
  const { userId, name } = req.body;
  if (typeof userId !== 'string') {
    res.status(400).json({ error: '"userId" must be a string' });
  } else if (typeof name !== 'string') {
    res.status(400).json({ error: '"name" must be a string' });
  }

  const params = {
    TableName: USERS_TABLE,
    Item: {
      userId: userId,
      name: name,
    },
  };

  dynamoDb.put(params, (error) => {
    if (error) {
      console.log(error);
      res.status(400).json({ error: 'Could not create user' });
    }
    res.json({ userId, name });
  });
})

module.exports.handler = serverless(app);
```
The resulting REST API will have the following additional endpoints:
* `GET /users/:userId` to query and retireve a User
* `POST /users` to create a new User

Redeploy the app:
```powershell
>  serverless deploy
```
Capture the resulting endpoint API URL, and store it in an env variable for future use:
```powershell
>  $env:SERVERLESS_URL = 'https://t1zobfla26.execute-api.us-east-1.amazonaws.com/dev'
```
In Powershell, the 'curl' command is by default aliased to the 'Invoke-WebRequest' utility.  For consistency, we will invoke the API with Powershell-native Invoke-WebRequest.  To Create a User, send a POST request:
```diff
- INSERT CMD from Confluence!
```

```powershell
```
Retrieve a user, send a GET request to the API:
```diff
- INSERT CMD from Confluence!
```

```powershell
```



## Customization
The Serverless Framework provides out-of-the box support for provisioning some AWS services, but this mainly serves the purpose of abstracting away the details of Lambda + API Gateway.  Fine grained control over your entire AWS Environment is possible by utilizing the `resources` section, and including raw CloudFormation YAML.  In this context, Serverless Framework serves as more of a build and deploy tool, facilitating process of deployment.  This is in contrast to opinionated development frameworks.  

Additional customization of the framework itself is enabled thru the plugin system.

## Additional Documentation and Resources
For additional information on the framework, the concepts, and features:
* [Official Serverless Framework Documentation](https://serverless.com/framework/docs/)
* [Official Enterprise Vendor Support](https://serverless.com/enterprise/)
* [Blog](https://serverless.com/blog/)
* [Official Community Forums](https://forum.serverless.com/)
* [Github Repo](https://github.com/serverless/serverless)


# Troubleshooting

## Network timeout / errors from serverless CLI
This most likely is an indication that you are on a corporate network and the HTTP/HTTPS proxies need to be set in your.  In Powershell:
```powershell
>  $env:HTTP_PROXY = 'PROXY_URL'
>  $env:HTTPS_PROXY = 'PROXY_URL'
```
 

