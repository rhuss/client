## Crafting a new release

### Merge in upstream release to openshift repo:

```bash
# Check that a remote reference to openshift and upstream exists
$ git remote -v | grep -e 'openshift\|upstream'
openshift	git@github.com:openshift/knative-client.git (fetch)
openshift	git@github.com:openshift/knative-client.git (push)
upstream	https://github.com/knative/client.git (fetch)
upstream	https://github.com/knative/client.git (push)

# Create an openshift branch pointing to openshift client's master
# (if not already done)
$ git co -b openshift openshift/master

# Go to scripts dir
$ cd openshift/release

# Create new release branch. Parameters are the upstream release tag
# and the name of the branch to create
$ ./create-release-branch.sh v0.9.0 release-v0.9.0

# Push release back to openshift branch
$ git push openshift release-v0.9.0
```

### Create a CI config and re-create Prow jobs

* Create a fork of https://github.com/openshift/release (if not already done)
* Create a new config YML for your release. Copy over `release-next.yaml` and adapt it by updating the image name from `knative-nightly` to a version specific name `knative-v0.9.0`

```bash
# Jump into the knative client config directory in the openshift/release
$ cd ci-operator/config/openshift/knative-client

# Copy over the nightly builds config to a release specific config
$ cp openshift-knative-client-release-next.yaml openshift-knative-client-release-v0.9.0.yaml

# Adapt the configuration for a new image name
# - Change .promotion.name to a release specific name (knative-v0.9.0)
$ vi openshift-knative-client-release-v0.9.0.yaml

# Jump to top-level repo directory
$ cd ../../../../

# Call job generator from Docker. You need a local Docker daemon installed
$ docker run -it -v $(pwd)/ci-operator:/ci-operator:z  \
     registry.svc.ci.openshift.org/ci/ci-operator-prowgen:latest \
     --from-dir /ci-operator/config --to-dir /ci-operator/jobs

# Verify the changes
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	ci-operator/config/openshift/knative-client/openshift-knative-client-release-v0.9.0.yaml
	ci-operator/jobs/openshift/knative-client/openshift-knative-client-release-v0.9.0-postsubmits.yaml
	ci-operator/jobs/openshift/knative-client/openshift-knative-client-release-v0.9.0-presubmits.yaml

# Add & Commit all and push to your repo
$ git add ....
$ git commit -a -m "New CI config for knative-client v0.9.0"
$ git push

# Create pull request on https://github.com/openshift/release with your changes
```
