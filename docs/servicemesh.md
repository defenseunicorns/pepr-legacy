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

### AuthorizationPolicies

The proposed initial set of `AuthorizationPolicy` resources would be effectively mirrors of all `NetworkPolicy` resources. For example, if a `NetworkPolicy` exists to allow pod A in namespace A to talk with pod B in namespace B, this would be duplicated in an `AuthorizationPolicy` as well.

### Make sure new namespaces have IstioInjection

1. Make sure new namespaces have Istio injected via a policy like this: https://kyverno.io/policies/istio/add-sidecar-injection-namespace/add-sidecar-injection-namespace/ 

### mTLS Enforcement

Istio mTLS mode should be configured and default to STRICT for the namespace, with a [`PeerAuthentication`](https://istio.io/latest/docs/reference/config/security/peer_authentication/). Annotations can be added for configuring this to a different mode or setting certain ports to PERMISSIVE. Note the scope/object for these annotations, some can be set on the HelmRelease for more "global" effect, while some are pod specific.

| Annotation | Object | Description |
| -----------| ------ | ----------- |
| `servicemesh.bigbang.dev/mtlsMode`  | HelmRelease | (optional) The mtls mode to set on the namespace (STRICT, PERMISSIVE, DISABLE), defaults to STRICT |
| `servicemesh.bigbang.dev/permissivePorts` | Pod/Deployment (any resource that creates a pod) | (optional) Comma separated list of ports on the given pod to set to PERMISSIVE mTLS mode |

Examples of these annotations:
- `servicemesh.bigbang.dev/mtlsMode=PERMISSIVE`, set on the `foo` HelmRelease would create a namespace-wide `PeerAuthentication` to set PERMISSIVE mTLS mode on all pods that are part of the `foo` HelmRelease
- `servicemesh.bigbang.dev/permissivePorts=9090,9091`, set on deployment A in `foo` HR/namespace would result in a `PeerAuthentication` with a pod selector for all pods in deployment A, which sets ports 9090 and 9091 to PERMISSIVE
