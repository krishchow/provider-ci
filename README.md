# provider-ci

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

## Running the Operator

### Operator-SDK CLI

The easiest way to get started with running the operator is through the Operator-SDK CLI tool. Installation instructions can be found in the [Operator-SDK documentation](https://sdk.operatorframework.io/docs/building-operators/golang/installation/).

### Operator Lifecycle Manager

Additionally, if you are not using an OpenShift cluster, then you will need to install the Operator Lifecycle Manager. Installation instructions can be found in the [Operator Lifecycle Manager documentation](https://olm.operatorframework.io/docs/getting-started/), or it can be installed through the Operator-SDK by running `operator-sdk olm install`

### Run

If both of these steps are met, then you can simply run: `operator-sdk run bundle $BUNDLE_IMG`, where `BUNDLE_IMG` is the tag for the bundle image. This will start the process of spinning up the operator, installing the CRDs and setting up all the RBAC resources.
