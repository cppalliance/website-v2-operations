<!--
Copyright (c) 2024 The C++ Alliance, Inc. (https://cppalliance.org)

Distributed under the Boost Software License, Version 1.0. (See accompanying
file LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)

Official repository: https://github.com/boostorg/website-v2
-->

## GCP Notes

The website is hosted on Google Cloud Platform in a project named "boostorg-project1" within a CPPAlliance account.

- An Autopilot kubernetes cluster is running in the us-central1 region.

- A redis memorystore instance was created for each environment.

- Archive Registry stores docker images of each website release.

---

GKE - Kubernetes

Set up new k8s cluster:

In GCP create project boostorg-project1  
Enable Kubernetes Engine API  
Create a cluster boostorg-cluster1 in us-central1  
Used all defaults settings. That means, it will choose the network ranges.  
Regular channel. default.  

Review and create.    
Double check your cluster settings. Pay extra attention to the ones that can't be changed later.  

```
 Cluster basics
  Cluster name: boostorg-cluster1 
  Cluster location: us-central1 
 Networking
  Network: default
  Subnetwork: default
  Network access: Public cluster 
  Override control plane's default private endpoint subnetwork: Disabled
  Cluster default pod address range: /17 
  Service address range: /22 
  Control plane authorized networks: Disabled
 Advanced settings
  Release channel: Rapid channel
  Maintenance window: Disabled
  Anthos service mesh: Disabled 
  Binary authorization: Disabled
  Google Groups for RBAC: Disabled
  Secret encryption at the application layer: Disabled
  Boot disk encryption: Google-managed
``` 

On a local computer, follow these steps to gain access.  

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

---

Memorystore Redis Instances

https://console.cloud.google.com/memorystore/redis/instances?referrer=search&project=boostorg-project1

| Name | Type | Version | Location | Capacity |
| ---- | ---- | ------- | -------- | -------- |
| cppal-dev-redis | Basic | 6.x | us-central1 | 1 GB |
| production-redis | Standard |	6.x | us-central1 | 1 GB |
| stage-redis |	Basic |	6.x | us-central1 | 1 GB |

---

Artifact Registry

The path to the container images is noted in the Github Actions workflow file:  

```
DOCKER_IMAGE: "us-central1-docker.pkg.dev/boostorg-project1/website/website"
DOCKER_REGISTRY: "us-central1-docker.pkg.dev"
```

https://console.cloud.google.com/artifacts/docker/boostorg-project1/us-central1/website/website?project=boostorg-project1

In IAM, created a service account that will push the new images.  
"boostwebsitesa"  
boostwebsitesa@boostorg-project1.iam.gserviceaccount.com  
Gave this service account Artifact Registry Reader and Writer permissions.  

```
export GKE_PROJECT=boostorg-project1
export SA_EMAIL=boostwebsitesa@boostorg-project1.iam.gserviceaccount.com

gcloud projects add-iam-policy-binding $GKE_PROJECT \
	--member=serviceAccount:$SA_EMAIL \
	--role=roles/container.admin

gcloud projects add-iam-policy-binding $GKE_PROJECT \
	--member=serviceAccount:$SA_EMAIL \
	--role=roles/storage.admin

gcloud projects add-iam-policy-binding $GKE_PROJECT \
	--member=serviceAccount:$SA_EMAIL \
	--role=roles/container.clusterViewer

```	

Keys exist. Checking:  
```
gcloud iam service-accounts keys list --iam-account=$SA_EMAIL
```	

Example of creating a key:    
```
gcloud iam service-accounts keys create key2.json --iam-account=$SA_EMAIL
```

Example usage:  
```
export GKE_SA_KEY=$(cat key.json | base64)  
export GKE_SA_KEY=$(cat boostwebsitesa.json.base64)  
```
or  
```
export GKE_SA_KEY=$(cat boostwebsitesa.json | base64)
echo "$GKE_SA_KEY"   # notice the quotes
```

Created GHA secret GKE_SA_KEY based on the json key. Not base64 encoded.  

GHA requires docker credentials also. Using the same account.  

GKE_DOCKER_REGISTRY_USERNAME, literally the string "_json_key_base64"  
GKE_DOCKER_REGISTRY_PASSWORD, the contents of the key provided by Google in boostwebsitesa.json.base64  

In Artifact Registry, a Cleanup Policy removes images older than 6 months.  
