<!--
Copyright (c) 2024 The C++ Alliance, Inc. (https://cppalliance.org)

Distributed under the Boost Software License, Version 1.0. (See accompanying
file LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)

Official repository: https://github.com/boostorg/website-v2
-->

Website Admin
==============

Staging Sync
--------------

Occasionally it's helpful to sync the production database and S3 assets over to the staging environment, so there is more content available such as blog posts and user accounts.  

An Ansible server is configured on admin-server.boost.cpp.al. There is role cppalliance.stagesync being developed which is capable of syncing to staging. It is functional. Ongoing dev.

