# Amazon-Managed-ELK-stack-configuration

here we are going to stream our cloudwatchlogs to amazon manged Elastic Search (ELK Stack)


if you don't have any cloudwatch logs, enable VPC flow logs or push any server logs with cloudwatch agent.
https://github.com/itsmeshabeeb/amazoncloudwatchagent


we are going to create a Elasticsearch / Opensearch Cluster


1. open the Opensearch Service and create a Domain.
2. give domain name. if you have custom Domain name in Route53, enable the Custom endpoint and give the Subdomain.
3. deplyment type we can select Development and testing
4. version, we will get opensearch and elasticsearch select latest from one of them.
5. select nodes as 1 AZ and t3 small/ medium.
6. in network section, select public
7. from Fine-grained access contro, Enable fine-grained access control and select Create master user. Create one user id and apssword to access the webui of Kibana
8. in Access policy section select "Only use fine-grained access control" 
9. now create the clluster

now we can go to cloudwatchlogs to stream logs to this cluster:

1. select the log group you want to stream and in actions use > Create Amazon OpenSearch Service subscription filter
2. select the ELK cluster.
(here Cloudwatchlogs uses lambda to deliver logs to elasticsearch. so we must specify an IAM role that grant lambda permission to push datas to elastic cluster.)
3. open IAM service, and create a role (lambda-elasticsearch-execution-role) for lambda and attach below permissions
    - AWSLambdaBasicExecutionRole
    - AmazonOpenSearchServiceFullAccess
    (this is for learning purpose. full access not recommended in in production)
4. select this role in subscription filter.
5. select log format as Amazon Cloudtrail and give a subscription name
6. now start streaming.
7. open our kibana domain endpoint from openseach service and login with our credentials.
8. in manage section, select index patterns and create index pattern.
9. here you will see the index pattern available in easticsearch based on the loggroup.
10. select your index pattern and for primary timefield select @timestamp then create index pattern.
11. now go to discover change the indexes to get the logs.




(here, Lambda will create a nodejs function in backend to complete the stream. by default lambda will allow you to stream one log group to elasticsearch. if you want to stream multiple log groups, you have to do some changes in lambda)

// index name format: cwl-YYYY.MM.DD
     var indexName = [
         'cwl-' + timestamp.getUTCFullYear(),           // year
         ('0' + (timestamp.getUTCMonth() + 1)).slice(-2),  // month
         ('0' + timestamp.getUTCDate()).slice(-2)          // day
     ].join('.');


     you can find the above code in lambda function (default code line 62) replace this with :


// index name format: cwl-YYYY.MM.DD
     var appName =payload.logGroup.toLowerCase();
     var indexName = '';
     var indexName = [
         'cwl-'+ appName + '-' + timestamp.getUTCFullYear(), // year
         ('0' + (timestamp.getUTCMonth() + 1)).slice(-2), // month
         ('0' + timestamp.getUTCDate()).slice(-2), // day
     ].join('.');

deply the function and test by streaming multipule logs