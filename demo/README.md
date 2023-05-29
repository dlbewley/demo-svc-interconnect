<https://access.redhat.com/documentation/en-us/red_hat_application_interconnect/1.1/html/creating_a_service_network_with_kubernetes/index>

This tutorial shows how to connect the following namespaces:

* west - runs the frontend service and is typically a public cluster.
* east - runs the backend service.

First, try this instead of [Skupper Init](#skupper-init)

```bash
oc apply -k https://github.com/dlbewley/demo-metallb/metallb\?ref\=main

CLUSTER=west
oc apply -k base # done, waiting for success
oc apply -k overlays/$CLUSTER
skupper status -n $CLUSTER
Skupper is enabled for namespace "west" in interior mode. It is not connected to any other sites. It has no exposed services.
skupper token create -n $CLUSTER link-secret.yaml

oc apply -k overlays/east
skupper link create -n east link-secret.yaml
```

## Skupper Init

* West cluster (frontend)

```bash
CLUSTER=west
oc apply -k $CLUSTER
skupper init -n $CLUSTER
skupper status -n $CLUSTER
```

* East cluster (backend)
```bash
CLUSTER=east
oc apply -k $CLUSTER
skupper init -n $CLUSTER
skupper status -n $CLUSTER
```

## Skupper Link

* West cluster (frontend)

```bash
CLUSTER=west
skupper token create -n $CLUSTER link-secret.yaml
```

* East cluster (backend)

```bash
CLUSTER=east
skupper link create -n $CLUSTER link-secret.yaml
```

* Status

```bash
CLUSTER=west
skupper status -n $CLUSTER
```

## Application

* West cluster (frontend)

```bash
CLUSTER=west
$ kubectl -n $CLUSTER create deployment hello-world-frontend --image quay.io/skupper/hello-world-frontend
$ kubectl -n $CLUSTER expose deployment hello-world-frontend --port 8080 --type LoadBalancer
$ kubectl -n $CLUSTER get service/frontend

# curl front end and expect:
Trouble! HTTPConnectionPool(host='hello-world-backend', port=8080): Max retries exceeded with url: /api/hello (Caused by NewConnectionError('<urllib3.connection.HTTPConnection object at 0x7fbfcdf0d1d0>: Failed to establish a new connection: [Errno -2] Name or service not known'))
```

To resolve this situation, you must create the backend service and make it available on the service network.

* East cluster (backend)

```bash
CLUSTER=east
$ kubectl -n $CLUSTER create deployment hello-world-backend --image quay.io/skupper/hello-world-backend
$ skupper expose deployment hello-world-backend --port 8080 --protocol tcp -n $CLUSTER
$ skupper status -n $CLUSTER
$ oc get routes -n $CLUSTER
# curl frontend and expect
Hi, <name>. I am Mathematical Machine (backend-77f8f45fc8-mnrdp).
```

## Tips

### Dependencies

* AMQ Streams Operator will be installed

### Alt app deploy

* Howto otherwise create the demo apps

```bash
oc new-app \
  --image=quay.io/skupper/hello-world-frontend \
  --namespace=west \
  --name=hello-world-frontend \
  --dry-run \
  -o yaml yq '.items[] | split_doc' \
  > ovelays/west/fronend.yaml

oc new-app \
  --image=quay.io/skupper/hello-world-backend \
  --namespace=east \
  --name=hello-world-backend \
  --dry-run \
  -o yaml yq '.items[] | split_doc' \
  > ovelays/east/backend.yaml
```
