## Source Identification

**Data Sources**   
+ Social Media
+ Call centre log files
+ SMS message
+ Website Forms

**Formats and Frequency**
+ Social media
  + Format: Semi-Structured data along with metadata(e.g user_id, timestamp, message)
  + Frequency: Real Time
  + Assumption: Our DM has automated bots that reply to users and get their details as well as their complaint, as soon as we get it, a pipeline is triggered to process this data and send it to the next phase in our pipeline
+ Call centre log files
  + Format: Semi structured data along with metadata(e.g call_id, timestamp, call_duration, transcript)
  + Frequency: Batch. The logs will be put together in log files and picked up (hourly)
+ SMS message
  + Format: Semi structured (metadata + sms text)
  + Frequency: Real Time. As soon as we recieve the message, it is processed immediately
+ Website Forms
  + Format: Semi structured format (customer details, complaint_categorie, complaint_text and some optional attachments like screenshots).
  + Frequency: Real Time - Each form submission triggers ingestion  
  + Assumption: Since it is a webiste form, we hav control over how the customers fill the form so we can actually make life easier for ourself by adding a complaint category to the form and this way, the complaint are categorized from source.

## Ingestion strategy

+ Social Media - Every message is captured in real time. and enters a pipeline that will classify it. Social media data is tricky, complaints can be logged via DM, under comments or even posted publicly.
+ Call Center Logs - Ingested in batch mode - Hourly
+ SMS message – Every complaint should be handled in real time so it will trigger a pipeline to ingest the data
+ Website forms – Every form submitted should trigger some kind of pipeline to ingest the data

## Processing and Transformation

+ Every complaint should get an identifier
+ We will have an internal timezone which all timezone will be converted into - (why? it is possible for two states in the same country to have different timezone so we need to be explicit enough to avoid timezone issues)
+ Complaint classfification: We will integrate AI in our App to classify complaints gotten from sms, social media, and call center log files. Also some user might find it humorous to stress us so they will send messages are not complaints. Tagging those too will be helpful. 
+ Complaint Tag: Almost all complaints we get will be pending i.e unsolved. Tagging them will help us track this and allocate adequate staff to handle all complaints as fast as possible

## Storage Strategy

**Data Lake**
+ The data lake will store the raw data. There are three reasons for this
  + Storage is cheap
  + Data processing could go wrong for some reason and we need to make sure to have a source where the data is guaranteed to be available. We cannot trust the actual source systems to keep this data for so long
  + We may have another usecase for our data in the future that we are not considering yet, having the data as it came from the source will help in this situation
+ The data lake can be partitioned by source and by timestamp i.e the data from sms will be stored in sms folder and in a file with its timestamp as its name

**Data Warehouse**
+ Clean and structured complaint will be sored here
+ Only relevant complaints are stored here 

## Data Serving

+ Querying: Data analysts and Business analysts can use SQL to query and aggregate the data
+ Dashboards: Our warehouse can also feed visualization dashboards to track complaint volume and how long we are taking to respond and also potential trends in complaints
+ APIs - We can create an API that queries our warehouse, it will be linked to a frontend and our agents can resolve pending complaints

## Orchestration and monitoring

- **How often will this pipeline run?**
  + Streaming source i.e. Social Media SMS and Website Forms
    + + When the data gets to the lake, those from the streaming source(i.e Social Media, SMS, Website form) should be processed in realtime
    + i.e They are transfomed, and classified and pushed to warehouse immediately
    + We cant have pressing complaints solved too long after complaint has been made
  + Batch Source i.e Call center log files
    + Logs come in hourly and so processing should be at least hourly
    + Here the Complaints show up in dashboards within 1 hour but this is okay because calls involves conversation with the customer service and it is very likely to have been solved

- **How will failures be detected or notified?**
  + Since the pipeline is code, then the errors should be raised via code
  + And when this error is detected, the admin should be notified via mail that a pipeline has broken

- **Monitoring**
  + We need to set up systems to monitor pipeline general health and usage of memory and ram of the server running the pipeline
  + We can hav metrics on spikes in volume i.e if a very large volume is processed all at once

## DataOps

+ Pipeline runs on cloud (So we can scale up and down as much as we like)
+ We can set up CI/CD