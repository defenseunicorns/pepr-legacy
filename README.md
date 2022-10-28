# setup

## Deploy Big Bang

### IronBank Credentials

Create a file at ./bigbang/credentials.yaml that contains your iron bank creds:

```yaml
registryCredentials:
- registry: registry1.dso.mil
  username: runyontr
  password: XXXXXXXXXX
```

### Deploy BigBang

```bash
helm upgrade -i bigbang --create-namespace oci://registry.dso.mil/platform-one/big-bang/bigbang/bigbang --version 1.45.0 \
    -n bigbang \
    -f bigbang/credentials.yaml \
    -f bigbang/loki.yaml \
    -f bigbang/ingress-certs.yaml \
    -f bigbang/kyverno.yaml 
kubectl apply -f rolebinding.yaml # gives kyverno more power
```


### Default Namespace Configuration

By default, each namespace should have some network policies that:
* Prevent Egress traffic
* Prevent Ingress Traffic
* Allow traffic within the namespace
* allow egress UDP traffic out on port 53

And an image pull secret for your iron bank creds:

```bash
$ kubectl apply -f global                               
clusterpolicy.kyverno.io/copy-pull-creds created
clusterpolicy.kyverno.io/add-imagepullsecrets created
clusterpolicy.kyverno.io/add-networkpolicy configured
```
And now if we make a new namespace:

```
$ kubecl create namespace foobar
namespace/foobar created
$ kubectl get networkpolicies -n foobar                 
NAME                   POD-SELECTOR   AGE
default-deny-ingress   <none>         14s
default-deny-egress    <none>         14s
allow-egress-dns       <none>         14s
$ kubectl get secrets -n foobar                                                                                                      
NAME                  TYPE                                  DATA   AGE
default-token-md7gr   kubernetes.io/service-account-token   3      43s
private-registry      kubernetes.io/dockerconfigjson        1      42s
```


### Istio Enablement

When/if this is done via a controller, we could lookup the istio values dynamically, but for now, lets see how istio is deployed

```bash
$ helm get values -n bigbang istio-system-istio > bigbang/istio.yaml
$ helm upgrade -i pepr-istio istio  -n bigbang -f bigbang/istio.yaml
Release "pepr-istio" does not exist. Installing it now.
NAME: pepr-istio
LAST DEPLOYED: Fri Oct 28 11:46:30 2022
NAMESPACE: bigbang
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

### Deploy Podinfo


```bash
$  kubectl apply -f examples/podinfo.yaml 
namespace/podinfo created
gitrepository.source.toolkit.fluxcd.io/podinfo created
helmrelease.helm.toolkit.fluxcd.io/podinfo created
```

You'll note there's a lot of stuff that shows up in the podinfo namespace:

```bash
$ kubectl get vs -n podinfo        
NAME              GATEWAYS                  HOSTS                     AGE
podinfo-autogen   ["istio-system/public"]   ["podinfo.bigbang.dev"]   42s
$ kubectl  get vs -n podinfo podinfo-autogen -o yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  creationTimestamp: "2022-10-28T17:03:56Z"
  generation: 1
  labels:
    app.kubernetes.io/managed-by: kyverno
    kyverno.io/background-gen-rule: expose-service
    kyverno.io/generated-by-kind: HelmRelease
    kyverno.io/generated-by-name: podinfo
    kyverno.io/generated-by-namespace: podinfo
    policy.kyverno.io/gr-name: ur-l2vtr
    policy.kyverno.io/policy-name: virtualservice
    policy.kyverno.io/synchronize: enable
    somekey: somevalue
  name: podinfo-autogen
  namespace: podinfo
  resourceVersion: "741146"
  uid: 4ebfe826-8335-4529-b5e9-905b803b36dc
spec:
  gateways:
  - istio-system/public
  hosts:
  - podinfo.bigbang.dev
  http:
  - route:
    - destination:
        host: podinfo-podinfo.podinfo.svc.cluster.local
        port:
          number: 9898
$ kubectl get networkpolicies -n podinfo                               
NAME                   POD-SELECTOR   AGE
default-deny-ingress   <none>         57s
istiod-egress          <none>         57s
default-deny-egress    <none>         57s
allow-egress-dns       <none>         57s
istio-ingress          <none>         57s
```

Note the annotations on the pod info `HelmRelease` object that faciliate how Pepr deploys istio

```bash
$ kubectl get hr -n podinfo podinfo -o yaml | head -n 15
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"helm.toolkit.fluxcd.io/v2beta1","kind":"HelmRelease","metadata":{"annotations":{"servicemesh.bigbang.dev/expose":"podinfo-podinfo","servicemesh.bigbang.dev/host":"podinfo","servicemesh.bigbang.dev/port":"9898"},"name":"podinfo","namespace":"podinfo"},"spec":{"chart":{"spec":{"chart":"charts/podinfo","sourceRef":{"kind":"GitRepository","name":"podinfo"}}},"interval":"1m0s","targetNamespace":"podinfo","values":{"image":{"repository":"registry.dso.mil/platform-one/big-bang/apps/sandbox/podinfo/podinfo"},"istio":{"enabled":true},"logLevel":"trace","podAnnotations":{"kiali.io/dashboards":"go,envoy","kiali.io/runtimes":"go,envoy"},"redis":{"enabled":false,"repository":"registry1.dso.mil/ironbank/bitnami/redis"},"replicaCount":1,"serviceMonitor":{"enabled":true,"interval":"30s"}}}}
    servicemesh.bigbang.dev/expose: podinfo-podinfo
    servicemesh.bigbang.dev/host: podinfo
    servicemesh.bigbang.dev/port: "9898"
  creationTimestamp: "2022-10-28T17:03:55Z"
  finalizers:
  - finalizers.fluxcd.io
  generation: 1
  name: podinfo
  namespace: podinfo
```


| Annotation | Description |
| --- | ---|
| servicemesh.bigbang.dev/expose| Name of the service to expose |
| servicemesh.bigbang.dev/port| Port exposed on the virtual service |
| servicemesh.bigbang.dev/host | host name that goes as a subdomain under the default domain configured via istio |
