# Pepr

## Overview
Pepr aims to automate the complexities of integrating applications (we'll refer to them as "mission apps") with secure software baseline platforms. Initially it will target some common security services for containerized workloads going into a kubernetes cluster. Specifically, it will target these services and tools but over time extend to support multiple tool options for the various security componenets. For those who are familiar with Department of Defense (DoD) Platform One's [Big Bang](https://github.com/DoD-Platform-One/big-bang), an open source secure platform, you'll recognize this stack. 
- Ingress and Egress Control
  - [Istio](https://istio.io/)
- Logging Stack  
  - [Elasticsearch](https://www.elastic.co/), [Fluentbit](https://fluentbit.io/), [Kibana](https://www.elastic.co/kibana/) (EFK)
  - Future support for [Promtail](https://grafana.com/docs/loki/latest/clients/promtail/?pg=logs&plcmt=tab-5), [Loki](https://grafana.com/oss/loki/), [Grafana](https://grafana.com/grafana/) (PLG)
- Monitoring
  - [Prometheus](https://grafana.com/oss/prometheus/) and [Grafana](https://grafana.com/grafana/)
- Service Mesh
  - [Istio](https://istio.io/)
- Runtime security
  - [NeuVector](https://neuvector.com/)
- Policy Management
  - [Kyverno](https://kyverno.io/)

For those who prefer pictoral description, see below!
![Alt](/docs/pepr_overview.png "Pepr Overview")


## Getting Started
As of 30 Jan 23, Pepr is in the early stages of transitioning from conceptual to implementation. We're experimenting with different proof of concept ideas. To take a look at what has been considered so far, please check out [the Architecture Decision Record (ARD)](./adr/0002-pepr-implementation-options.md). To see some early development, head over to the [Proof of Concept directory](./proof-of-concept/) and check out the Istio (mTLS and ingress for this example) integration proof of concept. 

## Reference Docs
As Pepr continues to grow, feel free to check out the [docs](./docs/) which will describe what integration with each security component means and our [ADRs](./adr/) which will be drafted as we continue exploration (and always submitted as a PR for comment). 

## How to Contribute
Since Pepr is in its early stages, there are lots of details to sort through, including building out a contributing guide. For now, feel free to engage by opening new issues, commenting on existing issues, or commenting on PRs as they come through. Stay tuned for more details! 
