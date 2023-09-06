# DynamoDBConnector

MSK Connect with DynamoDB as a source connector to stream DDB change events from various DDB tables to some MSK topics.


### Steps:

Please find the below steps for creating MSK DynamoDB Source Connector.

1. If your connector require internet access, create MSK Cluster in Private subnets with NAT Gateway attached. https://docs.aws.amazon.com/msk/latest/developerguide/msk-connect-internet-access.html 

2. Create sample DynamoDB table and enable stream on table by following https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Streams.html#Streams.Enabling 

3. Then create Custom plugin by following the below steps:
   + Download the DDB.ZIP file attached to this repo.
   + Then upload the ZIP file to any S3 Bucket.
   + Now create the custom plugin by selecting that ZIP file from your S3 bucket by following https://docs.aws.amazon.com/msk/latest/developerguide/mkc-create-plugin.html

4. After you create the Plugin, start creating connector by using the below configuration:

Configuration For MSK NoAuth cluster:
```
connector.class=dynamok.source.DynamoDbSourceConnector
tasks.max=1
bootstrap.servers=b-2.test.kafka.us-east-1.amazonaws.com:9092,b-1.test.kafka.us-east-1.amazonaws.com:9092
tables.prefix=DynamoDB-table-names
offset.flush.interval.ms=10000
key.converter.schemas.enable=false
topic.format=your-MSK-Topic-name
value.converter.schemas.enable=false
value.converter=org.apache.kafka.connect.storage.StringConverter
region=us-east-1
key.converter=org.apache.kafka.connect.storage.StringConverter
```

Configuration for MSK IAM cluster:

```
connector.class=dynamok.source.DynamoDbSourceConnector
tasks.max=1
bootstrap.servers=b-2.test.kafka.us-east-1.amazonaws.com:9098,b-1.test.kafka.us-east-1.amazonaws.com:9098
tables.prefix=DynamoDB-table-names
offset.flush.interval.ms=10000
key.converter.schemas.enable=false
topic.format=your-MSK-Topic-name
value.converter.schemas.enable=false
value.converter=org.apache.kafka.connect.storage.StringConverter
region=us-east-1
key.converter=org.apache.kafka.connect.storage.StringConverter
sasl.mechanism=AWS_MSK_IAM
security.protocol=SASL_SSL
sasl.jaas.config=software.amazon.msk.auth.iam.IAMLoginModule required
sasl.client.callback.handler.class=software.amazon.msk.auth.iam.IAMClientCallbackHandler
```

If you would like to add any configuration, please refer to the documentation - https://github.com/shikhar/kafka-connect-dynamodb#configuration-options-1 

5. Create your connector’s IAM role and make sure it should have proper MSK Cluster and DynamoDB permissions. For testing purposes, I added below AmazonDynamoDBFullAccess permission.

+ AmazonDynamoDBFullAccess
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "kafka-cluster:Connect",
                "kafka-cluster:DescribeCluster",
                "kafka-cluster:ReadData",
                "kafka-cluster:DescribeTopic",
                "kafka-cluster:WriteData",
                "kafka-cluster:CreateTopic",
                "kafka-cluster:AlterGroup",
                "kafka-cluster:DescribeGroup",
                "kafka-cluster:*",
                "kafka:*"
            ],
            "Resource": [
                "arn:aws:kafka:us-east-1:<account-id>:cluster/<cluster-name>/3b5b86fe-fe9c-4a43-939f-0993d2f761cb-2",
                "arn:aws:kafka:us-east-1:<account-id>:topic/<cluster-name>/*",
                "arn:aws:kafka:us-east-1:<account-id>:group/<cluster-name>/*",
            ]
        }
    ]
}
```
  

6. Finally, consume the data from your MSK Topic and see the changes of your DDB table written to the topic.
