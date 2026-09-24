<!--
Copyright (c) 2024 The C++ Alliance, Inc. (https://cppalliance.org)

Distributed under the Boost Software License, Version 1.0. (See accompanying
file LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)

Official repository: https://github.com/boostorg/website-v2
-->

## GCP Notes for Developers

This document is aimed at developers who need to reach the website services running on Google Cloud Platform,
to open a shell on a pod or read the logs. But you are more than welcome to read the other page,
[ADMINISTRATORS.md](./ADMINISTRATORS.md), which is aimed at the administrators who create and maintain that
infrastructure.

Access is granted by an administrator. If the commands below report that you are not authorized, ask an admin
to add your account.

## Connect to the web server instances

On a local computer, follow these steps to gain access.  

Hint: it's convenient to launch a docker container locally, and run all of this in a container which will isolate the environment on your computer. However that is not required.  

Install gcloud. https://cloud.google.com/sdk/docs/install#deb

Login to gcloud:

```
gcloud auth login
```

Set configuration: 

```
gcloud config set project boostorg-project1
gcloud config set compute/region us-central1
```

Get credentials to the cluster:

```
gcloud container clusters get-credentials boostorg-cluster1
```

or

```
gcloud container clusters get-credentials boostorg-cluster1 --region us-central1 --project boostorg-project1
```

Configure kubectl:

```
alias k='kubectl'
k config set-context --current --namespace=__
```

Run kubectl commands:

```
k get pods -n cppal-dev
k get pods -n stage
k exec -it pod/__ -n stage -- bash
```

## Observability, Monitoring, Logs

The quickest way to read the logs of a single pod, once you have cluster access as described above:

```
k logs -f pod/__ -n stage
```

Logs from all environments are also collected in Cloud Logging, which is searchable and retains history
after a pod has been replaced:

https://console.cloud.google.com/logs/query

Choose the GCP project in the console before running a query. Get the project name from your admin.

An example query, limiting the results to the containers of one namespace:

```
resource.type="k8s_container"
resource.labels.namespace_name="stage"
```

To view GKE observability, navigate to Kubernetes Engine, Clusters, the Boost cluster, Workloads -> boost,
and open the Observability tab. That page charts CPU, memory, and traffic for the workload. We must balance
security and clarity, so a direct URL is not published here. Ask an admin if you have trouble locating the
cluster or the workload.
