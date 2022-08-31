# Deploying

## Prerequisites

- [curl](https://curl.se/download.html)
- [pack](https://buildpacks.io/docs/tools/pack/) >= `0.23.0`

## Deployment

### Code Iteration without OCI Images

Use Node directly: 
```
npm install && npm start -- --log-level info
```

To test if it worked: `curl localhost:8080`

### Docker

Build your image and run it using Docker: 

```
pack build node-function --path . --buildpack paketo-buildpacks/nodejs --builder paketobuildpacks/builder:base
```

Where `node-function` is the name of your runnable function image, later used by Docker.

Then run via Docker:

```
docker run -it --rm -p 8080:8080 node-function
```

You do a quick test to see if it worked: `curl localhost:8080`


### Using Tanzu CLI

```
rm -rf node_modules
rm package-lock.json
tanzu apps workload apply -f config/workload.yaml --local-path . --source-image <repository>
tanzu apps workload tail node-function
tanzu apps workload get node-function
```
where `<repository>` is the image repository where your source code is staged before being built

### Using Tilt in a Kubernetes cluster

You may use [tilt](https://github.com/tilt-dev/tilt) `>v0.27.2` in combination with TAP's VS Code plugin to enable live development features including Application Live View and Live Update.

Update the `allow_k8s_contexts` line of the `Tiltfile` to indicate the Kubernetes context to use. 

Update the `Tiltfile` or set the SOURCE_IMAGE environment variable to indicate the registry path where TAP should store your image. 

```
export SOURCE_IMAGE=registry/project/my-function-name
export K8S_TEST_CONTEXT="a-kubernetes-context"
tilt up
tilt down
```


## Deploying to Tanzu Application Platform (TAP)

Using the `config/workload.yaml` it is possible to build, test and deploy this application onto a
Kubernetes cluster that is provisioned with Tanzu Application Platform (https://tanzu.vmware.com/application-platform).

> NOTE: The provided `config/workload.yaml` file uses the Git URL for this sample. When you want to modify the source, you must push the code to your own Git repository and then update the `spec.source.git` information in the `config/workload.yaml` file.


### Deploying to Kubernetes as a TAP workload with Tanzu CLI

You need to select `Include TAP deployment resources` when generating the project for the steps below to work.

When you are done developing your function, you can simply deploy it using:

```
tanzu apps workload apply -f config/workload.yaml
```

If you would like deploy the code from your local working directory you can use the following command:

```
tanzu apps workload create my-python-fn -f config/workload.yaml \
  --local-path . \
  --source-image <REPOSITORY-PREFIX>/my-python-fn-source \
  --type web
```

### Interacting with Tanzu Application Platform

Determine the URL to use for the accessing the app by running:

```
tanzu apps workload get my-python-fn
```

> NOTE: This depends on the TAP installation having DNS configured for the Knative ingress.

After deploying your function, you can interact with the function by using:

> NOTE: Replace the <URL> placeholder with the actual URL.
