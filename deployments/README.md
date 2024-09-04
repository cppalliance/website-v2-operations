<!--
Copyright (c) 2024 The C++ Alliance, Inc. (https://cppalliance.org)

Distributed under the Boost Software License, Version 1.0. (See accompanying
file LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)

Official repository: https://github.com/boostorg/website-v2
-->

## Deployments

If you are interested in testing the Boost website-v2, it is possible you do not need a full cloud deployment. Some alternatives include:

- Local docker-compose: [development_setup_notes](https://github.com/boostorg/website-v2/blob/develop/docs/development_setup_notes.md)
- The existing staging site: https://www.stage.boost.cppalliance.org/
- The existing production site: https://www.boost.io/

All of those options are more convenient. In particular, docker-compose. If you will be joining the C++ Alliance's development efforts, code may be checked in directly to the staging and production sites, so another site isn't needed.

## Cloud Deployment

Review the other documentation in this repository including GCP, AWS, Fastly, and mailman.

The website is hosted in a kubernetes cluster on GKE. Launch and configure a GKE cluster in autopilot mode.  

The actual deployment is handled by Github Actions. Those jobs are included in the codebase of https://github.com/boostorg/website-v2. Go through the GHA files and replace all secrets and environment variables, such that they point at a new cluster, or the intended cluster.

In the current environment, when code is checked into the `develop` branch, it will deploy to the k8s `stage` namespace. When code is checked into the `master` branch, it will deploy to the k8s `production` namespace.

It is strongly recommended to commit new code changes in `develop` branch, and when they have passed testing, fast-forward merge them into the `master` branch.  Do not allow the two git branches to permanently diverge.

Create AWS S3 buckets. A "media" bucket corresponding to "boost.org.media", and a "docs" bucket corresponding to "boost.org.v2". Contact a boost website developer or administrator to get a copy of the staging AWS keys. That will allow you to copy all the content - mainly Boost library documentation.

The production website is able to stay up-to-date with Boost content via the `boostorg/website-v2-docs` github actions, as well as the `boostorg/release-tools` periodic deployments. Those also keep the staging website up-to-date. Any other copy of the site that is neither "production" nor official "staging" will inevitably become stale over the course of time. Sync S3 buckets from production or staging, if needed.

A Fastly CDN is not strictly required to have a functional testing setup. It may not be necessary during testing. If used, check with current admins or developers to be sure you have the newest copy of the Fastly VCL code which has been deployed live.
