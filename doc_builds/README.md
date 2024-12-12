
## Doc Builds

How are docs built? This is broad question which can have multiple meanings. Let's explore each of the topics.

[Antora Docs on the website](#antora-docs-on-the-website)  
[Boost Docs on the website](#boost-docs-on-the-website)  
[Boost Archives](#boost-archives)  
[Doc Previews](#doc-previews)  

## Antora Docs on the website 

The page https://www.boost.io/docs/ hosts the [User Guide](https://www.boost.io/doc/user-guide/index.html) and [Contributor Guide](https://www.boost.io/doc/contributor-guide/index.html).

Those are developed in the repository https://github.com/boostorg/website-v2-docs

Each time a commit is made, GitHub Actions runs. It executes a local script `libdoc.sh`, which runs Antora, and then the actions job uploads the resulting build/ directory to S3. More info about those S3 buckets in [aws](../aws/README.md). The Django website reads the content directly from S3 and presents it to the user on the website.

More details:  

The website-v2-docs build process is covered in [website-v2-docs/antora-ui/README.adoc](https://github.com/boostorg/website-v2-docs/blob/develop/antora-ui/README.adoc). Antora knows about the boostorg/boostlook repository and automatically downloads the latest boostlook.css. See website-v2-docs/antora-ui/gulp.d/tasks/build.js.  

When a new commit is made in the boostorg/boostlook repository a Github Action in boostorg/boostlook remotely triggers the boostorg/website-v2-docs workflows to run.

A commit is pushed to the static/ folder of website-v2, `develop` branch. That causes a new deployment of staging on website-v2.

## Boost Docs on the website

Each Boost library's docs are published on the website at a URL such as https://www.boost.io/doc/libs/release/doc/html/boost_asio.html .

There is a long chain of events leading to a page being displayed:

- A boost author commits a modification to a boost library in their GitHub repository. 
- The superproject (boostorg/boost) commitbot runs on a schedule in GitHub Actions. It notices this commit, and updates the superproject.
- CircleCI notices a change to boostorg/boost, and so the CircleCI job runs. It builds a release using boostorg/release-tools.  
- The release is uploaded the AWS S3 bucket s3://boost-archives. 
- A cron script on the EC2 instances brorigin1,2 downloads files from s3://boost-archives every minute. These EC2 servers are backends for the Fastly hosted https://archives.boost.io/master/ or https://archives.boost.io/develop/ , where the files are published for download.  
- Additionally the release-tools script (ci_boost_release.py) run by CircleCI uploads a copy of the files to Django's S3 bucket. More info about those S3 buckets in [aws](../aws/README.md).  
- boost.io hosts that content. It is displayed directly from the S3 source. In the case of "master" and "develop", the URL can be found by replacing the version string with the branch:  https://www.boost.io/doc/libs/develop/doc/html/boost_asio.html
- Official releases are created every 3 months by the release managers when they run the release-tools publish_release.py script. That step also uploads to Django's S3, in a similar way as how the master and develop snapshots were uploaded.  
- It would be easy to imagine that publish_release.py builds a boost release, but in fact during that step an existing snapshot from the master branch is copied and renamed. In other words, 'building the docs from scratch' is (surprisingly) not part of the release process. A master branch archive is re-used.

## Boost Archives

The above steps covered a high-level overview of the release process. To see more in-depth information, review the python scripts ci_boost_release.py and publish_release.py in https://github.com/boostorg/release-tools.  All boost submodules are checked out, b2 is executed, etc.

## Doc Previews

Master Branch / Develop Branch / Per-Pull-Request

Built by Jenkins - see https://github.com/cppalliance/jenkins-ci/tree/master/docs

