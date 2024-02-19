<!--
Copyright (c) 2024 The C++ Alliance, Inc. (https://cppalliance.org)

Distributed under the Boost Software License, Version 1.0. (See accompanying
file LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)

Official repository: https://github.com/boostorg/website-v2
-->

## Mailman Notes

Mailman-core servers have been installed:  

mailman.boost.cppalliance.org  
mailman-stage.boost.cppalliance.org  
mailman-cppal-dev.boost.cppalliance.org  
mailman-revsys.boost.cppalliance.org  

The core servers ought to be re-installed using new DNS. For now, they may be enough to test.  

```
MAILMAN_REST_API_URL = env("MAILMAN_REST_API_URL", default="http://localhost:8001")
MAILMAN_REST_API_USER = env("MAILMAN_REST_API_USER", default="restadmin")
MAILMAN_REST_API_PASS = env("MAILMAN_REST_API_PASS", default="restpass")
MAILMAN_ARCHIVER_KEY = env("MAILMAN_ARCHIVER_KEY", default="password")
MAILMAN_ELASTIC_INDEX = env("MAILMAN_ELASTIC_INDEX", default="haystack")
``` 

Those environment variables are provided by kube secrets which have been uploaded in the cluster.  

Mailman3 is composed of 3 main parts:  
  - postorius  
  - hyperkitty  
  - mailman-core  

Include postorius and hyperkitty into the web project, via both these pip packages:  
  - mailman-web  
  - mailman-hyperkitty   

Another pip package is installed on the mailman-core servers only. External to the web project.   
  - mailman  

In terms of the database it's a similar idea. The mailman-web database would become part of the main Django database. The mailman database is separate. Exclude mailman-core from both the web project and the web database.  

This ansible role https://github.com/cppalliance/ansible-mailman3 has a 'master' branch to install a full mailman instance, and a 'mailman3-core' branch which only installs mailman core and was used to install the above servers. See the docs/ folder of that repository also.  

ElasticSearch is running on each mailman instance. Django is then configured to access that.  

