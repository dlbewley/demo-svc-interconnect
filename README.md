# demo-svc-interconnect

Exploring Red Hat Service Interconnect based on Skupper

This is a smoke test of v1.2.2 operator

# Getting Started


* https://skupper.io/install/index.html
* https://github.com/skupperproject/skupper/releases/download/1.3.0/skupper-cli-1.3.0-mac-amd64.tgz
* https://github.com/skupperproject/skupper-operator

## East Backend

```bash
$ skupper init -n east
Skupper is now installed in namespace 'east'.  Use 'skupper status' to get more information.
$ skupper status -n east
Skupper is enabled for namespace "east" in interior mode. It is not connected to any other sites. It has no exposed services.

$ oc get pods,configmaps,services,deployments -n east
NAME                                              READY   STATUS    RESTARTS   AGE
pod/skupper-router-948d8f5d5-fhdqt                2/2     Running   0          119s
pod/skupper-service-controller-6bb56bcc46-fcp9z   1/1     Running   0          115s

NAME                                 DATA   AGE
configmap/config-service-cabundle    1      10m
configmap/config-trusted-cabundle    1      10m
configmap/kube-root-ca.crt           1      10m
configmap/openshift-service-ca.crt   1      10m
configmap/skupper-internal           1      118s
configmap/skupper-sasl-config        1      2m3s
configmap/skupper-services           0      119s
configmap/skupper-site               12     2m4s

NAME                           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)               AGE
service/skupper                ClusterIP   172.30.132.143   <none>        8081/TCP              118s
service/skupper-router         ClusterIP   172.30.90.60     <none>        55671/TCP,45671/TCP   2m1s
service/skupper-router-local   ClusterIP   172.30.92.30     <none>        5671/TCP              2m1s

NAME                                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/skupper-router               1/1     1            1           2m
deployment.apps/skupper-service-controller   1/1     1            1           116s

$ oc get routes -n east
NAME                   HOST/PORT                                           PATH   SERVICES         PORT           TERMINATION            WILDCARD
claims                 claims-east.apps.hub.lab.bewley.net                        skupper          claims         passthrough/Redirect   None
skupper-edge           skupper-edge-east.apps.hub.lab.bewley.net                  skupper-router   edge           passthrough/None       None
skupper-inter-router   skupper-inter-router-east.apps.hub.lab.bewley.net          skupper-router   inter-router   passthrough/None       None


$ oc new-app --image=quay.io/skupper/hello-world-backend --namespace=east \
                     --name=hello-world-backend --dry-run -o yaml yq '.items[] | split_doc' > ovelays/east/backend.yaml
$ oc apply -k overlays/east/
```

## West Frontend

$ skupper init -n west

$ skupper token create -n west overlays/east/secret.yaml
Token written to overlays/west/secret.yaml

skupper link create -n east overlays/east/secret.yaml
Site configured to link to https://claims-west.apps.hub.lab.bewley.net:443/3f919678-faa4-11ed-8f9a-3e22fb3c2a0b (name=link1)
Check the status of the link using 'skupper link status'.

skupper link status -n east

Links created from this site:
-------------------------------
Link link1 is connected

Current links from other sites that are connected:
----------------------------------------
There are no connected links

skupper status -n west

```bash
$ oc new-app --image=quay.io/skupper/hello-world-frontend --namespace=west \
                     --name=hello-world-frontend --dry-run -o yaml yq '.items[] | split_doc' > overlays/west/frontend.yaml
```

