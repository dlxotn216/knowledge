프로젝트를 진행하면서 간단히 Local 환경에 Spring application 과 mongodb로 테스트를 할 일이 생겨  
설정을 진행한 내용을 정리한다.

### POM.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.2.2.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>io.crscube.safetyapp</groupId>
	<artifactId>chat-webhooks</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>chat-webhooks</name>
	<description>Webhook client for chat service</description>

	<properties>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-mongodb</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-mail</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
			<scope>runtime</scope>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-configuration-processor</artifactId>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>de.flapdoodle.embed</groupId>
			<artifactId>de.flapdoodle.embed.mongo</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
			<exclusions>
				<exclusion>
					<groupId>org.junit.vintage</groupId>
					<artifactId>junit-vintage-engine</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
				<configuration>
					<executable>true</executable>
				</configuration>
			</plugin>
		</plugins>
		<finalName>twilio-webhooks</finalName>
	</build>

</project>

```


#### Docker run container

sudo yum -y install docker #도커 설치   

sudo service docker run   
도커가 실행 중이 아니라면 아래와 같은 에러가 날 수 있다   
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?  


docker run --name mongo-test -p 27017:27017 -e MONGO_INITDB_ROOT_USERNAME=root -e MONGO_INITDB_ROOT_PASSWORD=mongodb -d mongo

#### 만약 접근이 필요하다면
docker exec -it mongo-test bash


#### application.yml
```yaml
logging:
  level:
    org.springframework.data.mongodb.core.MongoTemplate: DEBUG
    org.springframework.data.mongodb.core.ReactiveMongoTemplate: DEBUG

spring:
  profiles:
    active: local
  data:
    mongodb:
      host: localhost
      port: 27017
      database: test
      authentication-database: admin
      username: root      # MONGO_INITDB_ROOT_USERNAME
      password: mongodb   # MONGO_INITDB_ROOT_PASSWORD
```



`com.mongodb.MongoTimeoutException: Timed out after 30000 ms while waiting to connect. Client view of cluster state`
위와 같은 에러가 난다면 mongodb 컨테이너가 정상적으로 떠 있는지 포트가 연결 되었는 지 확인이 필요하다


`com.mongodb.MongoCommandException: Command failed with error 13 (Unauthorized): 'command insert requires authentication`
위와 같은 에러 발생 시 username, password 관련 정보가 안맞거나 없는 것이다.



