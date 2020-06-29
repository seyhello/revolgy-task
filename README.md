# Demo Cloud Solution for Revolgy
- Cloud example 1 - AWS / EC2 - just for self-learning
- Cloud example 2 - AWS / ECS Fargate - Main cloud solution

## Prerequisites
- AWS account (Access Key ID and Secret Access Key required) with administrative permissions
- !!Only for EC2 example!! Generated SSH KeyPair and imported in AWS account, *.pem file saved on local disk (you will be prompted for file name and path)

### EC2 deployment
- TerraformEC2 folder
- run terraform init / apply
- after successfull deployment, web application should be accessible via http://<public_ip>:8080

#### Front APP
- will be deployed automatically
- static HTML page
- exposed on port 8080
- doesn't communicate with backend DB for now

#### Backend APP just for learning - not in use
- POSTGRE SQL database
- exposed on port 5432

## ECS deployment
- **TerraformECSFargate** folder
- My "Hi Revolgy" web app doesn't use database, but there is an example how to create PostgreSQL database in AWS
- CI/CD workflow is done on GitHub side, described below

### Service A / Service B
- two different revisions of the webapp used in EC2 example - service-a and service-b directories
- static HTML page served by nginx, containerized by Docker
- simplified Docker hub repository: **seyhello/revolgy-task:service-a[b]** (It would be better to use unique repositories for each service and use :tags for app versions, this is for demonstration)
- CI workflow (defined in **BuildServiceA[B]Container.yaml**): docker image is built and pushed to the docker hub automatically when there are changes in service-a or service-b directories. "git push" starts the workflow
- CD workflow (defined in **DeployServiceA[B].yaml)**: starts CI workflow and deploy the service to the AWS ECS cluster as Fargate service. It is ok as demonstration example because fixed variables are used in .yaml file. There would be better to use dynamic variables, but I didn' find out the way how to do it.

### Creating cluster
- just run **terraform init / apply** in the root directory
- you will be prompted for the VPC name (or project name, in this case)
- VPC, subnets, IGW, IAM role and ECS cluster is created

### Managing service(s)
- To make it simple for you, there are two BASH scripts in the root project directory for managing services

#### add-service.sh
- script gets *terraform output* from cluster creation and service parameters specified in TFVARS file, and finally run *terraform init / apply* in its own directory and deploys the service to the cluster
- you must specify <service>.tfvars file as required parameter, for example:
  ```
  ./add-services.sh services/service-a.tfvars
  ```
- examples for service A and service B are in **services** directory
- TFVARS file defines variables required for service creation - service name, docker repository, application port, instances count, number of CPUs and memory. File should look like this
  ```
  app_name       = "service-a"
  app_image      = "seyhello/revolgy-task:service-a"
  app_port       = 80
  app_count      = 2
  fargate_cpu    = 1024
  fargate_memory = 2048
  ```
- *you will be also prompted for the **project** name. This will be used for unique naming AWS resources*
  
#### remove-service.sh
- run this script for removing deployed service
- you will be prompted for the service name
- script will run *terraform destroy* in the service directory and finally deletes the directory

### Adding database(s)

### Removing database(s)
