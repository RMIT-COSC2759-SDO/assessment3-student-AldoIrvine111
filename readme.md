# Servian TechTestApp
```
Student ID: S3692192 
Name: Aldo Irvine
```

This is a documentation for ACME corp. project.
Previously, we have containerized the application and expanded the CI build to create a container and publish it. Now we are starting to use Kubernetes to host the application.

Due to ACME corp.'s lack experience with DevOps, we will be using SaaS tools to integrate this project. We will be using: 

### 1. Terraform
Terraform is a provisioning infrastructure tool, like building, changing, and versioning. It as an AWS Partner Network (APN) Advanced Technology Partner and member of the AWS DevOps Competency, hence why this tool is choosen. 

### 2. AWS
Amazon Web Services, or more likely known as AWS, is a secure cloud services platform. It offer lots of service that help businesses scale and grow. Some of the services we will be using is EC2, where we will make and store our Virtual Machine (VM), RDS, for our databases, VPC, where we will use it to isolate our resources and secure them.

### 3. CircleCI
CircleCI is an Continuous Integration tools. This help developer to integrate their code into a master branch of a shared repo early and often.

### 4. Docker
Docker is a tool designed to make it easier to create, deploy, and run applications by using containers. Containers allow a developer to package up an application with all of the parts it needs, such as libraries and other dependencies, and deploy it as one package.

### 5. Kubernetes
Kubernetes is a portable, extensible, open-source platform for managing containerized workloads and services, that facilitates both declarative configuration and automation. It has a large, rapidly growing ecosystem. Kubernetes services, support, and tools are widely available.

### 6. Helm
Helm is the first application package manager running atop Kubernetes. It allows describing the application structure through convenient helm-charts and managing it with simple commands.

## Create a HELM chart to deploy the application to Kubernetes.

First, we need to create a new directory on the root folder of the project and change it to the directory and initialize Helm. 

```bash
mkdir helm 
cd helm
helm create acme
```
![repo images](https://github.com/RMIT-COSC2759-SDO/assessment3-student-AldoIrvine111/blob/master/pic/helm.PNG)

**helm create acme** will initialize the project name acme and it will generate several files such as template folder (which stores the deployment.yaml and services.yaml), charts.yaml, and values.yaml. For this application, we will create a deployment and a service to allow us to access the application from
the internet (an ingress controller will automatically be created for us). We will configure the application to use port 80.

We will set image and database endpoint by using variables which we will store in values.yaml. By using variables, we will achieve easier maintainability and reusability of the code. It will collect the value of the variable from values.yaml and then passed it back.


![repo images](https://github.com/RMIT-COSC2759-SDO/assessment3-student-AldoIrvine111/blob/master/pic/deployment.PNG)
![repo images](https://github.com/RMIT-COSC2759-SDO/assessment3-student-AldoIrvine111/blob/master/pic/services.PNG)
![repo images](https://github.com/RMIT-COSC2759-SDO/assessment3-student-AldoIrvine111/blob/master/pic/values.PNG)

This is the variable for Images and Database Endpoints.

Now the helm is configured and we will move on to the next step.

## Deploy the application into a non-production environment

Before we start deploying, we need to set CircleCI to our GitHub repository and set AWS credentials on our CircleCI.

Next, we are going to deploy our Kubernetes Cluster with scripts which was written before.

```bash
cd environment
make up
``` 

make up will generate an output which we will use to configure on our Makefile in infra folder and config.yaml. These outputs are our information for S3 bucket, dynamoDB, and ECR URL which we deployed in AWS. 

![repo images](https://github.com/RMIT-COSC2759-SDO/assessment3-student-AldoIrvine111/blob/master/pic/output.PNG)

We are going to set up a remote backend for terraform, to store the statefile somewhere we can be accessed by the CI pipeline. We do this by setting up an S3 bucket and a DynamoDB table. We will make some changes to our terraform deployment Makefile to point the correct bucket and table
![repo images](https://github.com/RMIT-COSC2759-SDO/assessment3-student-AldoIrvine111/blob/master/pic/init.PNG)

Moving on, we will set s3 bucket too along with ECR URL in our CircleCI config.yaml.
![repo images](https://github.com/RMIT-COSC2759-SDO/assessment3-student-AldoIrvine111/blob/master/pic/cibucket.PNG)
![repo images](https://github.com/RMIT-COSC2759-SDO/assessment3-student-AldoIrvine111/blob/master/pic/ciecr.PNG)

We are configuring this to allow CircleCI to access to our existing dynamoDb, ECR, and s3 bucket on our AWS. We will then change back to environment directory and make kube-up

```
cd environment
make kube-up
```

This will set-up VPC and subnet in our AWS console and we will take their ID and configure it inside our terraform.tfvars. This allow our app to use the VPC and subnet-ID of our cluster that we have built.
![repo images](https://github.com/RMIT-COSC2759-SDO/assessment3-student-AldoIrvine111/blob/master/pic/subnet.PNG)
![repo images](https://github.com/RMIT-COSC2759-SDO/assessment3-student-AldoIrvine111/blob/master/pic/vpc.PNG)
![repo images](https://github.com/RMIT-COSC2759-SDO/assessment3-student-AldoIrvine111/blob/master/pic/tfvars.PNG)

Now all we have to do is configure the CircleCI config.yaml.

We will be deploying the application into a non-production environment. We will build a new job which we call **deploy-test**, set the image using cimg/base:2020.01 (this is a docker image we are using) and then set the environment to test. Then, add attach_workspace to the step section to allow us to access the terraform and other artefacts. Next, attach setup-cd command which we have configured to match our selected s3 bucket.

Next, we will setup the infrastructure which we call **deploy infra** and add this job in our circleCI workflow. We are configuring this job to run in test environment and deploy database to back the application. Do not forget to 

```
terraform output endpoint > ../dbendpoint.txt
```
so we can get the dbendpoint and pass it to the deploy app part. We need to set the environment to test on all our command. To prevent error 137 while upgrading database, invoke
```
sleep 10
```
 to upgrade database command.

![repo images](https://github.com/RMIT-COSC2759-SDO/assessment3-student-AldoIrvine111/blob/master/pic/workflowtest.PNG)
![repo images](https://github.com/RMIT-COSC2759-SDO/assessment3-student-AldoIrvine111/blob/master/pic/deploy-test.PNG)
![repo images](https://github.com/RMIT-COSC2759-SDO/assessment3-student-AldoIrvine111/blob/master/pic/testtest.PNG)
![repo images](https://github.com/RMIT-COSC2759-SDO/assessment3-student-AldoIrvine111/blob/master/pic/testinfra.PNG)
![repo images](https://github.com/RMIT-COSC2759-SDO/assessment3-student-AldoIrvine111/blob/master/pic/test.PNG)



### Change the end-to-end test to run against the non-production environment 

To set e2e test to run against test production, we will need to get the external IP of the apps in test cluster. 

We can get the External-IP by invoking
```
kubectl get services -n test
```

-n test is to set tell the command to get the app that is in the test environment. 

Set the workflow from circleCi that to run e2e , it will require deploy-test to finish. 

![repo images](https://github.com/RMIT-COSC2759-SDO/assessment3-student-AldoIrvine111/blob/master/pic/kubectl.PNG)

![repo images](https://github.com/RMIT-COSC2759-SDO/assessment3-student-AldoIrvine111/blob/master/pic/e2ecode.PNG)

![repo images](https://github.com/RMIT-COSC2759-SDO/assessment3-student-AldoIrvine111/blob/master/pic/e2etest.PNG)

![repo images](https://github.com/RMIT-COSC2759-SDO/assessment3-student-AldoIrvine111/blob/master/pic/teste2e.PNG)

![repo images](https://github.com/RMIT-COSC2759-SDO/assessment3-student-AldoIrvine111/blob/master/pic/workflowe2e.PNG)


### Deploy the application into a production environment 

Next we will deploy the application into production environment, we will build a new job in our pipeline call **deploy-prod**.  We can recycle the all the code from **deploy-test** and change all the environment from test to prod. 

![repo images](https://github.com/RMIT-COSC2759-SDO/assessment3-student-AldoIrvine111/blob/master/pic/deploy-prod.PNG)

![repo images](https://github.com/RMIT-COSC2759-SDO/assessment3-student-AldoIrvine111/blob/master/pic/prod.PNG)

![repo images](https://github.com/RMIT-COSC2759-SDO/assessment3-student-AldoIrvine111/blob/master/pic/prodinfra.PNG)

![repo images](https://github.com/RMIT-COSC2759-SDO/assessment3-student-AldoIrvine111/blob/master/pic/prodprod.PNG)


### Create a stage-gate before deployments are allowed into production

We want to be able to only deploy approved changes to production, so we need to put a stage gate. We will create a new job in workflow called approval, set the type as approval and make the job to require deploy-test.
We will then set up the dependency of the job **deploy-prod** to required **approval**. This way, it will prompt from CircleCI pipeline for approval to continue deployment to the production environment. Once we approved the job, it will start **deploy-job**.
![repo images](https://github.com/RMIT-COSC2759-SDO/assessment3-student-AldoIrvine111/blob/master/pic/workflowprod.PNG)

![repo images](https://github.com/RMIT-COSC2759-SDO/assessment3-student-AldoIrvine111/blob/master/pic/approval.PNG)



