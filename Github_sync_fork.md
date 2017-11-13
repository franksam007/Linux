
### Configuring a remote for a fork

You must configure a remote that points to the upstream repository in Git to sync changes you make in a fork with the original repository. This also allows you to sync changes made in the original repository with the fork.

1. Open Terminal.

2. List the current configured remote repository for your fork.

    <code>git remote -v</code>
    <pre><code>origin  https://github.com/YOUR_USERNAME/YOUR_FORK.git (fetch)
    origin  https://github.com/YOUR_USERNAME/YOUR_FORK.git (push)</code></pre>

3. Specify a new remote upstream repository that will be synced with the fork.

    <code>git remote add upstream https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git</code>

4. Verify the new upstream repository you've specified for your fork.

    <code>git remote -v</code>
    <pre><code>origin    https://github.com/YOUR_USERNAME/YOUR_FORK.git (fetch)
    origin    https://github.com/YOUR_USERNAME/YOUR_FORK.git (push)
    upstream  https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git (fetch)
    upstream  https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git (push)</code></pre>

### Syncing a fork

Sync a fork of a repository to keep it up-to-date with the upstream repository.

Before you can sync your fork with an upstream repository, you must configure a remote that points to the upstream repository in Git.

1. Open Terminal.

2. Change the current working directory to your local project.

3. Fetch the branches and their respective commits from the upstream repository. Commits to master will be stored in a local branch, upstream/master.

    <code>git fetch upstream</code>
    <pre><code>remote: Counting objects: 75, done.
    remote: Compressing objects: 100% (53/53), done.
    remote: Total 62 (delta 27), reused 44 (delta 9)
    Unpacking objects: 100% (62/62), done.
    From https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY
     * [new branch]      master     -> upstream/master</code></pre>

4. Check out your fork's local master branch.

    <code>git checkout master</code>
    Switched to branch 'master'

5. Merge the changes from upstream/master into your local master branch. This brings your fork's master branch into sync with the upstream repository, without losing your local changes.

    <code>git merge upstream/master</code>
    <pre><code>Updating a422352..5fdff0f
    Fast-forward
     README                    |    9 -------
     README.md                 |    7 ++++++
     2 files changed, 7 insertions(+), 9 deletions(-)
      delete mode 100644 README   
      create mode 100644 README.md</code></pre>

6. If your local branch didn't have any unique commits, Git will instead perform a "fast-forward":

    <code>git merge upstream/master</code>
    <pre><code>Updating 34e91da..16c56ad
    Fast-forward
     README.md                 |    5 +++--
     1 file changed, 3 insertions(+), 2 deletions(-)</code></pre>



