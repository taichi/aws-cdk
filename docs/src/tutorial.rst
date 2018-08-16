.. Copyright 2010-2018 Amazon.com, Inc. or its affiliates. All Rights Reserved.

   This work is licensed under a Creative Commons Attribution-NonCommercial-ShareAlike 4.0
   International License (the "License"). You may not use this file except in compliance with the
   License. A copy of the License is located at http://creativecommons.org/licenses/by-nc-sa/4.0/.

   This file is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND,
   either express or implied. See the License for the specific language governing permissions and
   limitations under the License.

.. _tutorial:

##############
|cdk| Tutorial
##############

This topic is a step-by-step tutorial that creates an apple dispensing service
using the |cdk|.

The tutorial uses the following services:

- |APIG| to interface our |Lambda| code with ???
- |IAM| to create a role to allow |LAMBDA| to ???
- |S3| to store ...
- |LAMBDA| to ...

.. _step_zero:

Step Zero: Install the |cdk|
===========================

If you haven't installed the |cdk|,
run the following command.

.. code-block: sh

   npm i -g aws-cdk

Once you've installed the |cdk|, run **cdk --version**
and make sure it matches the version you installed.

.. _step_one:

Step One: Create a New |cdk| Project
====================================

Create an *apples* directory and navigate into it.

.. tabs::

   .. code-block:: sh

      mkdir apples
      cd apples

Create the **apples** project in TypeScript.

.. code-block:: sh

   cdk init -l typescript

This creates a skeleton |cdk| app with the following files and directories (folders on Windows):

- README.md contains some useful information about how to interact with your new package
- bin contains the source file *apples.ts*
- cdk.json contains a directive so that your **cdk** commands need not include a **-a** option
- node_modules contains the (3500+) Node.js library files
- package-lock.json is the npm lock file for *package.json*
- package.json contains metadata for the app
- tsconfig.json contains configuration information for the TypeScript compilerOptions

.. _step_two:

Step Two: Create an Apple Service
=================================

Create a *lib* directory at the same level as the *bin* directory,
then create the file *AppleService.js* in the *lib* directory
with the following code:

.. code-block:: ts

   import cdk = require('@aws-cdk/cdk');

   export class AppleService extends cdk.Construct {
       constructor(parent: cdk.Construct, name: string) {
           super(parent, name);
       }
   }

.. _step_three:

Step Three: Add an Apple Service to our Stack
=============================================

Replace the **import** statements for |SNS| and |SQS| with the following:

.. code-block: ts

   import appleservice = require('../lib/AppleService');

Replace the |SNS| topic and |SQS| queue in **AppleStack** with the following code:

.. code-block:: ts

   new appleservice.AppleService(this, 'Apples');

.. _step_four:

Step Four: Add an |S3| Bucket to the Apple Service
==================================================

Our service is pretty useless, so to make it useful,
add an |S3| bucket to the service so we can store our apples.

Run the following command to install the core
|S3| library:

.. code-block: sh

   npm i @aws-cdk/aws-s3

Add the following **import** statement to *AppleService.ts*:

.. code-block: ts

   import s3 = require('@aws-cdk/aws-s3');

And add the following code to the end of the constructor for the **AppleService** class:

.. code-block: ts

   new s3.Bucket(this, 'AppleStore');

Let's see what we have so far.
Run the following command to see the current |CFN| template:

.. code-block: sh

   cdk synth

Oops, what happened?
We forgot to compile our TypeScript code into JavaScript.
To avoid this,
run the following command in a separate window to
automatically compile our TypeScript code into JavaScript as we save the file:

.. code-block: sh

   npm run watch

Rerun **cdk synth**.
You should see something like the following,
where ABC123YZ is an eight-character value that the |cdk|
creates to ensure that the resource is unique:

.. code-block: yaml

   Resources:
       ApplesAppleStoreABC123YZ:
           Type: 'AWS::S3::Bucket'
       CDKMetadata:
           Type: 'AWS::CDK::Metadata'
           Properties:
               Modules: '@aws-cdk/aws-kms=0.8.0,@aws-cdk/aws-s3=0.8.0,@aws-cdk/cdk=0.8.0,@aws-cdk/cx-api=0.8.0,apples=0.1.0,js-base64=2.4.5'

.. _step_five:

Step Five: Add |ABP| ??? to the Apple Service
=======================================================

Add |ABP| to the service so we can ???
and |IAM| so we can allow |ABP| to ???.

Run the following command to install the |ABP| and |IAM| libraries:

.. code-block: sh

   npm i @aws-cdk/aws-apigateway @aws-cdk/aws-iam

Add the following **import** statement to *apple_service.ts*:

.. code-block: ts

   import apigateway = require('@aws-cdk/aws-apigateway');
   import iam = require('@aws-cdk/aws-iam');

And add the following code to the end of the constructor for the **AppleService** class:

.. code-block: ts

   new apigateway.cloudformation.RestApiResource(this, 'ApplesApi', {
       restApiName: 'Apple Service',
       description: 'Serves up apples'
   })

   const servicePrincipal = new cdk.ServicePrincipal('apigateway.amazon.com');

   new iam.Role(this, 'IntegrationRole', {
       assumedBy: servicePrincipal
   });

Let's see what we have so far.
Run the following command to see the current |CFN| template:

.. code-block: sh

   cdk synth

You should see something like the following:

.. code-block: yaml

   Resources:
       ApplesAppleStoreE5F233DA:
           Type: 'AWS::S3::Bucket'
       ApplesApplesApi99ED60FA:
           Type: 'AWS::ApiGateway::RestApi'
           Properties:
               Description: 'Serves up apples'
               Name: 'Apple Service'
       ApplesIntegrationRoleDE27B761:
           Type: 'AWS::IAM::Role'
           Properties:
               AssumeRolePolicyDocument:
                   Statement:
                       -
                           Action: 'sts:AssumeRole'
                           Effect: Allow
                           Principal:
                               Service: apigateway.amazon.com
                   Version: '2012-10-17'
       CDKMetadata:
           Type: 'AWS::CDK::Metadata'
           Properties:
               Modules: '@aws-cdk/aws-apigateway=0.8.0,@aws-cdk/aws-iam=0.8.0,@aws-cdk/aws-kms=0.8.0,@aws-cdk/aws-s3=0.8.0,@aws-cdk/cdk=0.8.0,@aws-cdk/cx-api=0.8.0,apples=0.1.0,js-base64=2.4.5'

.. _step_six:

Step Six: Add a |LAMBDA| Function to the Apple Service
======================================================

Create a *resource* directory at the same level as *bin* and *lib*
and create the file *apples.js* in that directory with the following content:

.. code-block: js

   // apple.js
   const AWS = require('aws-sdk');
   const S3 = new AWS.S3();

   const bucketName = process.env.BUCKET;

   exports.main = function(event, context, callback) {
       switch (event.operation) {
           case "list":
               S3.listObjectsV2({ Bucket: bucketName })
                   .promise()
                   .then(function(data) {
                       callback(null, { apples: data.Contents.map(function(e) { return e.Key }) });
                   })
                   .catch(rejectedPromise);
               break;
           case "create":
               S3.putObject({
                   Bucket: bucketName,
                   Key: event.apple.name,
                   Body: JSON.stringify(event.apple, null, 2),
                   ContentType: 'application/json'
               }).promise()
                   .then(function() { callback(null, event.apple); })
                   .catch(rejectedPromise);
               break;
           case "show":
               S3.getObject({ Bucket: bucketName, Key: event.name})
                   .promise()
                   .then(function(data) {
                       callback(null, JSON.parse(data.Body.toString('utf-8')));
               })
               .catch(rejectedPromise);
               break;
           case "delete":
               S3.deleteObject({ Bucket: bucketName, Key: event.name })
                   .promise()
                   .then(function(data) {
                       callback(null, { success: true });
               })
               .catch(rejectedPromise);
               break;
           default:
               return callback("Unknown operation: " + event.operation, null);
       }

       function rejectedPromise(error) {
           callback(error.stack || JSON.stringify(error, null, 2), null);
       }
   }
