# TechConf Registration Website

## Project Overview
The TechConf website allows attendees to register for an upcoming conference. Administrators can also view the list of attendees and notify all attendees via a personalized email message.

The application is currently working but the following pain points have triggered the need for migration to Azure:
 - The web application is not scalable to handle user load at peak
 - When the admin sends out notifications, it's currently taking a long time because it's looping through all attendees, resulting in some HTTP timeout exceptions
 - The current architecture is not cost-effective 

In this project, you are tasked to do the following:
- Migrate and deploy the pre-existing web app to an Azure App Service
- Migrate a PostgreSQL database backup to an Azure Postgres database instance
- Refactor the notification logic to an Azure Function via a service bus queue message

## Dependencies

You will need to install the following locally:
- [Postgres](https://www.postgresql.org/download/)
- [Visual Studio Code](https://code.visualstudio.com/download)
- [Azure Function tools V3](https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local?tabs=windows%2Ccsharp%2Cbash#install-the-azure-functions-core-tools)
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
- [Azure Tools for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack)

## Project Instructions

### Part 1: Create Azure Resources and Deploy Web App
1. Create a Resource group
2. Create an Azure Postgres Database single server
   - Add a new database `techconfdb`
   - Allow all IPs to connect to database server
   - Restore the database with the backup located in the data folder
3. Create a Service Bus resource with a `notificationqueue` that will be used to communicate between the web and the function
   - Open the web folder and update the following in the `config.py` file
      - `POSTGRES_URL`
      - `POSTGRES_USER`
      - `POSTGRES_PW`
      - `POSTGRES_DB`
      - `SERVICE_BUS_CONNECTION_STRING`
4. Create App Service plan
5. Create a storage account
6. Deploy the web app

### Part 2: Create and Publish Azure Function
1. Create an Azure Function in the `function` folder that is triggered by the service bus queue created in Part 1.

      **Note**: Skeleton code has been provided in the **README** file located in the `function` folder. You will need to copy/paste this code into the `__init.py__` file in the `function` folder.
      - The Azure Function should do the following:
         - Process the message which is the `notification_id`
         - Query the database using `psycopg2` library for the given notification to retrieve the subject and message
         - Query the database to retrieve a list of attendees (**email** and **first name**)
         - Loop through each attendee and send a personalized subject message
         - After the notification, update the notification status with the total number of attendees notified
2. Publish the Azure Function

### Part 3: Refactor `routes.py`
1. Refactor the post logic in `web/app/routes.py -> notification()` using servicebus `queue_client`:
   - The notification method on POST should save the notification object and queue the notification id for the function to pick it up
2. Re-deploy the web app to publish changes

## Monthly Cost Analysis
Complete a month cost analysis of each Azure resource to give an estimate total cost using the table below:

| Service type                    | Region  | Description                                                                                                               | Estimated monthly cost |
|---------------------------------|---------|---------------------------------------------------------------------------------------------------------------------------|------------------------|
| App Service                     | East US | Free Tier; 1 F1 (0 Core(s), 1 GB RAM, 1 GB Storage) x 729 Hours; Linux OS                                                 | $0.00                  |
| Azure Database for PostgreSQL   | East US | Single Server Deployment, Basic Tier, 1 Gen 5 (1 vCore) x 730 Hours, 5 GiB Storage, 100 GiB Additional Backup storage - LRS redundancy | $35.32                 |
| Service Bus                     | East US | Basic tier: 1 million messaging operations                                                                                | $0.05                  |
| Azure Functions                 | East US | Consumption tier, Pay as you go, 128 MB memory, 100 milliseconds execution time, 5 executions/mo                          | $0.00                  |


The app service includes the web app and the function app.

## Architecture Explanation
In this project I used 4 services: App service, Service bus, Azure Postgres, Azure function

App service: the pre-existing TechConf web all is already provided so I used app service to deploy quickly and the traffic is not really high so I can use free tier for cost-optimization

Service bus and Azure function: Azure service bus (Basic tier) cost $0.05 per million operations, almost free for this application and Azure function includes a monthly free grant of 1 million requests. Aditionally, the notification or sending email are moved to background jobs in Azure so I don't need to loop in all attendees to do something and the result is better performance

Azure Postgres: the most expensive service in this project. I choose this because it offers a high availability SLA of up to 99.99% and built-in automation for database maintenance, patching,...
