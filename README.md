---
title: How to build an application in SAP HANA, express edition and deploy it locally and on the cloud
description: This tutorial will show you how to develop and deploy anywhere with any kind of data, in any programming language, with SAP HANA, express edition.
tags: [  tutorial>intermediate, products>sap-hana\,-express-edition ]
---

## Prerequisites  
1. [Decide an installation option for SAP HANA, express edition](https://www.sap.com/developer/how-tos/2016/09/hxe-ua-version.html)
2. [Binary install (for custom install on Linux machines)](https://www.sap.com/developer/tutorials/hxe-ua-installing-binary.html) 

   **OR**
   
   [Virtual Machine image install (for VMs on Windows/Mac OS X, or prebuilt install on Linux machines)](https://www.sap.com/developer/tutorials/hxe-ua-installing-vm-image.html)
   
3. For each install option chosen, please proceed to the respective **Next Steps** as outlined in each tutorial.
4. Make sure Web IDE is up and running, ready with a workspace environment.


## Next Steps
 - Go to the Tutorials section on the [SAP HANA, Express Edition Developer Page](https://www.sap.com/developer/topics/sap-hana-express.tutorials.html)

## Details

### You will learn
How to build a sample Python application with SAP HANA, express edition, and deploy it to the cloud. This tutorial uses an environment that has SAP HANA, express edition pre-installed, with a preloaded sample dataset. The example here shows how to develop a service that quickly assesses the trustworthiness or credit rating of the customers of a bank. Everything runs on a single, mobile device: the HANA database, the HANA XS Application Server, as well as Web IDE.

### Time to Compete
**30 min**

---
***Please note: Work in Progress***

[ACCORDION-BEGIN [Step 1: ](Import source code)]

###Import Source Code

First, we set up the environment by importing the Python service and the UI code. We do this by cloning the code from the given `git` links below, into Web IDE.

1. Right click on your **Workspace** folder and go to `Git` **--> Clone Repository**

    ![Git](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step1-1.png)
    
2. A popup modal appears, asking for details of the repository to be cloned. Enter the following URL to clone the `git` for the Python service: 

    [https://github.wdf.sap.corp/hana-demo-team/DKOM_PYTHON_SERVICE.git](https://github.wdf.sap.corp/hana-demo-team/DKOM_PYTHON_SERVICE.git)
    
    Select **OK**. 
    
    ![Clone](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step1-2.png)
    
    Wait for the repository to be cloned. Once cloned, it should appear on your workspace. 
    
    > ##### Note
    >For additional guidance on cloning `git` repositories, please refer to this tutorial: 
    [SAP HANA XS Advanced, Connecting to the WebIDE and cloning a Git Repository to begin development](https://www.sap.com/developer/tutorials/xsa-connecting-webide.html)
    
3. Now similarly, clone the `git` code for the minimized UI source code as well:
    
    [https://github.wdf.sap.corp/hana-demo-team/dkom_minimized.git](https://github.wdf.sap.corp/hana-demo-team/dkom_minimized.git)
    
    You should have both your repositories cloned in your workspace like this:
    
    ![Cloned workspace](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step1-3b.png)

[ACCORDION-END]

[ACCORDION-BEGIN [Step 2: ](Database Setup)]

###Database Setup

1. Go to the **Database Configuration** icon on the left panel and click on the **"+"** sign to create a new database.

    ![Cloned workspace](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step2-1.png)
    
2. A modal to add a new database pops up. Enter the following configuration details: 
 - Database type: SAP HANA Database (Multitenant)
 - Host: `IP address of your machine where HXE is running`
 - Instance number: `90`
 - Database: System database
 - User: `XSA_DEV`
 - Password: `your HANA master password` 

    Select **OK**
   
    ![Cloned workspace](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step2-2.png)
    
3. Your database will be created, which looks like this:

    ![Cloned workspace](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step2-3.png)
    
[ACCORDION-END]

[ACCORDION-BEGIN [Step 3: ](Show source tables)]

###Show source tables

1. Here you can see three tables, Parties, Ratings and Transactions. The Parties table holds detailed information about the customers of the bank. For these customers, we also put external rating data as semi-structured JSON documents into the brand new HANA document store. Finally, we have the Transactions table which contains money transfers between our parties. These transactions form a payment network, which can then be represented as a graph in our new in-memory graph engine.

    ![Source tables](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step1_1.png)
    
2. All we need to do, to use the graph engine, is to create a graph workspace in which we define the parties as nodes and the transactions as edges between them.

[ACCORDION-END]

[ACCORDION-BEGIN [Step 4: ](Replace Rating function with Graph rating)]

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

[ACCORDION-BEGIN [Step 5: ](Show Service Wrapper in Python)]

###Show Service Wrapper in Python

Now, we’re done with the database artifacts. Let’s move on and create a consumable service on top. We talked about using any programming language. We can leverage Python or any other language in SAP HANA

1. With SAP HANA XS Advanced, we can create application containers in any language. All we need to do is tell XS Advanced how to compile and run our container. As we may plan to integrate machine learning algorithms in the future, we chose Python and specified it in this manifest file. The python buildpack runs your container by executing a file called `server.py`

    ![Python file](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step3_1.png)
    
2. It contains a simple Python HTTP service. For now, we implemented the HTTP GET method to just call the Rating stored procedure and return the result.
    
    ![HTTP GET method](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step3_2.png)

[ACCORDION-END]

[ACCORDION-BEGIN [Step 6: ](Deploy locally)]

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

[ACCORDION-BEGIN [Step 7: ](Deploy on Cloud)]

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

[ACCORDION-BEGIN [Step 8: ](Create API)]

###Create API

1. We have developed the database artifacts along with the consumable service, and have deployed all of it in the Cloud. The final step is to use the service in a Fiori frontend. Go to the SAP API Management console at : [https://apiportalapimgmtdr-a37442951.hana.ondemand.com/#/shell/homepage](https://apiportalapimgmtdr-a37442951.hana.ondemand.com/#/shell/homepage) and select the **Create** button to create an API. 

    API management allows the user to add policies that secures his apps on the cloud.
    
    **Quota Policy** specifies the number of requests allowed for the API per the interval of big time unit. 
    
    ![API Management](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step6_1.png)
    
2. The modal will pop up. Enter the details to create your API. Add the URL of your cloud service as the API URL. 
    
    ![Create API](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step6_2.png)
    
3. Your API will be created, and you can view its properties. Go to the options on the top-right corner and go to **Policies**
    
    ![Policies](https://github.com/AnnieSuantak/build-in-hxe-deploy-to-cloud/blob/master/images/Step6_3.png)
    
4. Here you can view the policies for your API. Go to **Pre Flow** and then scroll down on the list of policy types on the right, to select and create a new **Spike Arrest** policy.

    **Spike Arrest** specifies the number of requests allowed for the API per the interval of small time unit, so the service is save of denial of service attacks and protects against performance lags and downtime.

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

[ACCORDION-BEGIN [Step 9: ](Add API URL to app)]

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

[ACCORDION-END]
---

### A few notes
- To conclude, with SAP HANA, express edition, we were able to develop a service on the go, and also directly deploy the service in the cloud. For the data, we required a document store and a graph engine. Here we were able to leverage the new SAP HANA NoSQL capabilities. On top of that, we had the freedom to use the programming language of our choice and decided to use Python for our service wrapper. Any data, any programming language, any deployment, this is SAP HANA 2.0.


## Next Steps
 - Go to Step 4 on the [SAP HANA, Express Edition Developer Page](https://www.sap.com/developer/topics/sap-hana-express.tutorials.html)