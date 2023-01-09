# ServiceMesh

## Integration with ServiceMesh

### VirtualService

A VirtualService needs to be created to expose services within a package.  Current implementation monitors the HelmRelease that the application is deployed with and creates one based on annotations on the HelmRelease:


| Annotation | Object | Description |
| -----------| ------ | ----------- |
| `servicemesh.bigbang.dev/expose`  | HelmRelease | The name of the service to expose with a virtual service |
| `servicemesh.bigbang.dev/host` | HelmRelease | the subdomain for the virtual service.  Get's prefixed to the domain that BigBang is deployed with |
| `servicemesh.bigbang.dev/port` | HelmRelease | the port on the service to expose |
   
Future implementations should also monitor the [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) objects created.

When a `VirtualService` is created, a `NetworkPolicy` is also required to allow communication from the ingress pod in the istiosystem namespace to talk to the pods being exposed by the VirtualService.  This is currently done via [This NetworkPolicy](../istio/templates/networkpolicy-ingress.yaml)


### NetworkPolicies

1. In order for sidecars to talk to istiod, there needs to be a network policy allowing commmincation from the namespace (and all pods) to the istiod pod.  It's implemented [here](../istio/templates/networkpolicy-istiod.yaml)


### Make sure new namespaces have IstioInjection

1. Make sure new namespaces have Istio injected via a policy like this: https://kyverno.io/policies/istio/add-sidecar-injection-namespace/add-sidecar-injection-namespace/ 

### mTLS Enforcement

Istio mTLS mode should be configured and default to STRICT for the namespace, with a [`PeerAuthentication`](https://istio.io/latest/docs/reference/config/security/peer_authentication/). Annotations can be added for configuring this to a different mode or setting certain ports to PERMISSIVE.

| Annotation | Object | Description |
| -----------| ------ | ----------- |
| `servicemesh.bigbang.dev/mtlsMode`  | HelmRelease | (optional) The mtls mode to set on the namespace (STRICT, PERMISSIVE, DISABLE), defaults to STRICT |
| `servicemesh.bigbang.dev/permissivePorts` | Pod | (optional) Comma separated list of ports on the given pod to set to PERMISSIVE mTLS mode |
