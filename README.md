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
[ACCORDION-BEGIN [Step 1: ](Import Code)]

###Import Code

1. Clone the service and UI code from `git` to Web IDE:

    - [Python Service](https://github.wdf.sap.corp/hana-demo-team/DKOM_PYTHON_SERVICE.git)
    - [Minimized Source Code](https://github.wdf.sap.corp/hana-demo-team/dkom_minimized.git)

[ACCORDION-END]

[ACCORDION-BEGIN [Step 2: ](Database)]

###Database

1. Execute the queries present on data.sql and insert.sql (DKOM_PYTHON_SERVICE project) in SAP HANA Studio or in Web IDE.

[ACCORDION-END]

[ACCORDION-BEGIN [Step 3: ](Create user provided service)]

###Create user provided service

The user provided service gets access to data foreign to the XSA application container. It needs to assign a database user that has authorization with grant option to the foreign schema. The `data.sql` script from the last action already assigns these authorizations to `XSA_ADMIN`.
    
1. Execute the script to create the user provided service (if it is already created, check by typing `xs s` and look for `CROSS_SCHEMA_ACCESS_SERVICE`
    
    `cups` to create or `uups` to update a new service
    
    `xs cups CROSS_SCHEMA_ACCESS_SERVICE -p`
    `'{"host":"<host>","port":"<port>","user":"XSA_ADMIN","password":"<password>","driver":"com.sap.db.jdbc.Driver","tags":["hana"] , "schema":"<schema>" }'`
    
    `<host>`: `host name or machine IP address`
    
    `<port>`: `3<instance number><13 multi containers or 15 single containers>`

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

---

### A few notes
- 

## Next Steps
 - Go to Step 4 on the [SAP HANA, Express Edition Developer Page](https://www.sap.com/developer/topics/sap-hana-express.tutorials.html)