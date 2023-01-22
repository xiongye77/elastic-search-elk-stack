# elastic-search-elk-stack

![image](https://user-images.githubusercontent.com/36766101/213895534-1ebcee64-7e90-4cdd-9378-f03166554ada.png)

Logstash:
Logstash is an open-source, server-side data processing pipeline that can simultaneously ingest data from many sources, analyze the data, filter, transform and enrich the data, and then forward it to a downstream system.

Data flows through a Logstash pipeline in three stages: the input stage, the filter stage, and the output stage.

Input stage: data is ingested into Logstash from a source. Logstash doesn’t access and collect the data, it uses input plugins to ingest the data from various sources.

Filter stage: Once data is ingested, one or more filter plugins take care of the processing part in the filter stage. In this stage, necessary data elements are extracted from the input stream.

Output stage: processed data is sent to a receiver and are output. Output plugins are available for many different endpoints, including those for Elasticsearch, HTTP, e-mail, S3 file, PagerDuty alert, or Syslog to name just a few.

Logstash’s processed data is saved in a high-performance, searchable storage engine, and easily viewable from a user interface tier.



# Send Data to Elasticsearch with Security
![image](https://user-images.githubusercontent.com/36766101/213895991-bdef7a88-e1a7-448e-a42a-c2fc2e880fd1.png)

Copy certificate and give enough permission

![image](https://user-images.githubusercontent.com/36766101/213896008-e91f068b-77d5-4acc-ba84-4f240411a980.png)

elastic1@elastic1:/usr/share/logstash$ bin/logstash -f logstash-filter.conf
