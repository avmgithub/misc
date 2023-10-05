
## This reference may only work on IBM Cloud ROKS on Classic


Use this command to look at the pvc used by the internal registry
```
oc describe pvc -n openshift-image-registry image-registry-storage
```

See this link to expose a route for the internal registry:
https://cloud.ibm.com/docs/openshift?topic=openshift-registry#route_internal_registry

```
oc get svc -n openshift-image-registry
oc create route reencrypt --service=image-registry -n openshift-image-registry
oc get route image-registry -n openshift-image-registry
```

Edit the route to set the load balancing strategy to source so that the same client IP address reaches the same server, as in a passthrough route setup. You can set the strategy by adding an annotation in the metadata.annotations section: haproxy.router.openshift.io/balance: source. You can edit the configuration file from the Red Hat OpenShift Application Console or in your command line by running the following command.

```
oc edit route image-registry -n openshift-image-registry
```
Add the annotation.

```
apiVersion: route.openshift.io/v1
kind: Route
metadata:
annotations:
    haproxy.router.openshift.io/balance: source
...
```



If corporate network policies prevent access from your local system to public endpoints via proxies or firewalls, allow access to the route subdomain that you create for the internal registry in the following steps.

Log in to the internal registry by using the route as the hostname.

```
docker login -u $(oc whoami) -p $(oc whoami -t) image-registry-openshift-image-registry.<cluster_name>-<ID_string>.<region>.containers.appdomain.cloud
```

Now that you are logged in, try pushing a sample hello-world app to the internal registry.

    Pull the hello-world image from DockerHub, or build an image on your local machine.
```
docker pull hello-world
```

Tag the local image with the hostname of your internal registry, the project that you want to deploy the image to, and the image name and tag.

docker tag hello-world:latest image-registry-openshift-image-registry.<cluster_name>-<ID_string>.<region>.containers.appdomain.cloud/<project>/<image_name>:<tag>

Push the image to your cluster's internal registry.

```
docker push image-registry-openshift-image-registry.<cluster_name>-<ID_string>.<region>.containers.appdomain.cloud/<project>/<image_name>:<tag>
```

Verify that the image is added to the Red Hat OpenShift image stream.
```
    oc get imagestream

    Example output

    NAME          DOCKER REPO                                                            TAGS      UPDATED
    hello-world   image-registry-openshift-image-registry.svc:5000/default/hello-world   latest    7 hours ago
```

To enable deployments in your project to pull images from the internal registry, create an image pull secret in your project that holds the credentials to access your internal registry. Then, add the image pull secret to the default service account for each project.

    List the image pull secrets that the default service account uses, and note the secret that begins with default-dockercfg.

```
oc describe sa default

Example output

...
Image pull secrets:
all-icr-io
default-dockercfg-mpcn4
...
```

Get the encoded secret information from the data field of the configuration file.

```
oc get secret <default-dockercfg-name> -o yaml

Example output

apiVersion: v1
data:
  .dockercfg: ey...=
```

Decode the value of the data field.
```
echo "<ey...=>" | base64 -D

Example output

{"172.21.xxx.xxx:5000":{"username":"serviceaccount","password":"eyJ...
```

Create a new image pull secret for the internal registry.

    secret_name: Give your image pull secret a name, such as internal-registry.
    --namespace: Enter the project to create the image pull secret in, such as default.
    --docker-server: Instead of the internal service IP address (172.21.xxx.xxx:5000), enter the hostname of the image-registry route with the port (image-registry-openshift-image-registry.<cluster_name>-<ID_string>.<region>.containers.appdomain.cloud:5000).
    --docker-username: Copy the "username" from the previous image pull secret, such as serviceaccount.
    --docker-password: Copy the "password" from the previous image pull secret.
    --docker-email: If you have one, enter your Docker email address. If you don't, enter a fictional email address, such as a@b.c. This email is required to create a Kubernetes secret, but is not used after creation.

```
oc create secret image-registry internal-registry --namespace default --docker-server image-registry-openshift-image-registry.<cluster_name>-<ID_string>.<region>.containers.appdomain.cloud:5000 --docker-username serviceaccount --docker-password <eyJ...> --docker-email a@b.c
```

Add the image pull secret to the default service account of your project.

```
        oc patch -n <namespace_name> serviceaccount/default --type='json' -p='[{"op":"add","path":"/imagePullSecrets/-","value":{"name":"<image_pull_secret_name>"}}]'
```
        Repeat these steps for each project that you want to pull images from the internal registry.

Now that you set up the internal registry with an accessible route, you can log in, push, and pull images to the registry. For more information, see the Red Hat OpenShift documentation.


## Creating an image pull secret with different IAM API key credentials
https://cloud.ibm.com/docs/openshift?topic=openshift-registry#other_registry_accounts

## Setting up builds in the internal registry to push images to IBM Cloud Container Registry
https://cloud.ibm.com/docs/openshift?topic=openshift-registry#builds_registry