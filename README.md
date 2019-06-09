# Salesforce-sync-appconnect
app connect sample to sync data between external system and sales force accounts using IBM app connect 

## About The Project
This project is created to synchronize data from an external system to Salesforce (one-way synchronization).
Here are the challeges that this project address:
* Create new accounts in Salesforce for new customers in the external system, in an intial sync step. Mapping the Customer name and id fields rom external system to saleforce.
* Check for any subsequent customers created or updated on the external system after the intial sync using the If-Modified-Since header which is implemented by the external system api.


### Built With
There were serveral options to use to implement this project using any programing, scripting language or using iPaaS. 

Here I decided to use the iPass cloud offering as it provides the agility, flexabiliy needed and allow you easily to sync and integrate between diffrenet systems. I choosed to use IBM Cloud integration offering to develop this projcet as I have pirior experince working with IBM cloud and I didn't have much time to develop and using iPass IBM cloud offering helped me to speed up the devlopments to couple of hours. I used IBM app connect cloud service which allow you to connect your different applications and make your business more efficient, Set up automation flows to direct how events in one application trigger actions in another, and map the information you want to share between them. 

* [APP Connect](https://cloud.ibm.com/catalog/services/app-connect)
* [IBM Cloud](https://cloud.ibm.com/)
* [Salesforce](https://eu26.salesforce.com/001/o)
* [IBM Cloudant](https://cloud.ibm.com/catalog/services/cloudant)


### Setup and Prerequisites

As the devlopment was done using cloud palttforms. The following accounts and ervices are needed:

* Creation of an account on [IBM Cloud](https://cloud.ibm.com/) 
  * Provisioning of [APP Connect](https://cloud.ibm.com/catalog/services/app-connect) service to develop our event driven flow.
  * Provisioning of [IBM Cloudant](https://cloud.ibm.com/catalog/services/cloudant) service to be used as an external  configuration service. The config file attached in the repository (config.json) to be stored in the service and api keys to be gerated to be used in the app connect flow. 

* Creation of [Salesforce] (https://developer.salesforce.com/signup) devloper account to store the data retrived from the external system.

## Implementation


I devloped an Event-driven flow on App connect that retrieve new customers from external system and sync these accounts with salesforce created dev space. in the follwoing points I break down the implementation details. Aslo I higligh some limitaions faced during implementation.

### Implementation limitaions

  using free devloper accounts for IBM and salesforce, I faced limitiations on processing and storing large amount of data. I had to used If-Modified-Since head for both intial sync and subsequent sync calls to limit the retrun of data as calling the external system service without limit returned more than 60,000 customers. which was impossible to process and store using cloud free accounts due to account plan limittion. The implementation logic is the same only on smaller amount of data. The only impact is not syncing The customers with CustomerId 1 and 2 which only displayed if no If-Modified-Since header is specified.     

### Implementation Steps

*The flow startes with scheduler node which is configure to run every 30 miunte to trigger the flow, Then it connects to IBM cloudant service which is a JSON document based DB accessible via Rest Apis that I used as an external configraution service to lookup and save manly two paramters lastruntime ( which the last time the external system was called and intialize with null) and intialruntime ( which is the date time usedd for the intial sync between external system and salesforce system). The flow checks for the value of the lastruntime config param if it's not empty (whcih means that this is an update for the customers not an intial sync) it continue with the flow setting date time as the lastruntime to retrieve the customers created/updated since the last run from the external system, else it set's it with intialruntime param to start an initial sync.

The config file is inlcuded in the respiratory (config.json)

![Event-Driven-Flow-1](https://raw.github.com/mismaeel/salesforce-sync-appconnect/master/resources/screenshots/flow-part1.PNG "Flow-1") 

* Then Http Get request is sent to look up the new/updated customers on the external system. The response body is parsed to and hte CustomerId and Customer Name is used for each customer on the response using  for each flow step configured to use salesforce connecter  for account create update API, which connect to the created saleforce devloper account. checking and logging the status of the response adn then continue the rest ofthe flow.

![Event-Driven-Flow-2](https://raw.github.com/mismaeel/salesforce-sync-appconnect/master/resources/screenshots/flow-part2.PNG "Flow-2") 

* The last part of the flow is updating the cloudant db external configuration service with the lastruntime to be used with subsequent calls.

![Event-Driven-Flow-3](https://raw.github.com/mismaeel/salesforce-sync-appconnect/master/resources/screenshots/flow-part3.PNG "Flow-3") 


## Using the flow 

The sample App Connect flow in resources/appconnect does a look up of a Customers in external system and sync it with  salesforce accounts using the custom created CustomerId field and mapping customer Name. To use this flow, complete the following steps:

*Import the flow into App Connect.

*In Operations, Edit the flow and change the Salesforce application where the customers are created/updated to use your own Salesforce account. change the IBM cloudant retrieve and update services to use your own created service.

*Start the API


## Testing the flow

I configured the flow to be trigerred every 30 minutes after the intial sync which I limited to start from (Sun, 02 Jun 2019 11:18:00 GMT) to be able to prcoees and store data normaly using free cloud accounts. I started the flow.

* The customers were created on my saleforce account for the intial sync 609 account were created ( all accounts created since  02 Jun which was used as a thershold) and every 30 minutes customer updated were synced. the service is running and the total records now since the intial sync is 630 record and will keep updating till hitting some limit on ibm cloud pricessing or salesforce account storing

![customers-salesforce](https://raw.github.com/mismaeel/salesforce-sync-appconnect/master/resources/screenshots/customers-salesforce.PNG "customers-salesforce") 

![IBM-cloud-logs](https://raw.github.com/mismaeel/salesforce-sync-appconnect/master/resources/screenshots/IBM-cloud-logs.PNG "IBM-cloud-logs") 

![salesforce-updated-record](https://raw.github.com/mismaeel/salesforce-sync-appconnect/master/resources/screenshots/salesforce-updated-record.PNG "salesforce-updated-record") 

* The IBM cloudant external configuration service record updated during processing

![cloudant-screenshot](https://raw.github.com/mismaeel/salesforce-sync-appconnect/master/resources/screenshots/cloudant-screenshot.PNG "cloudant-screenshot") 

* I will provide users and creditinals for the created IBM and saleforce account to better check the implementation.

