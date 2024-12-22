# Coworking Space Service Extension
The Coworking Space Service is a set of APIs that enables users to request one-time tokens and administrators to authorize access to a coworking space. This service follows a microservice pattern and the APIs are split into distinct services that can be deployed and managed independently of one another.

For this project, you are a DevOps engineer who will be collaborating with a team that is building an API for business analysts. The API provides business analysts basic analytics data on user activity in the service. The application they provide you functions as expected locally and you are expected to help build a pipeline to deploy it in Kubernetes.

## Getting Started

### Pre-checks which need to be done
I used `aws configure` to configure my account. `AWS_SESSION_TOKEN` is also essential to establish a successfull connection. Once done, to check whether the connection succeeded, we use `aws sts get-caller-identity`.

### 1. Creating a cluster

```bash
eksctl create cluster --name my-rubric-cluster --region us-east-1 --nodegroup-name my-rubric-nodes --node-type t3.small --nodes 1 --nodes-min 1 --nodes-max 2
```
Using `eksctl` create a cluster, here I created a cluster called `my-rubric-cluster` with additional args of region, and have also spin-up a nodegroup called `my-rubric-nodes`. I also specified the number of nodes in this cluster with min and max flags. We check whether the cluster is created and all the nodes defined are up and running.

### 2. Get PostgreSQL running
In short from my understanding,

`pvc.yaml` => This claims persistant storage from cluster

`pv.yaml` => This defines the storage that is being claimed by `pvc.yaml`

`postgresql-deployment.yaml` => This file defines the Postgresql deployment.

when `kubectl apply` the above 3 files, the resources will be created in the cluster from the details defined in the above config yaml files.

We can run `kubectl get pods` to see if the resource is created successfully

We then create a new file called `postgresql-service.yaml` which defines a new postgresql service. This service will expose the db for other applications in the cluster.

We then apply this service file and check if it is successfull by running `kubectl get services`

We can port forward the db service and connect to it to see if the things are working fine.

We can setup Postgres DB using helm. For this we need to add a bitnami repo as,

```bash
helm repo add postgresql-service https://charts.bitnami.com/bitnami
```

Once set with the repo we can insatll the appropriate helm chart. We can exec inside the pod and run `psql` to see if the connection succeeded. If suceeded we run the seed files provided to get the datadump inside our DB.

### 3. Test analytics application locally
To test the analytics application locally, we first `pip install` the `requirements.txt` file. Once the install succeeds, when then simply run the `app.py` file provided. This should start a local flask server.

We then modify the `coworking.yaml` and `configmap.yaml`. `coworking.yaml` will reference `configmap.yaml` and creates a deployment definition for our analytics application. `coworking.yaml` will also contains definition to create a service in our cluster for our application called `coworking`.

We now apply these two files as `kubectl apply -f deployment/`. Once this command succeeds, we now have two services, one for DB and one for the actual application in our cluster.

Then we open a new bash, and try to curl on the below two endpoints to see if the API is working and the db connection within was successfull or not.

```bash
curl 127.0.0.1:5153/api/reports/daily_usage
curl 127.0.0.1:5153/api/reports/user_visits
```
Once successful we now initiate the deployment process.

### 4. Dockerfile to get the analytics application to the cluster
We now write a Dockerfile providing it with steps to create a working image. We use a python base image (I'm using python:3.9-slim) and build on top of it. Once done we can build the image locally to see if it is indeed able to create an container.

### 5. Create a ECR Repository
ECR repository will be our container registry repository, where we'll store our images. We create a new ECR repository via console and keep it ready for our CodeBuild project to interact with in the next step

### 6. Write a `buildspec.yml` for CodeBuild to pick up
We write a `buildspec.yml`, a yaml/yml file which is used by AWS CodeBuild to create a image on a hook trigger (ex., pusing to main branch, merging to main branch etc.,) from the repo it is hosted. This paves way for CI/CD for our project. We now create a new AWS Codebuild project in the AWS Console and assign relevant role and accesses to read/write to our EKS Cluster. We also add the needed env variables to the CodeBuild project. The builspec build the image with our Dockerfile, tag it and pushes it to the ECR repo. We tag the image with `CODEBUILD_BUILD_NUMBER` which will serve our need for incremental versioning.


### 7. Give our repo a push!
We then initiate a commit to the main branch of our repo. This commit initiates a CodeBUild job. This can be viewed in CodeBuild project's build history. Wait for the status to be updated to `succeeded`. Once succeeded, we could see that CodeBuild has built our Dockerfile, tagged it and pushed it to our ECR repository.


### 8. Setup CloudWatch logs/monitoring
To setup CloudWatch navigate to the our EKS Cluster we first created. We now navigate to the cluster node. add an add-on to our cluster called `Amazon CloudWatch Observability EKS add-on` 
For this to work we also need to provide our node with `CloudWatchAgentServerPolicy` IAM policy. Once done we now can navigate to the CloudWatch log groups and click on our `coworking` app and click on a log stream and we can monitor and observe our `coworking` application live.

