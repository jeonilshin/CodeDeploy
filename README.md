# AWS CodeDeploy
## What is AWS CodeDeploy
It is a service offered by Amazon that is used for the purpose of deployment. It helps in the automation of the application deployment of **Amazon EC2 Instances, On-Premise Instances, Serverless Lambda Functions, and ECS Services.**

CodeDeploy has been designed in such a manner that it can work with multiple systems to manage configurations, perform source control, continuous integration, continuous deployment (CI-CD), and continuous delivery. In addition to this, it provisions a way of quickly searching for the user resources (which includes repositories, project builds, deployment applications, pipelines). CodeDeploy can deploy applications to the below mentioned compute platforms:

### EC2 or on premise
This platform talks about the physical servers which could be Amazon EC2 cloud instances, on premise servers or both. When an application is created with the help of EC2 or on-premise compute platform, it can contain executable files, configuration files, images and other types of data. When an application is deployed using the EC2 or on-premise compute platform, it helps manage the way in which traffic is routed to that instance, with the help of an in-place or blue-green deployment type.

### AWS Lambda
It is used to deploy applications which have an updated version of the Lambda function. Lambda helps in the management of Lambda function which is present in a server less compute environment. This environment contains high-available resources. The monitoring of these compute resources is taken care of by AWS Lambda itself.  
When an application is created with the help of the Lambda compute platform, it manages the method in which traffic is routed to the updated Lambda function’s versions. It also manages the deployment by choosing one of the methods from a canary, linear or all-at-once configuration.

### Amazon ECS
It is used to deploy Amazon ECS applications which have been containerized as a part of the task set. CodeDeploy deploys such applications using the blue/green deployment, wherein it installs an updated version of the containerized application as the new replacement task set. Once this is done, CodeDeploy reroutes the traffic from the original containerized application to the replaced task set. Once this is completed successfully, the original task gets terminated.  

### CodeDeploy comes with two methods of deployment: 

#### In-place deployment
In this method, the application present on every instance of the deployment group is stopped, and the latest application (which would have been revised) is installed. After this, the new version of this application starts and gets validation. A load balancer can be used so that every instance gets deregistered when it is being deployed and is restored when the deployment is complete. In-place deployments can be used only when the applications use EC2 or on-premise compute platform.  

AWS Lambda and Amazon ECS can’t deploy their applications using an in-place deployment.  

![image](https://user-images.githubusercontent.com/86287920/212469593-3d76b9d4-a3eb-4c4d-81ae-106cd6cd4b46.png)

#### Blue/green deployment
In this method, the deployment’s characteristics depends on which compute platform is used to deploy the application.  

- **Blue/green deployment on EC2 or on-premise compute platform:** The original environment’s instances are replaced by a different environment’s instances. The replacement environment is provisioned. The latest application (which is revised) gets installed as the replacement instance. There is an optional wait time so as to perform application testing and system verification. This newly replaced instance is registered with an Elastic Load Balancing load balancer, due to which the traffic gets rerouted to these instances. The instances present in the original environment are deregistered, and can be terminated as well, or could be run to perform other operations.  

An important point to remember is that blue/green deployments work only with Amazon EC2 instances.  

- **Blue/green deployment on AWS Lambda compute platform:** The traffic that comes to the current server less environment is rerouted to the updated Lambda function version. Lambda functions can be specified to perform validation tests, and specific way can be chosen to handle the shift in traffic. Any deployment which is performed on the AWS Lambda compute platform is considered as a blue/green deployment. This is the reason why a deployment type need not be specified while using Lambda compute platform.  
- **Blue/green deployment on Amazon ECS compute platform:** This is used with containerized applications. The traffic is rerouted from the original version of a containerized application which is present in Amazon ECS to a replacement task set that is present in the same ECS service. This production traffic is rerouted by specifying the protocol and port of the load balancer. When a deployment takes place, a test listener is used to handle traffic of the replacement task set while the validation tests are being executed.
<p align="center"><a href="https://github.com/aws-samples/ecs-blue-green-deployment"><img src="https://user-images.githubusercontent.com/86287920/189579005-cf687792-b03d-4549-a5d3-08bc5dae6fda.png" width="800" /></a></p>

## What is an Appspec.yml file?
The appspec.yml file stands for *application specification file*, app spec file for short, and is a file unique to CodeDeploy. It is designed to manage our deployments by a series of hooks, or events, that are defined in the Hooks section of the file. This is a file in YAML or JSON format. This file tells CodeDeploy what to install on our instances, and what to run based on hooks in response to our updates to the code.

### Examples of different Appspec files
**Lambda Appspec.yml file**
```
version: 0.0
Resources:
  - myLambdaFunction:
      Type: AWS::Lambda::Function
      Properties:
        Name: "<Lambda_Name>"
        Alias: "<Lambda_Alias>"
        CurrentVersion: "1"
        TargetVersion: "2"
Hooks:
  - BeforeAllowTraffic: "LambdaFunctionToValidateBeforeTrafficShift"
  - AfterAllowTraffic: "LambdaFunctionToValidateAfterTrafficShift"
```

**EC2 Appspec.yml file**
```
version: 0.0
os: linux
files:
  - source: /
    destination: /home/ec2-user
hooks:
  AfterInstall:
    - location: <sh_location>
      timeout: 180
      runas: root
  ApplicationStart:
    - location: <sh_location>
      timeout: 180
      runas: root
  ApplicationStop:
    - location: <sh_location>
      timeout: 180
      runas: root
```

**ECS Appspec.yml file**
```
version: 0.0
Resources:
  - TargetService:
      Type: AWS::ECS::Service
      Properties:
        TaskDefinition: "<TASK_DEFINITION>"
        LoadBalancerInfo:
          ContainerName: "<Container_Name>"
          ContainerPort: "<Container_Port>"
```

# Example with Tutorial
Please make sure that you add the following files to your project for this to work
 - appspec.yml
 - scripts/installapache
 - scripts/restartapache
 - scripts/startapache

#### Create IAM Roles
- CodeDeploy(iam.txt)
- EC2CodeDeploy(iam.txt)
 
#### Create EC2 instance with following categories

a. Choose AMI: Amazon Linux AMI

b. Choose Instance type: t2.micro

c. Configure Instance: Choose EC2CodeDeploy IAM role

d. Tag Instance: Name it what you please

e. Configure Security Group:
```
HTTP TCP 80 0.0.0.0/0

HTTP TCP 80 ::/0

SSH TCP 22 (YOUR IP ADDRESS)

HTTPS TCP 443 0.0.0.0/0

HTTPS TCP 443 ::/0
```
f. LAUNCH INSTANCE

#### Login to EC2 instance

#### Command line of Amazon Linux AMI

##### a. When server is booted
```
sudo su

yum -y update

yum install -y aws-cli

cd /home/ec2-user
```
##### b. Here you will setup your AWS access, secret, and region.
```
aws configure

aws s3 cp s3://aws-codedeploy-us-east-1/latest/install . --region ap-northeast-2

aws s3 cp s3://aws-codedeploy-us-west-2/latest/install . --region ap-northeast-2

chmod +x ./install
```
##### c. This is simply a quick hack to get the agent running faster.

```
sed -i "s/sleep(.*)/sleep(10)/" install

./install auto
```

##### d. Verify it is running.
```
service codedeploy-agent status 
```
