# Spring boot executable jar 설정하기

## 1. EC2 인스턴스 준비
### JDK 설정

JDK 설치 (1.8)
sudo yum install -y java-1.8.0-openjdk-devel.x86_64

새로 설치한 설정으로 대체
sudo /usr/sbin/alternatives --config java

EC2의 기존 JDK 제거
sudo yum remove java-1.7.0-openjdk 

### Maven build

아래 옵션을 포함하여 메이븐 빌드

```xml
<plugin>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-maven-plugin</artifactId>
	<configuration>
		<executable>true</executable>
	</configuration>
</plugin>
```

아래 명령어 실행 (app 이름이 동일해야 하는 듯)
sudo ln -s /home/ec2-user/app/myapp.jar /etc/init.d/myapp
-> 반드시 절대 경로로

jar와 동일 위치에 myapp.conf로 아래와 같이 줄 수 있음

JAVA_OPTS="-Xmx1024M -Dspring.profiles.active=dev"
LOG_FOLDER=/home/ec2-user/safetyapp/log
LOG_FILENAME=server.log

jar와 동일 위치에 application 설정 파일 위치하여 관리할 수 있음


### aws를 쓴다면
~/.aws/credentials 파일 생성
sudo mkdir ~/.aws
cd ~/.aws
sudo touch credentials 로 생성해야 함

<a href="https://docs.spring.io/spring-boot/docs/1.3.1.RELEASE/reference/html/deployment-install.html">55. Installing Spring Boot applications</a>
