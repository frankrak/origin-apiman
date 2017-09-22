# APIMan integration with OpenShift

APIMan provides common management capabilities for API providers
and consumers. You can learn more about the project at its
[home page](http://apiman.io/) or the code at its [github
page](https://github.com/fabric8io/fabric8-ipaas/tree/master/apiman).

Once this integration is deployed, APIs deployed as services on
OpenShift can be exposed externally via the APIMan gateway which
controls API usage. The management policy can be configured via
the APIMan web console, which is available to producers and
consumers of APIs according to their roles in OpenShift projects.

This project supplies the code and configuration for building the
APIMan integration.

* The [deployer](./deployer) is an image that is used to deploy the integration. It
provides templates and configuration for the other components.
* [apiman-builder](./apiman-builder) is a custom builder for OpenShift to build the
APIMan components from source.
* [ElasticSearch](./deployer/common/elasticsearch) is used as configuration and metrics storage.
* [Curator](./deployer/common/curator) expires data in ElasticSearch to save space.

## Building from source

The image for the integration can be built on OpenShift. Create the
builds from the build template:

    $ oc new-app -f hack/dev-builds.yaml

This builds from the standard source repository locations. To build
from a fork or branch, you can supply template parameters to new-app.

ImageStreams for the deployer components are created and the builds
populate them when they complete.

## Deploying the integration

The [deployer README](./deployer/README.md) gives general directions for
deployment. To use images from source builds, supply the `IMAGE_PREFIX`
corresponding to the local registry which can be found in the
ImageStreams. For instance, for the following output:

```
$ oc get is
NAME                   DOCKER REPO                                     TAGS                           UPDATED
apiman-builder         172.30.7.241:5000/apiman/apiman-builder         latest                         18 hours ago
apiman-console         172.30.7.241:5000/apiman/apiman-console         latest                         18 hours ago
apiman-curator         172.30.7.241:5000/apiman/apiman-curator         latest                         18 hours ago
apiman-deployer        172.30.7.241:5000/apiman/apiman-deployer        latest                         18 hours ago
apiman-elasticsearch   172.30.7.241:5000/apiman/apiman-elasticsearch   latest                         18 hours ago
apiman-gateway         172.30.7.241:5000/apiman/apiman-gateway         latest                         18 hours ago
centos                 library/centos                                  6.7,7,latest + 2 more...       18 hours ago
origin                 openshift/origin                                latest,v0.4,v0.5 + 2 more...   18 hours ago
```

... supply the template parameter `IMAGE_PREFIX=172.30.7.241:5000/apiman/`
so that the deployer will run with the built image.

## End-to-end (e2e) testing

Dev builds can be tested with the e2e script. See the [testing
README](./hack/testing/README) for details.

# quick setup steps 
## delete previous builds
oc delete images,builds,bc,imageStream --namespace=openshift -l build=apiman-builder
oc delete images,builds,bc,imageStream --namespace=openshift -l build=apiman-console 
oc delete images,builds,bc,imageStream --namespace=openshift -l build=apiman-deployer
oc delete images,builds,bc,imageStream --namespace=openshift -l app=apiman-elasticsearch
oc delete images,builds,bc,imageStream --namespace=openshift -l build=apiman-elasticsearch
oc delete images,builds,bc,imageStream --namespace=openshift -l build=apiman-gateway
oc delete images,builds,bc,imageStream --namespace=openshift -l build=apiman-curator
oc delete images,builds,bc,imageStream --namespace=openshift -l build=centos 
oc delete images,builds,bc,imageStream --namespace=openshift -l build=origin

## install app in namespace openshift
oc new-app -n openshift -f \data\git\onetrail\origin-apiman\hack\dev-builds.yaml

## get namespace prefix from openshift
oc get is -n openshift
```
NAME                   DOCKER REPO                                         TAGS                             UPDATED
apiman-builder         172.30.35.237:5000/openshift/apiman-builder
apiman-console         172.30.35.237:5000/openshift/apiman-console
....
```
will result in prefix: **172.30.35.237:5000/openshift/**

## creating / updating  missing templates
oc apply -n openshift -f https://raw.githubusercontent.com/openshift/origin-apiman/master/deployer/deployer.yaml

## create service accounts
oc new-app apiman-deployer-account-template

## give correct permissions for apiman-console system account
oadm policy add-cluster-role-to-user cluster-reader system:serviceaccount:**<project name>**:apiman-console

## run deployer with reinstall to remove old components. default is install
oc new-app apiman-deployer-template -n a1 --param IMAGE_PREFIX=**172.30.35.237:5000/openshift/** --PARAM MODE=reinstall
