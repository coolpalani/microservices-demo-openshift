# Sock Shop on OpenShift 
This repository is for running Sock Shop on Openshift. Original Sock Shop kubernetes manifests need full access to Kubernetes API. Therefore we can't Sock Shop on OpenShift using original those without any modifications. However, manifests in this have been modified to run on OpenShift without cluster-admin privilege.

```
$ oc new-project sock-shop
$ oc apply -f complete-demo.yaml
deployment.extensions/carts-db created
service/carts-db created
deployment.extensions/carts created
service/carts created
....

$ oc expose service/front-end
route.route.openshift.io/front-end exposed

$ oc get route
NAME        HOST/PORT                                           PATH      SERVICES    PORT      TERMINATION   WILDCARD
front-end   front-end-sock-shop.apps.xxxxxxxxxxxxxxxxxxxx.com             front-end   web                     None
```

## How to build catalogue-db and user-db
```
$ docker/user-db-mongodb-container/
$ s2i build . centos/mongodb-34-centos7 mosuke5/sockshop-user-db:latest
```

```
$ docker/catalogue-db-mysql-container/
$ s2i build . centos/mysql-57-centos7 mosuke5/sockshop-catalogue-db:latest
```
