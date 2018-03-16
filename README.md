create a dedicated project for socks shop then apply policy changes needed to run socks shop:

```
oc new-project socks-shop
```
# Socks shop pods need full access to Kubernetes API via 'socks-shop' service account

```
oc adm policy add-cluster-role-to-user cluster-admin -z socks-shop
```
# Socks shop pods also need to run as priviliaged containers, so grant 'priviliged' Security Context Constrains (SCC) for 'socks-shop' service account
```
oc adm policy add-scc-to-user privileged -z socks-shop

```
# Socks app has an init daemon that has to run as UID 0, so grant 'anyuid' SCC for 'default' service account

```
oc adm policy add-scc-to-user anyuid -z default
```

# Deploy apps
```
oc apply -f complete-demo.yaml 
```
