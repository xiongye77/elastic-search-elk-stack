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

Provisioning ES Domain
Now we gonna provision an ES Domain with Fine-Grained Access Control enabled. It provides 02 types of authorization and authentication methods.

Method 01 — A built-in user database, which makes it easy to configure usernames and passwords inside of Elasticsearch.

Method 02 — AWS Identity and Access Management (IAM) integration, which lets you map IAM principles to permissions.

STEP 01: Provision the ES Cluster
Create a file as es_domain.json and paste the below configuration.


Update the following paramters.

ES_DOMAIN : Your ES domain name
ES_INSTANCE_TYPE : ES domain cluster node size (Default r5.large.elasticsearch)
NO_OF_INSTANCES : Number of instances for the cluster (integer)
VOLUME_SIZE_PER_INSTANCE : Volumn per instance (integer; the total volume size is NO_OF_INSTANCES*VOLUME_SIZE_PER_INSTANCE)
AWS_REGION : ES Domina cluster region
ACCOUNT_ID : AWS account ID
MASTER_USERNAME : Master username
MASTER_PASSWORD : Master password
Once the file is done, execute the below command to provision the cluster

aws es create-elasticsearch-domain --cli-input-json  file://es_domain.json
It will take 5–10 mins to complete the provision.

STEP 02: Configure Fluent-Bit in EKS
Before starting, create an OIDC identity provider. Execute the command to create.

eksctl utils associate-iam-oidc-provider --cluster EKS_CLUSTER_NAME --approve
Then create IAM role and policy. Create a file named fluent-bit-policy.json and paste the below policy definition.


Update the following parameters.

ES_DOMAIN : Your ES domain name
AWS_REGION : ES Domina cluster region
ACCOUNT_ID : AWS account ID
Then create the policy.

aws iam create-policy --policy-name fluent-bit-policy --policy-document file://fluent-bit-policy.json
Finally, create an IAM role for fluent-bit. We will deploy fluent-bit in a namespace as fluentbit (You can have your own namespace here).

kubectl create namespace fluentbit
eksctl create iamserviceaccount \
    --name fluent-bit \
    --namespace fluentbit \
    --cluster EKS_CLUSTER_NAME \
    --attach-policy-arn "arn:aws:iam::ACCOUNT_ID:policy/fluent-bit-policy" \
    --approve \
    --override-existing-serviceaccounts
Finally, let’s configure backend roles for ES Cluster.

Backend roles, also called external identities, offer another way of mapping roles to users. Rather than mapping the same role to dozens of different users, you can map the role to a single backend role, and then make sure that all users have that backend role. Backend roles can be IAM roles or arbitrary strings.

We will add the Fluent Bit ARN as a backend role to the all_access role using the Elasticsearch API

First, retrieve the Fluent Bit Role ARN

export FLUENTBIT_ROLE=$(eksctl get iamserviceaccount --cluster EKS_CLUSTER_NAME --namespace logging -o json | jq '.[].status.roleARN' -r)
Next, get the Elasticsearch Endpoint

export ES_ENDPOINT=$(aws es describe-elasticsearch-domain --domain-name ES_DOMAIN_NAME --output text --query "DomainStatus.Endpoint")
Finally, update the Elasticsearch internal database. For this task, you can use postman.

curl -sS -u "${ES_DOMAIN_USER}:${ES_DOMAIN_PASSWORD}" -X PATCH \
https://${ES_ENDPOINT}/_opendistro/_security/api/rolesmapping/all_access?pretty \
    -H 'Content-Type: application/json' \
    -d'
[
  {
    "op": "add", "path": "/backend_roles", "value": ["'${FLUENTBIT_ROLE}'"]
  }
]'
Output:

{
  "status" : "OK",
  "message" : "'all_access' updated."
}
STEP 03: Deploy Fluent-bit
Finally, let’s deploy fluent-bit in our EKS cluster. Copy the below configuration file and save as fluentbit.yaml. (P.S: Find CHANGE_ME and update with your configurations)


Host : ES Domain host
AWS_Region : ES Domain region
Then apply the file kubectl apply -f fluentbit.yaml

Wait for pods to start.

You can see that we are using a DaemonSet instead of Deployment. (one pod per worker node)

STEP 04: Access Kibana Dashboard
You can access Kibana Dashboard with the URL https://ES_ENDPOINT/_plugin/kibana/. Use master username and master password configured to login.

Once you log into Explore on your own select Connect to your Elasticsearch index on the welcome page. Then click on create Index pattern. Add *fluent-bit* as the index pattern and hit Next step.

In the next step select @timestamp as the Time filter field name and close the Configuration window by clicking on Create index pattern.

Finally, go to Discover from the left side panel and monitor logs.


