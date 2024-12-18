# ecs-spring-boot

![springboot-backend](image/springboot-backend.jpg)

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

In case of Gradle
![gradle-setting](image/gradle-setting.png)

Creating a JAR file using Maven
![create-jar](image/create-jar.png)


Dockerfile
```bash
FROM openjdk:17-alpine
WORKDIR app
COPY target/docker-example-microservice-0.0.1-SNAPSHOT.jar ./docker-example.jar 
ENTRYPOINT ["java","-jar","/app/docker-example.jar"]
```

### Load Balancer, Target Group, and Domain
1. ACM create -> get CNAME and Value
2. Domain DNS management create CNAME record
3. Check ACM validated
4. Set Load Balancer the SSL certificate

1. Registered Domain - CNAME record main domain or subdomain (*) - Load Balancer DNS
2. Add Listener 443 with SSL certificate
3. Listener 80 - Redirect URL to Listener 443
4. Listener 443 - Add Listener Rule with Host Header condition
* Check ECS Service Security Group - Inbound Port 3000 for ALB Health check

### Website access Typo Error
Check Route 53 CNAME record TTL 
Load Balancer Host header condition listener rule change takes time according to DNS record TTL

### Too many redirections
Nginx HTTP to HTTPS configuration 
```bash
sudo ls /etc/nginx/sites-enabled/
cat /etc/nginx/sites-enabled/local  # Also check other files
```

## ECR
IAM user with AmazonEC2ContainerRegistryFullAccess or else.
aws configure

Run ECR push commands

ECS
# Task - Container port
# Target Group - 80
# ALB - 80

Cluster: name
Task family: name, container name, container image uri, container port, app protocol (HTTP, HTTP, GRC, None)
Service: name, desired task number, vpc, subnet, security group, ALB, Auto scaling

Target group: 
ALB: 

+ ecs task role IAM

ALB log -> S3 policy 
```bash
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::elb-account-id:root"
      },
      "Action": "s3:PutObject",
      "Resource": "s3-bucket-arn"
    }
  ]
}
```
elb-account-id
us-east-1	127311923021
us-east-2	033677994240
us-west-1	027434742980
us-west-2	797873946194

ecs
public instance + public ip enabled + public ALB
priate insance + public ip disabled + manually create target group and public ALB


# RDS
RDS - EC2 / Lambda / ECS container
![rds-connection](image/rds-connection.jpg)

### Secret Manager
1) Make Role with TaskExecutionPolicy + SecretManagerReadWrite or custom policy getSecret -> Give Role to [Task execution role]
2) ValueFrom Secret ARN:key::

!!!
Mixed Content: The page at 'https://a.dongwonlee.dev/' was loaded over HTTPS, but requested an insecure resource 'http://backend2.devcluster1:3001/api/email'. This request has been blocked; the content must be served over HTTPS.
backend API server endpoint should be https
make each other security group Inbound / Outbound


Local MySQL connection - Use separate host and port
```bash
    const dbConfig = {
      host: "localhost",
      port: 3306,
      user: "root",
      database: "items",
    };
```

EC2 - SecretManager
1. Add IAM Role with getSecret Policy
2. Check script on Secret Manger Secret for getting the secret
3. EC2: sudo apt install mysql-client / sudo apt install python3 / sudo apt install python3-boto3 / sudo apt install python3-pymysql / use script to use secret 
```python
import os
import boto3
import json
import subprocess

def get_secret():
    secret_name = <Secret Name>
    region_name = <Region>

    session = boto3.session.Session()
    client = session.client(
        service_name="secretsmanager",
        region_name=region_name
    )

    response = client.get_secret_value(SecretId=secret_name)
    secret = json.loads(response["SecretString"])
    return secret["password"]

def connect_to_mysql():
    password = get_secret()
    os.environ["MYSQL_PWD"] = password  
    command = [
        "mysql",
        "-h", <RDS instance endpoint>,
        "-u", <RDS instance user name>
    ]
    subprocess.run(command)

if __name__ == "__main__":
    connect_to_mysql()
```


# ECS communication between Services
Use Service Connect
!!! Task Definition - Port Mapping Name - Manually Input, do not auto create

!!! Use Chrome developer tools to get more information if there is a problem
Mixed Content: browser does not accept http response when https request is sent -> Nextjs use proxy api: frontend first send request to its backend and then send it to official backend

# CICD
Dockerfile, buildspec.yaml, appspec.yaml in project folder

### CodeBuild - permission + AmazonEC2ContainerRegistryReadOnly
