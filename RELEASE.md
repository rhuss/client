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
# Usage: ./create-release-branch.sh <upstream-tag> <downstream-release-branch>
# <upstream-tag>: The tag referring the upstream release
# <downstream-release-branch>: Name of the release branch to create
$ ./create-release-branch.sh v0.9.0 release-v0.9.0

# Push release branch to openshift/knative-client repo
$ git push openshift release-v0.9.0
```

### Create a ci-operator configuration and Prow job configurations

* Create a fork of https://github.com/openshift/release (if not already done)
* Create a new config YML for your release. Copy over `release-next.yaml` and adapt it by updating the image name from `knative-nightly` to a version specific name `knative-v0.9.0`
* Once the CI passes for the the release branch and is ready to go for QA, push the tag for the release for in the format `openshift-`<version>, e.g. `openshift-v0.9.0`
*
```bash
# Jump into the knative client config directory in the openshift/release
$ cd ci-operator/config/openshift/knative-client

# Copy over the nightly builds config to a release specific config with
# the name of the yaml file ends with the new release branch name (e.g. release-v0.9.0)
$ cp openshift-knative-client-release-next.yaml openshift-knative-client-release-v0.9.0.yaml

# Adapt the configuration for a new image name
# - Change .promotion.name to a release specific name (knative-v0.9.0)
$ vi openshift-knative-client-release-v0.9.0.yaml

# Jump to top-level repo directory
$ cd ../../../../

# Call Prow job generator via Docker. You need a local Docker daemon installed
$ docker run -it -v $(pwd)/ci-operator:/ci-operator:z  \
     registry.svc.ci.openshift.org/ci/ci-operator-prowgen:latest \
     --from-dir /ci-operator/config --to-dir /ci-operator/jobs

# Add the image mirroring settings
$ vi core-services/image-mirroring/knative/mapping_knative_v0_9_quay

# Add a line for the kn image like
# registry.svc.ci.openshift.org/openshift/knative-v0.9.0:knative-client quay.io/openshift-knative/knative-client:v0.9.0

# Verify the changes
$ git status
On branch master
Your branch is ahead of 'origin/master' by 180 commits.
  (use "git push" to publish your local commits)

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   core-services/image-mirroring/knative/mapping_knative_v0_9_quay

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	ci-operator/config/openshift/knative-client/openshift-knative-client-release-v0.9.0.yaml
	ci-operator/jobs/openshift/knative-client/openshift-knative-client-release-v0.9.0-postsubmits.yaml
	ci-operator/jobs/openshift/knative-client/openshift-knative-client-release-v0.9.0-presubmits.yaml

# Add & Commit all and push to your repo
$ git add ....
$ git commit -a -m "knative-client release v0.9.0 setup"
$ git push

# Create pull request on https://github.com/openshift/release with your changes

# When the PR is merged and the CI is passes so that we are ready for QA, create tag & push
$ git tag openshift-v0.9.0
$ git push --tags
```
