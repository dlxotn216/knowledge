
## Docker로 MongoDB 구성 
참고 (http://junil-hwang.com/blog/docker-mongodb/)  
docker run --name mongo-test -p 27017:27017 -e MONGO_INITDB_ROOT_USERNAME=root -e MONGO_INITDB_ROOT_PASSWORD=mongodb -d mongo
docker exec -it mongodb bash


## Maven Configuration
```xml
<dependency>
    <groupId>com.github.derjust</groupId>
    <artifactId>spring-data-dynamodb</artifactId>
    <version>5.1.0</version>
</dependency>
```

## Docker로 DynamoDB 구성
docker pull amazon/dynamodb-local
docker run -p 8000:8000 amazon/dynamodb-local -jar DynamoDBLocal.jar -inMemory -sharedDb

## DynamoDB를 조작하기 위해 AWS-CLI 설치

##  Webhook 서버를 위한 DynamoDB 테이블 구성
 
**>>> messageId + createdAt을 Global Secondary Index로 구성한 경우**  

* AWS_ACCESS_KEY_ID=x AWS_SECRET_ACCESS_KEY=x aws dynamodb create-table --endpoint-url http://localhost:8000 --table-name safetyapp_v2_dev  --attribute-definitions AttributeName=id,AttributeType=S AttributeName=messageId,AttributeType=S AttributeName=createdAt,AttributeType=S                       --key-schema AttributeName=id,KeyType=HASH --global-secondary-indexes IndexName=byMessageId,KeySchema=["{AttributeName=messageId,KeyType=HASH}","{AttributeName=createdAt,KeyType=RANGE}"],Projection="{ProjectionType=ALL}",ProvisionedThroughput="{ReadCapacityUnits=1,WriteCapacityUnits=1}" --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1
* AWS_ACCESS_KEY_ID=x AWS_SECRET_ACCESS_KEY=x aws dynamodb create-table --endpoint-url http://localhost:8000 --table-name safetyapp_v2_stg  --attribute-definitions AttributeName=id,AttributeType=S AttributeName=messageId,AttributeType=S AttributeName=createdAt,AttributeType=S                       --key-schema AttributeName=id,KeyType=HASH --global-secondary-indexes IndexName=byMessageId,KeySchema=["{AttributeName=messageId,KeyType=HASH}","{AttributeName=createdAt,KeyType=RANGE}"],Projection="{ProjectionType=ALL}",ProvisionedThroughput="{ReadCapacityUnits=1,WriteCapacityUnits=1}" --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1
* AWS_ACCESS_KEY_ID=x AWS_SECRET_ACCESS_KEY=x aws dynamodb create-table --endpoint-url http://localhost:8000 --table-name safetyapp_v2_edu  --attribute-definitions AttributeName=id,AttributeType=S AttributeName=messageId,AttributeType=S AttributeName=createdAt,AttributeType=S                       --key-schema AttributeName=id,KeyType=HASH --global-secondary-indexes IndexName=byMessageId,KeySchema=["{AttributeName=messageId,KeyType=HASH}","{AttributeName=createdAt,KeyType=RANGE}"],Projection="{ProjectionType=ALL}",ProvisionedThroughput="{ReadCapacityUnits=1,WriteCapacityUnits=1}" --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1
* AWS_ACCESS_KEY_ID=x AWS_SECRET_ACCESS_KEY=x aws dynamodb create-table --endpoint-url http://localhost:8000 --table-name safetyapp_v2_svc  --attribute-definitions AttributeName=id,AttributeType=S AttributeName=messageId,AttributeType=S AttributeName=createdAt,AttributeType=S                       --key-schema AttributeName=id,KeyType=HASH --global-secondary-indexes IndexName=byMessageId,KeySchema=["{AttributeName=messageId,KeyType=HASH}","{AttributeName=createdAt,KeyType=RANGE}"],Projection="{ProjectionType=ALL}",ProvisionedThroughput="{ReadCapacityUnits=1,WriteCapacityUnits=1}" --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1

```batch

AWS_ACCESS_KEY_ID=x             # Docker image를 사용하므로 임시로 값을 설정
AWS_SECRET_ACCESS_KEY=x         # Docker image를 사용하므로 임시로 값을 설정

aws dynamodb create-table 
--endpoint-url http://localhost:8000  # docker image 버전에서만 존재하는 endpoint url
--table-name safetyapp_v2_svc         # table name  

--attribute-definitions 
        AttributeName=id,AttributeType=S            # key-schema 필드
        AttributeName=messageId,AttributeType=S     # Global Secondary Index 필드 
        AttributeName=createdAt,AttributeType=S     # Global Secondary Index 필드                   

--key-schema 
        AttributeName=id,KeyType=HASH # table의 key 스키마 

--global-secondary-indexes                          # Key 스키마 외의 인덱스 설정 
        IndexName=byMessageId,KeySchema=["{AttributeName=messageId,KeyType=HASH}",
                                            "{AttributeName=createdAt,KeyType=RANGE}"],
                                            Projection="{ProjectionType=ALL}",
                                        ProvisionedThroughput="{ReadCapacityUnits=1,WriteCapacityUnits=1}" 
                
--provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1

```

**>>> messageId + createdAt을 Global Secondary Index로 구성한 경우**  
  
* AWS_ACCESS_KEY_ID=x AWS_SECRET_ACCESS_KEY=x aws dynamodb create-table --endpoint-url http://localhost:8000 --table-name safetyapp_v2_dev  --attribute-definitions AttributeName=id,AttributeType=S AttributeName=reportKey,AttributeType=N AttributeName=createdAt,AttributeType=S                       --key-schema AttributeName=id,KeyType=HASH --global-secondary-indexes IndexName=byReportKey,KeySchema=["{AttributeName=reportKey,KeyType=HASH}","{AttributeName=createdAt,KeyType=RANGE}"],Projection="{ProjectionType=ALL}",ProvisionedThroughput="{ReadCapacityUnits=1,WriteCapacityUnits=1}" --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1
* AWS_ACCESS_KEY_ID=x AWS_SECRET_ACCESS_KEY=x aws dynamodb create-table --endpoint-url http://localhost:8000 --table-name safetyapp_v2_stg  --attribute-definitions AttributeName=id,AttributeType=S AttributeName=reportKey,AttributeType=N AttributeName=createdAt,AttributeType=S                       --key-schema AttributeName=id,KeyType=HASH --global-secondary-indexes IndexName=byReportKey,KeySchema=["{AttributeName=reportKey,KeyType=HASH}","{AttributeName=createdAt,KeyType=RANGE}"],Projection="{ProjectionType=ALL}",ProvisionedThroughput="{ReadCapacityUnits=1,WriteCapacityUnits=1}" --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1
* AWS_ACCESS_KEY_ID=x AWS_SECRET_ACCESS_KEY=x aws dynamodb create-table --endpoint-url http://localhost:8000 --table-name safetyapp_v2_edu  --attribute-definitions AttributeName=id,AttributeType=S AttributeName=reportKey,AttributeType=N AttributeName=createdAt,AttributeType=S                       --key-schema AttributeName=id,KeyType=HASH --global-secondary-indexes IndexName=byReportKey,KeySchema=["{AttributeName=reportKey,KeyType=HASH}","{AttributeName=createdAt,KeyType=RANGE}"],Projection="{ProjectionType=ALL}",ProvisionedThroughput="{ReadCapacityUnits=1,WriteCapacityUnits=1}" --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1
* AWS_ACCESS_KEY_ID=x AWS_SECRET_ACCESS_KEY=x aws dynamodb create-table --endpoint-url http://localhost:8000 --table-name safetyapp_v2_svc  --attribute-definitions AttributeName=id,AttributeType=S AttributeName=reportKey,AttributeType=N AttributeName=createdAt,AttributeType=S                       --key-schema AttributeName=id,KeyType=HASH --global-secondary-indexes IndexName=byReportKey,KeySchema=["{AttributeName=reportKey,KeyType=HASH}","{AttributeName=createdAt,KeyType=RANGE}"],Projection="{ProjectionType=ALL}",ProvisionedThroughput="{ReadCapacityUnits=1,WriteCapacityUnits=1}" --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1

## 코드 예제
SDK를 직접 사용해서 하는 법
```java

package io.crscube.safetyapp.webhook.application;

import com.amazonaws.auth.AWSCredentials;
import com.amazonaws.auth.AWSCredentialsProvider;
import com.amazonaws.auth.AWSStaticCredentialsProvider;
import com.amazonaws.auth.BasicAWSCredentials;
import com.amazonaws.client.builder.AwsClientBuilder;
import com.amazonaws.services.dynamodbv2.AmazonDynamoDB;
import com.amazonaws.services.dynamodbv2.AmazonDynamoDBClientBuilder;
import com.amazonaws.services.dynamodbv2.model.*;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import io.crscube.safetyapp.webhook.interfaces.dto.PostWebhookRequest;
import io.crscube.safetyapp.webhook.interfaces.dto.PostWebhookRequest.CustomAttributes;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.apache.logging.log4j.util.Strings;
import org.springframework.stereotype.Component;

import java.util.HashMap;
import java.util.Map;
import java.util.UUID;

/**
 * Created by itaesu on 09/01/2020.
 *
 * @author Lee Tae Su
 * @version 2.0
 * @since 2.0
 */
@Slf4j
@Component @RequiredArgsConstructor
public class PostWebhookService {
    private final ObjectMapper objectMapper;

    public Map<String, AttributeValue> handle(PostWebhookRequest request) {
        AWSCredentials awsCredentials = new BasicAWSCredentials("key1", "key2");
        AWSCredentialsProvider awsCredentialsProvider = new AWSStaticCredentialsProvider(awsCredentials);

        AmazonDynamoDB amazonDynamoDb = AmazonDynamoDBClientBuilder.standard()
                                                                   .withCredentials(awsCredentialsProvider)
                                                                   .withEndpointConfiguration(
                                                                           new AwsClientBuilder.EndpointConfiguration("http://localhost:8000", "ap-northeast-2"))
                                                                   .build();

        Map<String, AttributeValue> item = new HashMap<>();
        item.put("id", (new AttributeValue()).withS("uuid"));
        item.put("messageId", (new AttributeValue()).withS(request.getMessageSid()));
        item.put("eventType", (new AttributeValue()).withS(request.getEventType()));
        item.put("index", (new AttributeValue()).withN(String.valueOf(request.getIndex())));
        item.put("channelSid", (new AttributeValue()).withS(request.getChannelSid()));
        item.put("body", (new AttributeValue()).withS(request.getBody()));

        Map<String, AttributeValue> attributes = new HashMap<>();
        CustomAttributes customAttributes;
        try {
            customAttributes = this.objectMapper.readValue(request.getAttributes(), CustomAttributes.class);
        } catch (JsonProcessingException e) {
            log.error("Custom attribute json parsing 중 에러 발생 {}", request.getAttributes(), e);
            customAttributes = new CustomAttributes();
        }
        attributes.put("userName", new AttributeValue().withS(customAttributes.getUserName()));
        attributes.put("reportKey", new AttributeValue().withN(customAttributes.getReportKey().toString()));
        attributes.put("twilioToken", new AttributeValue().withS(customAttributes.getTwilioToken()));
        item.put("attributes", (new AttributeValue()).withM(attributes));

        item.put("sender", (new AttributeValue()).withS(request.getFrom()));
        item.put("createdAt", (new AttributeValue()).withS(request.getDateCreated()));

        if (!Strings.isBlank(request.getMediaFilename())) {
            Map<String, AttributeValue> attachment = new HashMap<>();
            attachment.put("fileName", new AttributeValue().withS(request.getMediaFilename()));
            attachment.put("fileType", new AttributeValue().withS(request.getMediaContentType()));
            attachment.put("fileSid", new AttributeValue().withS(request.getMessageSid()));
            attachment.put("fileSize", new AttributeValue().withS(request.getMediaSize()));
            item.put("attachment", (new AttributeValue()).withM(attachment));
        }

        PutItemRequest putItemRequest = (new PutItemRequest())
                .withTableName("safetyapp_v2_dev")
                .withItem(item);

        PutItemResult putItemResult = amazonDynamoDb.putItem(putItemRequest);
        putItemResult.getSdkHttpMetadata().getHttpStatusCode();

        Map<String, AttributeValue> key = new HashMap<>();
        key.put("id", (new AttributeValue()).withS(UUID.randomUUID().toString()));

        GetItemRequest getItemRequest = (new GetItemRequest())
                .withTableName("safetyapp_v2_dev")
                .withKey(key);

        GetItemResult getItemResult = amazonDynamoDb.getItem(getItemRequest);
        return getItemResult.getItem();
    }
}

```


DynamoDbMapper를 이용하여 객체 매핑
```java
package io.crscube.safetyapp.webhook.application;

import com.amazonaws.auth.AWSCredentials;
import com.amazonaws.auth.AWSCredentialsProvider;
import com.amazonaws.auth.AWSStaticCredentialsProvider;
import com.amazonaws.auth.BasicAWSCredentials;
import com.amazonaws.client.builder.AwsClientBuilder;
import com.amazonaws.services.dynamodbv2.AmazonDynamoDB;
import com.amazonaws.services.dynamodbv2.AmazonDynamoDBClientBuilder;
import com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBMapper;
import com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBMapperConfig;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import io.crscube.safetyapp.webhook.domain.model.DevChat;
import io.crscube.safetyapp.webhook.domain.model.ChatFactory;
import io.crscube.safetyapp.webhook.interfaces.dto.PostWebhookRequest;
import io.crscube.safetyapp.webhook.interfaces.dto.PostWebhookRequest.CustomAttributes;
import io.crscube.safetyapp.webhook.interfaces.dto.PostWebhookResponse;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

import static io.crscube.safetyapp.webhook.interfaces.dto.PostWebhookResponse.from;

/**
 * Created by itaesu on 09/01/2020.
 *
 * @author Lee Tae Su
 * @version 2.0
 * @since 2.0
 */
@Slf4j
@Component @RequiredArgsConstructor
public class PostWebhookService {
    private final ObjectMapper objectMapper;

    public PostWebhookResponse handle(PostWebhookRequest request) {
        CustomAttributes customAttributes;
        try {
            customAttributes = this.objectMapper.readValue(request.getAttributes(), CustomAttributes.class);
        } catch (JsonProcessingException e) {
            log.error("Custom attribute json parsing 중 에러 발생 {}", request.getAttributes(), e);
            customAttributes = new CustomAttributes();
        }

        AWSCredentials awsCredentials = new BasicAWSCredentials("key1", "key2");
        AWSCredentialsProvider awsCredentialsProvider = new AWSStaticCredentialsProvider(awsCredentials);

        AmazonDynamoDB amazonDynamoDb = AmazonDynamoDBClientBuilder.standard()
                                                                   .withCredentials(awsCredentialsProvider)
                                                                   .withEndpointConfiguration(
                                                                           new AwsClientBuilder.EndpointConfiguration("http://localhost:8000", "ap-northeast-2"))
                                                                   .build();
        DynamoDBMapper dynamoDBMapper = new DynamoDBMapper(amazonDynamoDb, DynamoDBMapperConfig.DEFAULT);

        final Chat devChat = ChatFactory.build(
                request.getMessageSid(),
                request.getEventType(),
                request.getIndex(),
                request.getChannelSid(),
                request.getBody(),
                customAttributes,
                request.getFrom(),
                request.getDateCreated(),
                request.getMediaFilename(),
                request.getMediaContentType(),
                request.getMediaSid(),
                request.getMediaSize());
        dynamoDBMapper.save(devChat);

        return from(dynamoDBMapper.load(Chat.class, devChat.getId()));
    }
}

```
