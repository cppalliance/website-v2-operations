
## Website Environments

Currently there are multiple instances of the website running:

- production
- stage
- cppal-dev  

Each is a separate independent copy of the website code based on a github branch, and generally not overlapping with other environments, except they share a similar copy of the python Django codebase.

In this document, let's refer to these as "environments". They can equally be thought of as "websites", "instances", "copies", etc.  

Adding another such environment, perhaps a "test" instance, is a moderately significant undertaking. The choice is of doing that is neither an absolute Yes or No, but all the pros and cons should be considered. Most likely if a comparable solution can be found that is less costly or time intensive than a new environment, the other solution should be preferred.  

This document can be viewed as a description of how boost.org websites are deployed from an admin perspective, as well as attempting to address the topic of "more environments". 

Each instance involves a one-time cost of configuring it initially. And then ongoing maintenance, day-to-day tasks.  

Let's explore the details of a new environment. Or, skip ahead to [Developer Workflow](#developer-workflow) below.  

- New kubernetes "namespace"  

- New set of "secrets" and "configmaps" in kubernetes:
  - django-secret-key
  - fastly-secret
  - general-web-secrets (calendar)
  - githubtoken
  - hyperkitty-secret
  - mailgun-secret
  - media-bucket access
  - pg-secret (postgres)
  - slack token
  - static-content-secret

Each of the above "passwords" or "api" keys must be generated and set up.

- New database on the existing postgres database server.

- New Fastly CDN service. Each website has it's own separately maintained service, with VCL rules, and all rules are manually copied into each service, every time.

- Boostlook CI/deployment. The boostlook repository uses master/develop branches, and within github actions the css file is copied dynamically into the corresponding master/develop branches of website-v2 and website-v2-docs. Hypothetically imagine another "test" environment.  If that "test" environment were considered important, and were the focus of testing by multiple developers or end-users, then the boostlook CI process would need to take this into account, and "deploy" boostlook to the new branch.  And then we must keep track of the procedure. 


- AWS S3. Each environment is assigned two S3 buckets. A standard content bucket, and a media bucket. Create these. They require API keys to be generated, and set in k8s.  

- The kube/ directory in website-v2 has a values.yml files for each environment. Add another values.yaml file. 

- website-v2 Github Actions CI. The CI files reference specific branches. Update this CI process. 

- DNS entries. 

- SSL certificates, generated on both on GCP, and Fastly.

- Release tools - daily archive generation. Every time any boost library is updated, multiple times per day, a new archive is built, and that process uploads master/ and develop/ to AWS where the webpages are hosted and viewed. Configure the new environment here, because each environment gets its own copy of the master/ develop/ archives.  

- Release tools - boost releases (every 3-4 months).  When the release manager publishes the next version of boost, one of the steps is uploading the webpages, yet again, to S3, where they are hosted as the main documentation for end-users.  This is a GBs of files, and each environment gets its own copy.  The file transfer takes time.  Each new environment adds a GB of data transfer that occurs.   

## Developer Workflow  

Most boost libraries, and also the boost.org website, are using a gitflow type methodology. "A successful Git branching model"  https://nvie.com/posts/a-successful-git-branching-model/

That is based on develop/master branches. 

If another branch is introduced, you have to define where it comes into play within the gitflow model.  

For example, if the branch is analogous to a local developer copy of the website, it's mostly outside the gitflow, and would be "just another test environment". That is how cppal-dev works, being an extraneous test environment to experiment with k8s features before deploying them.  

On the other hand, if a new test environment is being introduced into gitflow:  test/develop/master ... where developers submit PRs to 'test' instead of 'develop', that is quite different. There are many concerns. The final decision to merge a pull request still must happen. That is, an open PR goes from pending to merged. Now it's in the codebase, in a branch.  Then testing may occur.  Ok.   But a question is - whether it's really preferable to for there to be 3 such branches, instead of 2 branches (develop/master). With 2 branches (develop/master) you are able to test extensively - locally.   And then, also, test in "staging".  This is quite a bit already.   The boost.org website is not a risky medical application,  for example. Yes, it should be tested.   But the risk of bugs is not so weighty that extreme caution is needed.

## Questions

Even if proceeding to create another environment, these are questions that come to mind, although they may not be blockers.   

What must be tested that can't be done either in "staging", or "locally", but requires yet another environment?  

Think through the gitflow very methodically. Who merges what, and when? Step by step. Carefully describe the day to day pull request process. Can fast-forward git history be maintained?  Is the new process more cumbersome or error-prone than it was before?  Imagine the situation, dozens of times, hundreds of times, when a developer submits code changes.  To which git branch? To which website.

Therefore I propose returning to this idea: "the test environment is analogous to a local developer copy of the website". "an extra environment".    

Such a method won't adversely affect the main gitflow, but can augment it with further testing. When a new Pull Request is introduced, the QA department runs git commands which will merge the Pull Request into a test environment and it will be online. This environment can deviate from staging if needed.  

## Alternatives  

- Local testing. Opened Github pull requests may be pulled down by other developers, and tested locally in each of their own development environment.  

- In staging. The point of running a staging environment is to test new changes before they go live. After local testing seems correct, merge to staging, and test again.  

- In cppal-dev. After thinking about the problem more, using the existing cppal-dev ought to solve the problem in the most efficient way.  


## Proposed Solution

Share the existing cppal-dev environment, which has not been shared previously. This environment is now controlled by the cppal-dev git branch of https://github.com/cppalliance/website-v2-qa. Grant the QA department permissions.  

We'll need to coordinate maintenance or planned outages, since the same environment can be used for k8s, infrastructure, and QA testing.  

These are the steps to merge PRs into the cppal-dev branch of cppalliance/website-v2-qa.  


```
# Initial configuration. Create a folder and checkout the repo.
mkdir -p $HOME/github/cppalliance
cd $HOME/github/cppalliance
git clone https://github.com/cppalliance/website-v2-qa
cd website-v2

# Initial configuration. Add the upstream repo.
git remote add upstream https://github.com/boostorg/website-v2
git fetch upstream

# Each time when testing a Pull Request
# sync the repo
git fetch upstream
# discover the latest commit
export LATEST_COMMIT=$(git rev-parse -q --verify upstream/develop)
# or manually
git log upstream/develop
# Checkout the relevant branch which is cppal-dev
git checkout cppal-dev
# Set this branch to the latest commit
git reset --hard ${LATEST_COMMIT}

# Merge the PR
export PR_NUMBER=___
git fetch origin pull/$(PR_NUMBER)/head:pr-${PR_NUMBER}
git merge pr-${PR_NUMBER}

# make sure you are on cppal-dev
git branch
# push the code
git push --force
```

Experiment with the above steps locally first, without pushing remotely.   

After a merge, observe Github Actions, which take 5-10 minutes: https://github.com/cppalliance/website-v2-qa/actions

The site will be live at https://www.cppal-dev.boost.org 

Occasionally merge production data into cppal-dev.   

Therefore (the plan can be) the contents of cppal-dev are not completely stable, but will sometimes be imported and overwritten from production.   

## New Developer Workflow

Continue to submit PRs to the `develop` branch. The standard gitflow is not disrupted. The same as before.  

PRs are opened, but they haven't been merged yet. Send a request for review/approval to the QA department. Unless it's a very basic code fix, a review should be requested from QA.  

When that happens, run the git commands listed above. Those can eventually be streamlined into an improved script, so they are easier and quicker to run frequently. That will deploy the PR onto https://www.cppal-dev.boost.org. Test. Approve the PR. Then the PR can be merged directly into develop.  

Based on the example commands shown above, create a new bash shell script `./merge-pr.sh 1234` which merges a pull request easily into https://github.com/cppalliance/website-v2-qa  in one step. This will involved adding some if-else statements to skip certain steps.  

Just as a reference, another bash script, that accomplishes a different task, is here [deploy-website.sh](https://github.com/boostorg/website-v2/blob/develop/docs/scripts/deploy-website.sh). Maybe the QA department can work on developing `merge-pr.sh`? Let me know. 

