
## Git Cherry Pick methodology

Typically the `develop` and `master` branches should stay in-sync, and always follow a fast-forward process. That means there are no merge commits, and each branch has an identical git history over the course of time.  

Occasionally it's necessary to diverge from this temporarily, by checking in a few commits separately, and this will cause the `develop` and `master` branches to vary. 

SOLUTION:

STEP 1: check-in selected commits, as needed.   

```
git checkout master
git pull
git cherry-pick commit
git cherry-pick commit
git push
```

Now `master` has been deployed with specific unusual commits.

STEP 2: "recovery"

Some time later, perhaps the following week, it's time to reconcile the git branches so they match again.  

1. Be sure to check out the latest copy of all branches, before continuing to any other steps:  

```
git checkout develop
git pull
git checkout master
git pull
```

2. Merge `develop` into `master`. This creates a merge commit.

```
git checkout master
git merge develop               
git push origin master
```

3. Merge `master` back into `develop`.  

```
git checkout develop
git merge master                
git push origin develop
```

However Step 3 may fail to merge anything into `develop` because it already contains all changes. Git refuses to act, in this case. If the branches are not yet on the same commit, as shown by `git log`, then proceed further to Step 4.  

4. (when necessary). Create a small modification to force a merge.

```
git checkout master
vi README.md
# Add a . someplace, or a space in-between words, but in a such a way that linting/testing won't fail.
# A blank space at the end of a line will cause a linting error.
git add .
git commit -m "Merge fix"
git push
git checkout develop
git merge master
git push 
```

5. Verify results  

- Locally, run `git log` on both the `develop` and `master` branches. The most recent commits should be identical.  
- On GitHub, compare the most recent commits of the `develop` and `master` branches. They should be the same.  
- On GitHub, confirm that GitHub Actions ran successfully.  

