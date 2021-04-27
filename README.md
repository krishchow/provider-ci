# provider-ci

## Input

This action accepts three inputs:

- The GitHub repository for the provider. Ex., `crossplane/provider-aws`
- The commit reference targeted for repackaging. Ex., `v0.17.0`
- The quay.io user which is the destination for the Docker images. Ex., `krishchow`

Additonally, have two configured secrets, which contain:

- `QUAY_TOKEN` - the token for the quay.io robot account.
- `QUAY_USERNAME` - the username for the quay.io robot account.

## Steps

1. Checkout: We first checkout the target provider Github repository.
2. Setup: We then want to set up all of our dependencies, namely:
    1. Go (v1.15)
    2. Other dependencies (`yq` and `operator-sdk`)
    3. Logging into quay.io with our service account.
3. Pull [olm-repackage](https://github.com/redhat-et/olm-repackage): We will clone, and then copy all of the files into the top-level of our repository.
4. Build Operator: Then, all we need to do is run the appropriate steps in our Makefile to build the Operator. Additionally, we need to supply the correct environment variables. During this process we run the code generation, build the Docker image and then push the image.
5. Build Bundle: The final step involves building the bundle for our provider. During this step we:
    1. Generate our `PROJECT` file
    2. Generate our RBAC manifests
    3. Template values into our manifests
    4. Create our ClusterServiceVersion
    5. Build and push the Bundle image

### An example

If we wanted to go through repackaging manually for the `provider-aws`.

1. We checkout our target repository, in this case, that would be `git clone https://github.com/crossplane/provider-aws`.
2. Then, we would need to verify that we have Go, `operator-sdk`, `yq` and Docker or podman. Additonally, we will need credentials setup for a container registry.
3. Next, we will need to clone the olm-repackage repository: `git clone https://github.com/redhat-et/olm-repackage .work`. We then will copy the contents of this `.work` folder into our `provider-aws` folder.
4. Now we can build our OLM operator, but first we need to define our image tag `OPERATOR_IMG="quay.io/krishchow/provider-aws:master"`, then we can run `make docker-build docker-push IMG=$OPERATOR_IMG`.
    1. This will build all of our manifests, run code generation, and build/push the OCI image.
5. The last step involves building the bundle for our operator. We can run `./gen_project.sh > PROJECT` to create our project file, then we will template various field into manifests, by running:
   - `find config \( -type d -name .git -prune \) -o -type f | xargs sed -i "s|IMAGE|$IMAGE|g"`
   - `find config \( -type d -name .git -prune \) -o -type f | xargs sed -i "s|REPO|$REPO|g"`
   - `find config \( -type d -name .git -prune \) -o -type f | xargs sed -i "s|TAG|$TAG|g"`
    Where `IMAGE=provider-aws`, `REPO=crossplane/provider-aws` and `TAG=master`.
6. Then, we need to correctly name the ClusterServiceVersion (csv) by running, `mv config/manifests/bases/IMAGE.clusterserviceversion.yaml config/manifests/bases/$IMAGE.clusterserviceversion.yaml`
7. We can now proceed with create the bundle: `make bundle IMG=$OPERATOR_IMG`, then we can set our bundle image, in this case I selected, `BUNDLE_IMG="quay.io/krishchow/provider-aws-bundle:master"`.
8. Last, we can build and push the OCI image with:
    - `make bundle-build BUNDLE_IMG=$BUNDLE_IMG`
    - `make docker-push IMG=$BUNDLE_IMG`

## Running the Operator

### Operator-SDK CLI

The easiest way to get started with running the operator is through the Operator-SDK CLI tool. Installation instructions can be found in the [Operator-SDK documentation](https://sdk.operatorframework.io/docs/building-operators/golang/installation/).

### Operator Lifecycle Manager

Additionally, if you are not using an OpenShift cluster, then you will need to install the Operator Lifecycle Manager. Installation instructions can be found in the [Operator Lifecycle Manager documentation](https://olm.operatorframework.io/docs/getting-started/), or it can be installed through the Operator-SDK by running `operator-sdk olm install`

### Run

If both of these steps are met, then you can simply run: `operator-sdk run bundle $BUNDLE_IMG`, where `BUNDLE_IMG` is the tag for the bundle image. This will start the process of spinning up the operator, installing the CRDs and setting up all the RBAC resources.
