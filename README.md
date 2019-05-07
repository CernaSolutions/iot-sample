# Internet of Things!

**Difficulty**: 2/10
**Pre-reqs**: NodeJS
**Language**: JS/Proprietary

## Description

- IoT extends the internet into physical devices
- Needs a system to interconnect those different devices
- Node-RED is a tool to build workflows between IoT Devices and Services
- Can run on a variety of devices, OSes
- Community Driven
- Hundreds of plugins for various services

## Setup (https://nodered.org/docs/getting-started/installation)

- Install NodeJS (https://nodejs.org/dist/latest-v8.x)
- Run `npm install -g --unsafe-perm node-red`
- Run `node-red`
- Creates a local Web Interface (8080)

## Instructions

- Add an **Inject** node

- Add a **Function** node

  - Wire the Inject to this node

  - Set the **Function** to 
    msg.payload = {
        short_description: 'Created from Node Red'
    }

    return msg;	

- Add an **HTTP Request** node

  - Wire the Function to this node
  - Set **Method** to `Post`
  - Set **URL** to a ServiceNow REST endpoint
  - Set **Authentication** to `Basic`
    - Fill in authentication details
  - Set **Return** to 'a parsed JSON object'

- Add a **Debug** node

  - Wire the Inject to this node

- Click **Deploy**

- Click the **Inject** Button

- Verify an Incident has been created in ServiceNow

# AWS IoT Button

Difficulty: 5/10
Pre-reqs: Node-RED, AWS Account, AWS IoT Enterprise Button
Language: JS/Proprietary

## Description

- Create an Incident via AWS IoT Enterprise Button

## Instructions

- Unpackage IoT Button

- Install AWS 1-Click App

  - Set AWS Region to US East (N. Virginia)
  - Claim Device and Connect it to the internet following instructions in App

- In AWS go to Service: Simple Queue Service (Make sure you are in US East - N. Virginia)

  - Create a new Standard Queue called 'IoT'
  - Take note of the URL and ARN values under the Queue Details

- In AWS go to Service: Lambda

  - Create a new function
  - Author from Scratch
    - Set **Function name** to 'sendIotMessageToQueue'
    - Set **Runtime** to 'Node.js 8.1.0'
  - Execution role: Create a new role with basic Lambda permissions

- In AWS go to Service: IAM

  - Go to Roles on the left hand menu
  - Find the 'sendIotMessageToQueue' role that was created
  - Click **Attach Policies**
  - Search for the 'AWSLambdaSQSQueueExecutionRole' policy
  - Check the item in the list
  - Click **Attach Policy**
  - Click **Attach Policies** again
  - Click **Create policy** (a second tab will open)
  - Set **Service**  to 'SQS'
  - Set **Write** to 'SendMessage'
  - Expand Resources
  - Check 'Any' (alternatively you can find the IoT Queue ARN and add that specifically)
  - Click **Review Policy**
  - Set **Name** to 'SQSSendMessage'
  - Click **Create Policy**
  - Return to the original tab and click the **refresh button**
  - Search for the 'SQSSendMessage' Policy
  - Check the item in the list
  - Click **Attach Policy**

- In AWS go back to Service: Lambda

  - Find the sendIotMessageToQueue function

  - Amazon SQS should now be listed as Resource on the right of the screen (Typically under Amazon CloudWatch Logs)

  - Click the 'sendIotMessageToQueue' header to show the Function code

  - Set the code to:
    var QUEUE_URL = 'QUEUE URL GOES HERE';
    var AWS = require('aws-sdk');
    var sqs = new AWS.SQS({region : 'us-east-1'});

    exports.handler = function(event, context) {
      var params = {
        MessageBody: 'An urgent alert has been triggered from Room ' + event.placementInfo.attributes.Room,
        QueueUrl: QUEUE_URL
      };

      sqs.sendMessage(params, function(err,data){
        if(err) {
          console.log('error:',"Fail Send Message" + err);
          context.done('error', "ERROR Put SQS");  // ERROR with message
        }else{
          console.log('data:',data.MessageId);
          context.done(null,'');  // SUCCESS 
        }
      });
    }

  - Replace **QUEUE URL GOES HERE** with the SQS Queue URL you noted earlier

  - Save the function

  - Select **Configure test events** from the top bar

    - Set **Event Name** to 'iotTest'
    - Set the **Event body** to
      {
          "deviceInfo": {
              "deviceId": "XXXXXXXXXXXXXXXX",
              "type": "button",
              "remainingLife": 99.995,
              "attributes": {
                  "projectRegion": "us-east-1",
                  "projectName": "CreateIncident",
                  "placementName": "Office",
                  "deviceTemplateName": "CreateIncident"
              }
          },
          "deviceEvent": {
              "buttonClicked": {
                  "clickType": "SINGLE",
                  "reportedTime": "2019-04-25T20:02:19.505Z"
              }
          },
          "placementInfo": {
              "projectName": "CreateIncident",
              "placementName": "Office",
              "attributes": {
                  "Room": "237"
              },
              "devices": {
                  "CreateIncident": "XXXXXXXXXXXXXXXX"
              }
          }
      }

  - Select 'iotTest' and click '**Test**'

- In AWS go back to Service: Simple Queue Service

  - Verify there is 1 Message Available in the IoT Queue

- In Node-RED go to Settings > Manage Palette

  - Select the 'Install' tab
  - Filter for AWS and add **node-red-contrib-aws** version 0.5.0
  - Install the plugin

- Add an **Inject** node

  - Set **Repeat** to 'interval', every '30', 'seconds'
  - Set **Name** to 'Check For Messages'

- Add an 'AWS SQS' node

  - Wire 'Check For Messages' to the node
  - Set 'AWS' to a new Credential
    - In AWS go to 'My Security Credentials'
    - Click '**Create access key**'
    - Copy Access ID to NodeRed AWS Credentials
    - Copy Secret Key to NodeRed AWS Credentials
  - Set **Operation** to 'ReceiveMessage'
  - Set **Name** to 'IoT'
  - Set **QueueUrl** to the AWS SQS Queue URL 

- Add a 'Function' node

  - Set **Name** to 'Create Incident From Emergency Button'

  - Set **Function** to

    if(typeof msg.payload.Messages !== 'undefined'){
		    msg.payload = {
		       short_description: msg.payload.Messages[0].Body
		    };
		    return [msg,null];
		} else {
		    return [null, msg];    
		}

  - Set 'Outputs' to '2'

  - Wire output one to to the 'HTTP Request' node

- Add a 'Debug' node 

  - Wire 'Create Incident From Emergency Button' output 2 to this node
  - Set 'Name' to 'Error'

- Click **Deploy**

- Click the IoT Button (finally)

- Verify the Incident is created in ServiceNow