---
title: How to build an application in SAP HANA, express edition and deploy it to SAP Cloud Platform
description: This tutorial will show you how to develop and deploy anywhere with any kind of data, in any programming language, with SAP HANA, express edition.
tags: [  tutorial>intermediate, products>sap-hana\,-express-edition ]
---

## Prerequisites  
 - [Decide an installation option for SAP HANA, express edition](https://www.sap.com/developer/how-tos/2016/09/hxe-ua-version.html)
 - [Binary install (for custom install on Linux machines)](https://www.sap.com/developer/tutorials/hxe-ua-installing-binary.html)
 - [Virtual Machine image install (for VMs on Windows/Mac OS X, or prebuilt install on Linux machines)](https://www.sap.com/developer/tutorials/hxe-ua-installing-vm-image.html)

## Next Steps
 - Go to the Tutorials section on the [SAP HANA, Express Edition Developer Page](https://www.sap.com/developer/topics/sap-hana-express.tutorials.html)

## Details

### You will learn
How to build a sample Python application with SAP HANA, express edition, and deploy it to a cloud platform like the SAP Cloud Platform. This tutorial uses an environment that has SAP HANA, express edition pre-installed, with a preloaded sample dataset. The goal here is to develop a service that quickly assesses the trustworthiness or credit rating of the customers of a bank. Everything runs on a single, mobile device: the HANA database, the HANA XS Application Server, as well as WebIDE.

### Time to Compete
**15 min**

---

#Part A: Set up the development environment

[ACCORDION-BEGIN [Step 1: ](Import Code)]

###Import Code

1. Clone the service and UI code from `git` to Web IDE:

    - [Python Service](https://github.wdf.sap.corp/hana-demo-team/DKOM_PYTHON_SERVICE.git)
    - [Minimized Source Code](https://github.wdf.sap.corp/hana-demo-team/dkom_minimized.git)

[ACCORDION-END]

[ACCORDION-BEGIN [Step 2: ](Database)]

###Database

1. Execute the queries present on `data.sql` and `insert.sql` (`DKOM_PYTHON_SERVICE` project) in SAP HANA Studio or in Web IDE.

[ACCORDION-END]

[ACCORDION-BEGIN [Step 3: ](Create user provided service)]

###Create user provided service

The user provided service gets access to data foreign to the XSA application container. It needs to assign a database user that has authorization with grant option to the foreign schema. The `data.sql` script from the last action already assigns these authorizations to `XSA_ADMIN`.
    
1. Execute the following script to create the user provided service (if it is already created, check by typing `xs s` and look for `CROSS_SCHEMA_ACCESS_SERVICE`)
    
    `xs cups CROSS_SCHEMA_ACCESS_SERVICE -p`
    `'{"host":"<host>","port":"<port>","user":"XSA_ADMIN","password":"<password>","driver":"com.sap.db.jdbc.Driver","tags":["hana"] , "schema":"<schema>" }'`
    
    `<host>`: `host name or machine IP address`
    
    `<port>`: `3<instance number><13 multi containers or 15 single containers>`
    
    `cups` is used to create a new service, and `uups` to update a new service.

[ACCORDION-END]

[ACCORDION-BEGIN [Step 4: ](Create Python Buildpack)]

###Create Python Buildpack

1. Clone the Python buildpack from `git`
    `git clone https://github.wdf.sap.corp/xs2/sap_python_buildpack.git`
    
2. Install the buildpack
    `xs create-buildpack python_buildpack /buildpackpath/ 3 –enable`
    
3. Run `xs buildpacks` to check if the installation was successful
    > ##### Note
    >If you have some issues with the encoding of the `env.sh` file, delete your buildpack by entering `xs delete-buildpack python_buildpack`. Switch to your cloned git repository and find the `env.sh` file, run `dos2unix /env.sh` to convert the encoding. Install the buildpack again.

[ACCORDION-END]

[ACCORDION-BEGIN [Step 5: ](Build DB service)]

###Build DB service

1. Get `mtar` file in `mta_archives` package after building the service in webide. Additional instructions for building are found on the `DKOM_PYTHON_SERVICE readme`.
2. Deploy`mtar` file using `cli` command `xs deploy file.mtar`

[ACCORDION-END]

[ACCORDION-BEGIN [Step 6: ](Build Python service)]

###Build Python service

1. Find the Web IDE project in your file system, or export it directly from Web IDE.
2. Run the command `xs push -f /DKOM_PYTHON_SERVICE/python
/manifest.yml` on the folder where you have the files.

[ACCORDION-END]

[ACCORDION-BEGIN [Step 7: ](Onboarding on Cloud Foundry Canary)]

###[Onboarding on Cloud Foundry Canary](https://docs.cloudfoundry.org/concepts/overview.html)

1. Install CF Cloud Foundry Command Line Interface CLI from: [https://docs.cloudfoundry.org/cf-cli/install-go-cli.html](https://docs.cloudfoundry.org/cf-cli/install-go-cli.html)
2. Login to CF Canary using `cli`:
    `cf login –a https://api.cf.sap.hana.ondemand.com -u <d-user>`
3. In case of facing an error because of proxy settings, set the proxy settings like this:
    ######Windows:
    >`set HTTPS_PROXY=proxy:8080`
    >`set HTTP_PROXY=proxy:8080`
    
    ######Linux:
    >`export HTTPS_PROXY=proxy:8080`
    >`export HTTP_PROXY=proxy:8080`

[ACCORDION-END]

[ACCORDION-BEGIN [Step 8: ](Export Database to Cloud Foundry (proxy settings and app))]

###[Export Database to Cloud Foundry (proxy settings and app)](https://github.wdf.sap.corp/cloudfoundry/chisel)

To deploy the service on cloud foundry, you need first to export the foreign database schema to cloud foundry. To achieve this you need to use chisel which is an HTTP client and server which acts as a TCP proxy, on cloud foundry it can be used to map TCP endpoints of your backend services to your local workstation.

1. Clone the chisel repository to your local workstation and follow the instructions on the readme file.
    `git clone https://github.wdf.sap.corp/cloudfoundry/chisel.git`
    
    > ##### Note
    >When pushing the chisel app use a globally unique name for your application, otherwise the app will not be created.

2. After connecting the tunnel with chisel, use the credentials you have to add the system to HANA Studio to start creating importing data.
    Remember to use `localhost` as hostname and `00` as instance number.

[ACCORDION-END]

[ACCORDION-BEGIN [Step 9: ](Deploy the service on cloud foundry)]

###Deploy the service on cloud foundry

1. `cf login –a https://api.cf.sap.hana.ondemand.com` 
    Organization: `DKOM2017` - to change the organization use `cf t –o <organization name> `
    Space: `milano` - to change the space use `cf t –s <space name>`
    
2. Execute this script to create the `User_Provided_Service`
    `uups` to update or `cups` to create a new service
    
    `cf cups CROSS_SCHEMA_ACCESS_SERVICE -p "{'host':'<host>','port':'<port>','user':'<user>','password':'<password>','driver':'com.sap.db.jdbc.Driver','tags':['hana'] , 'schema':'<schema>' }"`
    
    All of these `<host>, <port>, <user>, <password>, <schema>` you can get them from the previous step, since the database schema was created and populated already in cloud foundry. 
    In case an error happened because of quotations use this form:
    `cf cups CROSS_SCHEMA_ACCESS_SERVICE -p "{\"host\":\"<host>\",\"port\":\"<port>\",\"user\":\"<user>\",\"password\":\"<password>\",\"driver\":\"com.sap.db.jdbc.Driver\",\"tags\":[\"hana\"] , \"schema\":\"<schema>\" }"`
    
3. Deploy the locally built service on cloud foundry `cf deploy file.mtar`
4. The URL for the service should work like this: [https://dkom_python_service.cfapps.sap.hana.ondemand.com/?id=42&nrw=50&erw=50](https://dkom_python_service.cfapps.sap.hana.ondemand.com/?id=42&nrw=50&erw=50)

[ACCORDION-END]

[ACCORDION-BEGIN [Step 10: ](Exposing the Service using API management)]

###Exposing the Service using API management

Now that the service is available on cloud foundry, is can be exposed on API management for more security and extra features. 
For this we go to API Management: [https://apiportalapimgmtdr-a37442951.hana.ondemand.com/#/shell/homepage](https://apiportalapimgmtdr-a37442951.hana.ondemand.com/#/shell/homepage) and create a new API.

API management allows the user to add policies that secures his apps on the cloud.

**Quota Policy** specifies the number of requests allowed for the API per the interval of big time unit. 

**Spike Arrest** specifies the number of requests allowed for the API per the interval of small time unit, so the service is save of denial of service attacks and protects against performance lags and downtime.

**Run the app locally**

1. Clone the application code from git to Web IDE: [https://github.wdf.sap.corp/hana-demo-team/dkom_minimized.git](https://github.wdf.sap.corp/hana-demo-team/dkom_minimized.git)

2. Check the cloud foundry service URL in `backend/server.js -> var serviceAPI = "";`
3. Check the application backend service URL in `ui/resources/webapp/controller/app.controller.js -> Settings.API_PATH = "https://<vmware_host>:<port>";`

[ACCORDION-END]

#Part B: Develop and Deploy

[ACCORDION-BEGIN [Step 1: ](Show source tables)]

###Show source tables

1. Here you can see three tables, Parties, Ratings and Transactions. The Parties table holds detailed information about the customers of the bank. For these customers, we also put external rating data as semi-structured JSON documents into the brand new HANA document store. Finally, we have the Transactions table which contains money transfers between our parties. These transactions form a payment network, which can then be represented as a graph in our new in-memory graph engine.

    ![Source tables](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step1_1.png)
    
2. All we need to do, to use the graph engine, is to create a graph workspace in which we define the parties as nodes and the transactions as edges between them.

[ACCORDION-END]

[ACCORDION-BEGIN [Step 2: ](Replace Rating function with Graph rating)]

###Replace Rating function with Graph rating

1. To calculate this, we have a rating stored procedure, which uses plain SQL.

    ![Rating procedure](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step2_1.png)
    
2. These kinds of graph queries require lots of self-joins and unions, which are tough to write and maintain.
    
    ![Unions and joins](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step2_2.png)
    
3. Now let’s see how we can simplify the SQL statement using the new graph engine.
    
    ![Simplified SQL](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step2_3.png)
    
4. First, we’ll get rid of the old SQL.
    
    ![Remove old SQL](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step2_4.png)
    
5. And replace it with the new one.
    
    ![New SQL](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step2_5.png)
    
6. Using the new graph engine does not only look prettier, it also gives you much more expressiveness, flexibility and performance.
    
    ![Graph Engine](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step2_6.png)

[ACCORDION-END]

[ACCORDION-BEGIN [Step 3: ](Show Service Wrapper in Python)]

###Show Service Wrapper in Python

Now, we’re done with the database artifacts. Let’s move on and create a consumable service on top. We talked about using any programming language. We can leverage Python or any other language in SAP HANA

1. With SAP HANA XS Advanced, we can create application containers in any language. All we need to do is tell XS Advanced how to compile and run our container. As we may plan to integrate machine learning algorithms in the future, we chose Python and specified it in this manifest file. The python buildpack runs your container by executing a file called `server.py`

    ![Python file](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step3_1.png)
    
2. It contains a simple Python HTTP service. For now, we implemented the HTTP GET method to just call the Rating stored procedure and return the result.
    
    ![HTTP GET method](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step3_2.png)

[ACCORDION-END]

[ACCORDION-BEGIN [Step 4: ](Deploy locally)]

###Deploy locally

1. To run the Python service, we need to deploy it. Deployment is enabled by the `manifest.yml` file.

    ![Manifest local](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step4_1.png)
    
2. Type the following command via command line to execute XS push
    `xs push -f manifest.yml`
   
    ![Local deployment](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step4_2.png)
    
3. The Python service is now starting up.
    ![Local deployment](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step4_3.png)
    
4. This deployment can be viewed on the browser. You can see the results for the stored procedure that was called.

    ![Browser Local](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step4_4.png)

[ACCORDION-END]

[ACCORDION-BEGIN [Step 5: ](Deploy on Cloud)]

###Deploy on Cloud

1. This same service can also be deployed in the cloud. The `manifest-cf.yml` file enables the deployment. The only thing we have to do is to replace `xs` by `cf` in the command line, and `manifest.yml` by `manifest-cf.yml`
    
    ![Manifest cloud](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step5_1.png)

2. One simple command creates the application container, deploys the code, and starts our service in the Cloud.
    
    ![Cloud deployment](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step5_2.png)
    
3. Once deployed, which just take a few moments, we can reach the service via this URL.

    ![Cloud deployment](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step5_3.png)
    
4. This deployment can be viewed on the browser. You can see the results for the stored procedure that was called.
    
    ![Browser Cloud](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step5_4.png)

[ACCORDION-END]

[ACCORDION-BEGIN [Step 6: ](Create API)]

###Create API

1. We have developed the database artifacts along with the consumable service, and have deployed all of it in the Cloud. The final step is to use the service in a Fiori frontend. Go to the SAP API Management console at : [https://apiportalapimgmtdr-a37442951.hana.ondemand.com/#/shell/homepage](https://apiportalapimgmtdr-a37442951.hana.ondemand.com/#/shell/homepage) and select the **Create** button to create an API. 

    ![API Management](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step6_1.png)
    
2. The modal will pop up. Enter the details to create your API. Add the URL of your cloud service as the API URL. 
    
    ![Create API](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step6_2.png)
    
3. Your API will be created, and you can view its properties. Go to the options on the top-right corner and go to **Policies**
    
    ![Policies](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step6_3.png)
    
4. Here you can view the policies for your API. Go to **Pre Flow** and then scroll down on the list of policy types on the right, to select and create a new **Spike Arrest** policy.

    ![Create policy](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step6_4.png)
    
    Configure your new policy and then click **Add**
    
    ![Configure policy](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step6_4b.png)
    
5. You can make your desired chances to your policy and then select **Update** when done.

    ![Edit policy](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step6_5.png)
    
6. Back on your API screen, you can **Save and Deploy** your API.

    ![Save and deploy API](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step6_6.png)
    
7. You can see the deployed status of your API with its properties. 

    ![Save and deploy API](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step6_7.png)
    
    You can see your deployment URL in the **Target endpoint** tab
    
    ![Save and deploy API](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step6_7b.png)
    
8. Open the **API Proxy URL** in a browser tab and you can see the displayed results:
    
     ![Save and deploy API](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step6_8.png)
     
     ![Save and deploy API](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step6_8b.png)
     
     ![Save and deploy API](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step6_8c.png)

[ACCORDION-END]

[ACCORDION-BEGIN [Step 7: ](Add API URL to app)]

###Add API URL to app

1. We have a sample lightweight application in SAP HANA with node.js. 

    ![Browser Cloud](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step7_1.png)
    
    We can see the source code here. 
    
    ![Browser Cloud](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step7_1b.png)

2. We now add the URL of the Cloud Service to the application via this line of code.

    ![Browser Cloud](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step7_2.png)
    
    Go back to the API properties and copy the API proxy URL
    
    ![Browser Cloud](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step7_2b.png)
    
    And paste it back in that line of code, and then run the application with the **Run** button on the top.
    
    ![Browser Cloud](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step7_2c.png)
    
    You'll be able to see that your application is now running.
    
    ![Browser Cloud](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step7_2d.png)
    
3. Now go to the `ui` folder and under `resources --> webapp`, you will find `index.html`. Run this file.

    ![Browser Cloud](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step7_3.png)
    
4. You will see your Fiori style user interface where you can select your particular account number:
 
    ![Browser Cloud](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step7_4.png)
    
    You can now see your application deployed via the API URL you just entered.
    
    ![Browser Cloud](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step7_4b.png)
    
    ![Browser Cloud](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step7_4c.png)

[ACCORDION-END]
---

### A few notes
- To conclude, with SAP HANA, express edition, we were able to develop a service on the go, and also directly deploy the service in the cloud. For the data, we required a document store and a graph engine. Here we were able to leverage the new SAP HANA NoSQL capabilities. On top of that, we had the freedom to use the programming language of our choice and decided to use Python for our service wrapper. Any data, any programming language, any deployment, this is SAP HANA 2.0.


## Next Steps
 - Go to Step 4 on the [SAP HANA, Express Edition Developer Page](https://www.sap.com/developer/topics/sap-hana-express.tutorials.html)