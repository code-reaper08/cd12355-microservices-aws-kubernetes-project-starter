# Coworking Space Service Extension
The Coworking Space Service is a set of APIs that enables users to request one-time tokens and administrators to authorize access to a coworking space. This service follows a microservice pattern and the APIs are split into distinct services that can be deployed and managed independently of one another.

For this project, you are a DevOps engineer who will be collaborating with a team that is building an API for business analysts. The API provides business analysts basic analytics data on user activity in the service. The application they provide you functions as expected locally and you are expected to help build a pipeline to deploy it in Kubernetes.

## Getting Started

### 1. Creating a cluster

```bash
eksctl create cluster --name my-rubric-cluster --region us-east-1 --nodegroup-name my-rubric-nodes --node-type t3.small --nodes 1 --nodes-min 1 --nodes-max 2
```
Using `eksctl` create a cluster, here I created a cluster called `my-rubric-cluster` with additional args of region, and have also spin-up a nodegroup called `my-rubric-nodes`. I also specified the number of nodes in this cluster with min and max flags.

