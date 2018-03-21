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

## Creating and Deploying a simple REST API
First create a new directory and init npm (or hand craft a `package.json`):

```powershell
> mkdir express-app ; cd express-app
> npm init -f
```

Install a few dependencies.  The `express` framework and [`serverless-http`](https://github.com/dougmoscrop/serverless-http).  This combination of modules will provide a very similar experience to hosted Node.js + express apps:

```powershell
> npm install --save express serverless-http
```
The `serverless-http` module servers as the "glue" between the Node.js app and the AWS API Gateway.

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

Last but not least, create a `serverless.yml` file.  This serves as the configuration file to drive the Serverless CLI.  The configuration details within the file represent your high-level **Infrastructure as Code**.  More on this later, but simple, concise config parameters here drive the creation and of an AWS CloudFormation template.  CloudFormation Templates serve as your lower-level AWS **Infrastructure as Code** and can be rather esoteric.

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


# Extra Guides
Additional guides can be found on serverless.com that demonstrate the power and flexiblity of the framework.  [Here](https://serverless.com/blog/serverless-express-rest-api/), you can easily deploy a simple REST API with two endpoints via expressjs and perform CRUD like operations against a DynamoDB store.

#Troubleshooting

## Network timeout / errors in serverless framework CLI
This most likely is an indication that you are on a corporate network and the HTTP/HTTPS proxies need to be set in your shell / env.  In Powershell:
```powershell
>  $env:HTTP_PROXY = 'PROXY_URL'
>  $env:HTTPS_PROXY = 'PROXY_URL'
```
#### TODO 

#### TODO CLEANUP 
- >Launch a shell with elevated privileges (Ctrl + Win key -> type 'Powershell' -> right click -> Run as Administrator -> enter adminstrator credentials)
 
