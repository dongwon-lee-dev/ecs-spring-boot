# ecs-spring-boot

## 1) Create a Spring Boot application
1. Download JDK
https://www.openlogic.com/openjdk-downloads

2. Edit the system environment variables
System variables
JAVA_HOME: C:\Program Files\OpenLogic\jdk-17.0.13.11-hotspot
PATH: %JAVA_HOME%\bin

* Run java -version in CMD to check the actual current version of JDK 
* Check if there are directories to previous versions of jdk in PATH and delete them

![project-setting](image/project-setting.png)
![gradle-setting](image/gradle-setting.png)

Dockerfile
```bash
FROM openjdk:17-alpine
WORKDIR app
COPY target/docker-example-microservice-0.0.1-SNAPSHOT.jar ./docker-example.jar 
ENTRYPOINT ["java","-jar","/app/docker-example.jar"]
```

## ECR
IAM user with AmazonEC2ContainerRegistryFullAccess or else.
aws configure

Run ECR push commands

ECS
Cluster: name
Task family: name, container name, container image uri, container port, app protocol (HTTP, HTTP, GRC, None)
Service: name, desired task number, vpc, subnet, security group, ALB, Auto scaling

Target group: 
ALB: 

ecs
public instance + public ip enabled + public ALB
priate insance + public ip disabled + manually create target group and public ALB
