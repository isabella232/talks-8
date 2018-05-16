# Day 2, Exercise 1

## Step 1/12

_[Only do this step if you didn’t do it in your cluster earlier]_

Create the namespace we will use for this exercise:

```
kubectl create namespace dev
```

Shortly, the Deploy agent will notice this change, and sync the Deployment and Service files.

Watch for this happening in Weave Cloud or via:

```
watch kubectl -n dev get all
```

----

## Step 2/12

We’re going to make a code change and see it flow through CI, then deploy that change.
Call the version endpoint on the service to see what is running:

```
curl podinfo.dev:9898/version
```

----

## Step 3/12

In the editor, open `podinfo/pkg/version/version.go`, increment the version number and save:

```
var VERSION = "0.3.1"
```

Commit your changes and push to master:

```
cd /workspace/podinfo
git commit -m "release v0.3.1 to dev" .
git push
```

----

## Step 4/12

The CI pipeline will create an image tagged the same as the git commit
Git said something like `[master 89b8396]`; the tag will be like `master-89b8396`

Check by listing image tags (replace user with your username):

```
gcloud container images list-tags gcr.io/dx-training/USER-podinfo
```

_`USER`_ should be of the form `training-user-<number>`.

----

## Step 5/12

Navigate in the editor to `workspace/cluster/dev` and open `podinfo-dep.yaml`.

Where it says `image:`

replace `quay.io/stefanprodan/podinfo` with `gcr.io/dx-training/USER-podinfo`
replace the tag `0.3.0` with your tag `master-TAG`

Save the file and commit your changes and push to master:

```
cd ../cluster/dev
git commit -m "my first deploy" .
git push
```

----

## Step 6/12

Check in Weave Cloud to see when it has synced the Deployment.

Call the version endpoint on the service to see if it changed:

```
curl podinfo.dev:9898/version
```

----

## Step 7/12

Editing the YAML file was tedious. Let’s automate it!

In Weave Cloud Deploy, find the `podinfo` Deployment in `dev` Namespace.

Click Automate.
This creates a commit, because git is our single source of truth.

To keep things in sync, bring it into your workspace:

```
git pull
```

----

## Step 8/12

Open `podinfo/pkg/version/version.go`, increment the version number again, and save:

```
var VERSION = "0.3.2"
```

Commit your changes and push to master:

```
cd /workspace/podinfo
git commit -m "release v0.3.2" .
git push
```

----

## Step 9/12

Watch for the CI/CD to upgrade the app to 0.3.2:

```
watch curl podinfo.dev:9898/version
```

----

## Step 10/12

Suppose we don’t like the latest version: we want to roll back.

1. In Weave Cloud Deploy, find the `podinfo` Deployment in `dev` Namespace. Click Deautomate.

2. The UI shows a list of images - select the one you want and click Release, then again to confirm.

3. Check again which version is running:

```
watch curl podinfo.dev:9898/version
```

----

## Step 11/12

We can flag that the latest build should not be deployed

1. In Weave Cloud Deploy, find the podinfo Deployment in dev Namespace. Click 🔒Lock.

2. Give a reason, then click Lock again to confirm.

3. Each of these actions creates a git commit. Sync your workspace:

```
git pull
```

4. Reload `/workspace/cluster/dev/podinfo-dep.yaml` in the editor to see how the lock is applied.

----

## Step 12/12

We can flow the version number through the pipeline with a git tag, to show more meaningful versions

Create and push a git tag:

```
cd /workspace/podinfo
git tag 0.3.2
git push origin 0.3.2
```

This will trigger another CI build, and when that is finished you should have an image tagged `0.3.2`

