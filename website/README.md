
Website Admin
==============

Staging Sync
--------------

Occasionally it's helpful to sync the production database and S3 assets over to the staging environment, so there is more content available such as blog posts and user accounts.  

An Ansible server is configured on admin-server.boost.cpp.al. There is role cppalliance.stagesync being developed which is capable of syncing to staging. It is functional. Ongoing dev.

