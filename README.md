## Amazon Rekognition Engagement Meter

The Engagement Meter is a web application that calculates and displays engagement levels of an audience observed by a webcam. It also includes the capability to recognize attendants by associating their faces to individual user profiles.

[![Build Status](https://travis-ci.org/aws-samples/amazon-rekognition-engagement-meter.svg?branch=master)](https://travis-ci.org/aws-samples/amazon-rekognition-engagement-meter)


### Index

* [Architecture](#architecture)
* [Usage](#usage)
  * [Prerequisites](#prerequisites)
  * [Deployment](#deployment)
  * [Accessing the application](#accessing-the-application)
* [Remove the application](#remove-the-application)
* [Contributing](#contributing)

### Architecture

The Engagement Meter uses [Amazon Rekognition](https://aws.amazon.com/rekognition) for image and sentiment analysis, [Amazon DynamoDB](https://aws.amazon.com/dynamodb) for storage, [Amazon API Gateway](https://aws.amazon.com/api-gateway) and [Amazon Cognito](https://aws.amazon.com/cognito) for the API, and [Amazon S3](https://aws.amazon.com/s3), [AWS Amplify](https://aws.amazon.com/amplify), and [React](https://reactjs.org) for the front-end layer.

<img src="docs/amazon-rekognition-1.png" alt="Architecture Diagram" />

There are three main user flows:
* the **"add user"** flow (*yellow*) is triggered when clicking the *"Add user"* button
* the **"added users recognition"** flow (*green*) and the **"sentiment analysis"** flow (*blue*) are both triggered when clicking the *"Start Rekognition"* button and repeat until the *"Stop Rekognition"* button is clicked.

The diagram below represents the API calls performed by Amplify, which takes care of authenticating all the calls to the API Gateway using Cognito.

<img src="docs/amazon-rekognition-2.png" alt="User flow" />

#### The "add user" flow (*yellow*)

Amplify makes a `POST /faces/add` request to the API Gateway including the uploaded picture and an autogenerated unique identifier (known as *ExternalImageId*), then the API Gateway calls the `IndexFaces` action in Amazon Rekognition. After that, Amplify makes a `POST /people` request to the API Gateway including the ExternalImageId and some extra metadata (Name and Job Title), then the API Gateway writes that data to the `Faces` table in Amazon DynamoDB. To learn more about *IndexFaces* [see the Rekognition documentation](https://docs.aws.amazon.com/rekognition/latest/dg/API_IndexFaces.html).

#### The "added users recognition" flow (*green*)

Amplify makes a `GET /people` request to the API Gateway, which then queries the `Faces` table on Amazon DynamoDB. In case any people have been registered, Amplify makes another call to `POST /faces/search` including a screenshot detected from the webcam. Then, the API Gateway calls the `SearchFacesByImage` action in Amazon Rekognition. If any previously registered person is recognized, the service provides details about the matches, including each face's coordinate and confidence. In this case, the UI displays a welcome message showing the recognized users' names. To learn more about SearchFacesByImage [see the Rekognition documentation](https://docs.aws.amazon.com/rekognition/latest/dg/API_SearchFacesByImage.html).

#### The "sentiment analysis" flow (*blue*)

Amplify makes two parallel calls to the API Gateway (here represented in a sequential manner for simplicity). First, Amplify makes a `POST /faces/detect` request with a screenshot detected from the webcam to the API Gateway, which then calls the `DetectedFaces` action on Amazon Rekognition. If any face is detected, the service provides details about the matches, including physical characteristics and sentiments. In that case, a little recap is shown on the UI for each recognized person. Then Amplify makes a `POST /engagement` request with some of the recognized sentiments (Angry, Confused, Happy, Sad, Surprised) to the API Gateway, which writes that data to the `Sentiment` table in DynamoDB. In parallel, Amplify makes a `GET /engagement` request to the API Gateway, which then queries the `Sentiment` table in DynamoDB to retrieve an aggregate for all the sentiments recorded during the last hour, in order to calibrate and draw the meter. To learn more about DetectFaces [see the Rekognition documentation](https://docs.aws.amazon.com/rekognition/latest/dg/API_DetectFaces.html).


### Usage

#### Prerequisites

To deploy the sample application you will require an AWS account. If you don’t already have an AWS account, create one at <https://aws.amazon.com> by following the on-screen instructions. Your access to the AWS account must have IAM permissions to launch AWS CloudFormation templates that create IAM roles.

To use the sample application you will require a [modern browser](https://caniuse.com/#feat=stream) and a webcam.

#### Deployment

The demo application is deployed as an [AWS CloudFormation](https://aws.amazon.com/cloudformation) template.

> **Note**  
You are responsible for the cost of the AWS services used while running this sample deployment. There is no additional cost for using this sample. For full details, see the pricing pages for each AWS service you will be using in this sample. Prices are subject to change.

1. Deploy the latest CloudFormation template by following the link below for your preferred AWS region:

|Region|Launch Template|
|------|---------------|
|**US East (N. Virginia)** (us-east-1) | [![Launch the EngagementMeter Stack with CloudFormation](docs/deploy-to-aws.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=EngagementMeter&templateURL=https://s3-eu-west-1.amazonaws.com/rekognition-engagement-meter/template.yaml)|
|**US East (Ohio)** (us-east-2) | [![Launch the EngagementMeter Stack with CloudFormation](docs/deploy-to-aws.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-2#/stacks/new?stackName=EngagementMeter&templateURL=https://s3-eu-west-1.amazonaws.com/rekognition-engagement-meter/template.yaml)|
|**US West (Oregon)** (us-west-2) | [![Launch the EngagementMeter Stack with CloudFormation](docs/deploy-to-aws.png)](https://console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/new?stackName=EngagementMeter&templateURL=https://s3-eu-west-1.amazonaws.com/rekognition-engagement-meter/template.yaml)|
|**Asia Pacific (Seoul)** (ap-northeast-2) | [![Launch the EngagementMeter Stack with CloudFormation](docs/deploy-to-aws.png)](https://console.aws.amazon.com/cloudformation/home?region=ap-northeast-2#/stacks/new?stackName=EngagementMeter&templateURL=https://s3-eu-west-1.amazonaws.com/rekognition-engagement-meter/template.yaml)|
|**Asia Pacific (Sydney)** (ap-southeast-2) | [![Launch the EngagementMeter Stack with CloudFormation](docs/deploy-to-aws.png)](https://console.aws.amazon.com/cloudformation/home?region=ap-southeast-2#/stacks/new?stackName=EngagementMeter&templateURL=https://s3-eu-west-1.amazonaws.com/rekognition-engagement-meter/template.yaml)|
|**Asia Pacific (Tokyo)** (ap-northeast-1) | [![Launch the EngagementMeter Stack with CloudFormation](docs/deploy-to-aws.png)](https://console.aws.amazon.com/cloudformation/home?region=ap-northeast-1#/stacks/new?stackName=EngagementMeter&templateURL=https://s3-eu-west-1.amazonaws.com/rekognition-engagement-meter/template.yaml)|
|**EU (Ireland)** (eu-west-1) | [![Launch the EngagementMeter Stack with CloudFormation](docs/deploy-to-aws.png)](https://console.aws.amazon.com/cloudformation/home?region=eu-west-1#/stacks/new?stackName=EngagementMeter&templateURL=https://s3-eu-west-1.amazonaws.com/rekognition-engagement-meter/template.yaml)|

2. If prompted, login using your AWS account credentials.
1. You should see a screen titled "*Create Stack*" at the "*Specify template*" step. The fields specifying the CloudFormation template are pre-populated. Click the *Next* button at the bottom of the page.
1. On the "*Specify stack details*" screen you may customize the following parameters of the CloudFormation stack:
   * **Stack Name:** (Default: EngagementMeter) This is the name that is used to refer to this stack in CloudFormation once deployed. The value must be 15 characters or less.
   * **CollectionId:** (Default: RekogDemo) AWS Resources are named based on the value of this parameter. You must customise this if you are launching more than one instance of the stack within the same account.

   When completed, click *Next*
1. [Configure stack options](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-add-tags.html) if desired, then click *Next*.
1. On the review you screen, you must check the boxes for:
   * "*I acknowledge that AWS CloudFormation might create IAM resources*" 
   * "*I acknowledge that AWS CloudFormation might create IAM resources with custom names*" 

   These are required to allow CloudFormation to create a Role to allow access to resources needed by the stack and name the resources in a dynamic way.
1. Click *Create Change Set* 
1. On the *Change Set* screen, click *Execute* to launch your stack.
   * You may need to wait for the *Execution status* of the change set to become "*AVAILABLE*" before the "*Execute*" button becomes available.
1. Wait for the CloudFormation stack to launch. Completion is indicated when the "Stack status" is "*CREATE_COMPLETE*".
   * You can monitor the stack creation progress in the "Events" tab.
1. Note the *url* displayed in the *Outputs* tab for the stack. This is used to access the application.

#### Accessing the Application

The application is accessed using a web browser. The address is the *url* output from the CloudFormation stack created during the Deployment steps.

* When accessing the application, the browser will ask you the permission for using your camera. You will need to click "*Allow*" for the application to work.
* Click "*Add a new user*" if you wish to add new profiles.
* Click "*Start Rekognition*" to start the engine. The app will start displaying information about the recognized faces and will calibrate the meter.

### Remove the application

To remove the application open the AWS ClodFormation Console, click the Engagement Meter project, right-click and select "*Delete Stack*". Your stack will take some time to be deleted. You can track its progress in the "Events" tab. When it is done, the status will change from DELETE_IN_PROGRESS" to "DELETE_COMPLETE". It will then disappear from the list.

## Contributing

Contributions are more than welcome. Please read the [code of conduct](CODE_OF_CONDUCT.md) and the [contributing guidelines](CONTRIBUTING.md).

## License Summary

This sample code is made available under a modified MIT license. See the LICENSE file.
