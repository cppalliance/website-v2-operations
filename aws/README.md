<!--
Copyright (c) 2024 The C++ Alliance, Inc. (https://cppalliance.org)

Distributed under the Boost Software License, Version 1.0. (See accompanying
file LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)

Official repository: https://github.com/boostorg/website-v2
-->

## Amazon AWS Notes

AWS S3 buckets in the us-east-2 region store the following content.  

Media:

  - This includes user profile photos, news content, and other Django media items.

Docs: 
  - archives/ - Boost documentation. Full boost releases.
  - site-docs/ - Content from the https://github.com/boostorg/website-v2-docs build/ directory. See GHA in that repo.
  - site-pages/ - Content from the https://github.com/boostorg/website-v2-docs site-pages/ directory. See GHA in that repo.

| Bucket Name | Description |
| ----------- | ----------- |
| boost.org.media | Media - production environment |
| boost.org.v2 | Docs - production |
| stage.boost.org.media | Media - stage environment |
| stage.boost.org.v2 | Docs - stage |
| boost.revsys.dev | Docs - revsys cluster |
| boost.org-cppal-dev-v2.media | Media - cppal-dev testing environment |
| boost.org-cppal-dev-v2 | Docs - cppal-dev |
