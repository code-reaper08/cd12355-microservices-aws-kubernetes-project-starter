# Coworking Space Service Extension
The Coworking Space Service is a set of APIs that enables users to request one-time tokens and administrators to authorize access to a coworking space. This service follows a microservice pattern and the APIs are split into distinct services that can be deployed and managed independently of one another.

For this project, you are a DevOps engineer who will be collaborating with a team that is building an API for business analysts. The API provides business analysts basic analytics data on user activity in the service. The application they provide you functions as expected locally and you are expected to help build a pipeline to deploy it in Kubernetes.


## Cluster Provisioning:
We utilize `eksctl` to create an EKS cluster on AWS. This step involves defining the cluster name, region, node group configuration, and desired number of nodes.

## PostgreSQL Database Deployment:
We define yaml files for persistent volume claims (PVCs), persistent volumes (PVs), and a Postgres deployment. These YAML files specify storage requirements and deployment configuration for the PostgreSQL database. We apply these yaml files and deploy the postgresql DB. Alternatively, we leverage Helm charts to install and configure a managed PostgreSQL instance within the cluster. Once deployed we check access and seed the sql files provided into the DB.

## Analytics Application Testing:
Install application dependencies using `pip install -r requirements.txt`. Run the` app.py` script locally to verify application functionality. Create a `Dockerfile` that defines the application build process. We use a Python base image and install application dependencies within the container. Build the Docker image locally using `docker build .` to ensure a functional container is created. We run the docker image built to see everything is working fine. 

We then modify `coworking.yam` and `configmap.yaml` to define deployment configurations and secrets for the application in the Kubernetes cluster. Deploy these YAML files using `kubectl apply -f deployment/` to create deployments and services for the analytics application. Test the deployed application APIs using `curl` commands to validate functionality and database connectivity.

## ECR Repository:
Create an Amazon Elastic Container Registry (ECR) repository to store the container image. This repository will serve as a private registry for our application images.

## CI/CD Pipeline with CodeBuild:
We define a `buildspec.yml` file that specifies the build instructions for CodeBuild. This file references the Dockerfile and defines steps for building, tagging, and pushing the image to the ECR repository. Create a CodeBuild project in the AWS console. Configure the project to use the `buildspec.yml` file, grant it access to the EKS cluster and ECR repository, and define any necessary environment variables. Pushing code commits to the main branch will trigger the CodeBuild project. The project will build the Docker image using the Dockerfile, tag it with a unique identifier (e.g., build number), and push it to the ECR repository.

## Deployment with Kubernetes Manifests:
Create Kubernetes deployment for our application that reference the container image stored in ECR. These yaml files define how the application should be deployed within the cluster, including replicas and environment variables. Apply the files using `kubectl apply -f deployment/` to deploy the containerized application to the Kubernetes cluster. We can verify the deployment by running `kubectl get svc` and now we should see our application service up and running. If all good, we should be able to `curl` using the external IP.

## Monitoring with CloudWatch:
Enable the Amazon CloudWatch Observability EKS add-on within the cluster. This add-on facilitates collecting and storing logs from applications running on the cluster.
Configure the CloudWatch agent on the cluster nodes to collect and send logs from the Coworking Space analytics application. Access CloudWatch logs to monitor application health and troubleshoot any issues.
