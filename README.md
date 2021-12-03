Use APIs to integrate external systems with Amazon Honeycode
Amazon Honeycode gives you the power to build custom apps without programming. Amazon Honeycode APIs help you extend Honeycode beyond its existing capabilities and to integrate Honeycode with your existing systems. Honeycode APIs do require IT level skills to set them up. 
Honeycode Table APIs let you query the meta data such as listing tables within workbooks, perform batch operations on your table data such as adding, updating and deleting data in your Honeycode tables, and to import data from external sources. You can find a full list of Honeycode APIs here
Prerequisite
If you don’t have one already, create a new Honeycode account and login to your Honeycode account. To get started with Honeycode API you need to link your Honeycode team with your AWS Account
Overview
In this sample code, I setup a light weight customer relationship management app using Amazon Honeycode and and then deploy a serverless API application to integrate and extend my Honeycode app. Here is an overview of the the application:  
Create a Customer Tracker app using Amazon Honeycode
Honeycode provides templates for a number of common use cases. Using a Honeycode template creates a workbook with the required tables to store your data, as well as apps and automations with the basic functionality for that use case. You can then customize the app to meet your needs! Here are the steps to create a Customer Tracker app in Amazon Honeycode
1.	Create a new Workbook using the Customer Tracker template.
Note: If you are part of multiple teams in Honeycode, make sure to create the Workbook in the team which is linked to your AWS account.
2.	Open Tables > D_History table and add a new column called Exported. You use this column to indicate if a row of contact history has been stored in the S3 bucket
3.	Open Builder > Customer Tracker app, right click on any screen, click on Get ARN and IDs, and copy the Workbook ID. For help with this step refer to the Getting started with APIs article in Honeycode Community
Create a Serverless API application to extract data tables from Honeycode and store them in s3 as .csv
In this section, you create a Serverless API application to export customer contacts from Honeycode. After the customer contact history is exported to Amazon S3, the API application updates the Exported column in Honeycode with the current date. The API application is deployed using AWS CDK, which let you manage your infrastructure using code. The API application includes the ExportContactHistoryS3 Lambda function (invoked every minute) to read the latest customer contact history and store them as a CSV file in a S3 bucket. Here are the steps to deploy the serverless API application
1.	Create/Open a AWS Cloud9 IDE instance. This may take a few minutes to complete. I recommend the following configuration for this lab: 
o	Name: Honeycode API Lab
o	Environment type: Create a new EC2 instance for environment (direct access)
o	Instance type: t2.micro (1 GiB RAM + 1 vCPU)
o	Platform: Amazon Linux 2 (recommended)
o	Cost-saving setting: After 30 minutes (default)
Note: If you’d like to use your own machine for deploying the API application, you can do so by following the instructions to install and configure AWS CDK
2.	Download the source package by running the following commands on the Cloud9 terminal
git clone https://github.com/KristopherGant/Export-HoneyCode-data.git
cd amazon-honeycode-table-api-integration-sample
Note: Open bin/honeycode-api-lab.js to view the name of the stack and rename the stack from HoneycodeLab to say JohnHoneycodeLab by adding your first name so it is easier to identify the resources that you create
3.	Open lambda/env.json update the workbookId with the value copied from your Customer Tracker Honeycode app 
o	lambda/env.json
4.	Now in the same lambda/env.json update the TableName with D_History or of the table name referring to the data table you want to store in your s3 bucket. 
5.	Run the following commands to start the deployment. This will take a few minutes to complete.
npm install -gf aws-cdk
npm install
cdk bootstrap
cdk deploy
Note: Running cdk deploy will do the following
•	Create the S3 buckets
•	Create the Lambda functions
•	Create the event source for the Lambda functions
•	Add permissions for the Lambda to access Honeycode
6.	Once the deployment is complete, wait a couple of minutes and then verify the application is running correctly by opening the S3 bucket with the name *honeycodelab-contacthistorybucket* in your Amazon S3 console, open the directories under that bucket to download/open the CSV file containing the Contact History records. You can add new contact history by opening the Honeycode Customer Tracker app, selecting a customer and adding a new contact detail. Newly added contact history data should appear in the S3 bucket as a new CSV file in about a minute.
Use this serverless API function with other Honeycode applications. 
If you already have a Honeycode application or you if you would like to create a Honeycode application that is different than the previous example, you are able to accomplish this with the following adjustments:
1.	Get the workbookId associated with the Honeycode project you would like to work with.
-	https://docs.aws.amazon.com/honeycode/latest/UserGuide/table-row-operations-arns-and-ids.html
2.	Get the TableName associated with the Honeycode data table you would like to store in s3.
-	https://docs.aws.amazon.com/honeycode/latest/UserGuide/API_ListTables.html
-	Note: if the “list-tables” command does not work for you try switching to the us-west-2 region to run the command and then revert back to your working region. This can be accomplished with the following command:
export AWS_DEFAULT_REGION=us-west-2
export AWS_DEFAULT_REGION=”your region”
3.	Open lambda/env.json update the workbookId and the TableName with their respective values and save the updates
4.	Run the following commands to start the deployment. This will take a few minutes to complete.
		npm install -gf aws-cdk
		npm install
		cdk bootstrap
		cdk deploy
5.	Verify that the table was exported successfully just like step 6 in the previous example. 
Cleanup
1.	Remove the Serverless API application by running the following command in your Cloud9 IDE
cdk destroy
2.	Delete the Cloud9 IDE by opening the Cloud9 console and clicking on Delete
