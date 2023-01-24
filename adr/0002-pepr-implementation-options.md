# 2. Pepr Implementation Options

Date: 2023-01-24

## Status

Pending

## Context

In order for Pepr to progress development towards an MVP, we should agree on an approach. We may look back after some implementation and reverse this decision but starting somewhere will allow us to make progress. This decision should be re-evaluated after 3 of the integrations have been worked to a proof of concept (or before if we learn faster).

### Constraints
* Pepr should integrate seamlessly with other tools in the Stack but be functional outside of the Stack
* Pepr should integrate a mission app with core security components (specific components will vary with environment), initially this list is:
	- Istio
	- Logging (EFK)
	- Monitoring (PLG)
	- Kyverno
	- NeuVector
	- Keycloak
* Must Haves:
	- Simplicity
	- Configuration Clarity
	- Auditability
	- Modularity
	- Declarative
	- Government Sponsored
	- Easy integration of packages

* Should Haves:
	- Validation of Config before install
	- Drift/Change Detection
	- Reuse Existing Standards
	- Pieces are individually useful
	- Easy Community Adoption
	- Agnostic

* Could Haves:
	- Use all upstream helm charts
	- Automated everything
	- Support other runtimes
	- Enable users to replace packages with custom alternatives
	- Self-Identify real time security control status

* Won't Haves:
	- Multiple levels of intermediary charts
	- A security tool
	- Treat adopting agencies as second class citizens
	- Assume GitOps
	- Assume single solution
	- Things below k8s 

### Options
#### Option 1: Kyverno Everything
All code is Kyverno policy. Users must interact via annotations. Deployments lose their deterministic value if you utilize other interactions with Kyverno. This also could be extended to have a pre-processor integrated with Kyverno to take a slightly different approach, but still relies entirely on Kyverno. 
* Pros:
	- All done in a singular solution
	- Nothing but yaml, so there is no compiled code
	- Easy for users to customize
	- Could implement a pre-processor to handle things Kyverno doesn't handle natively (or an in cluster operator) to extend capability
	- Reduced technical diversity
* Cons:
	- Loses automatic functionality, because of manual transcribing into Kyverno annotations
	- Needs research to see if it's usable without an in cluster Kyverno operator
	- This is only run-time, dynamic changes. #magic
	- Uncertainty around integrations with tools other than Istio
	- Limited by Kyverno limits (unless using a pre-processor)

#### Option 2: Custom Operator
Complied code that does everything.
* Pros: 
	- Artisinal crafting of logic for every component
	- Unbounded
	- Deployment model is simple
	- Reduced technical diversity
* Cons: 
	- 100% black box magic (unless you sift through 100s of lines of Go)
	- Very complex
		- Hard to build
		- Hard to maintain
	-  Requires in cluster operator

#### Option 3: Choose the right tool for the job
Look at each individual compenent separately and determine the best solution for automating the integration. We should evaluate/discuss each individual components integration as work is attempted.
* Pros:
	- No force feeding (ie no square pegs in round holes) solutions into a selected approach
	- Flexibility to ID most effective way to accomplish individual component goals
	- Likely the quickest path to an MVP
* Cons:
	- Spiraling technical diversity
	- Number of tools needed to learn is high
	- Lots of places to go looking while troubleshooting

### Reccomendation 
We believe that option 3 provides the best path to continue experimentation and understanding of how to best develop this product for the long-term. With this recommendation, a guideline to follow is to see if something already solves the integration with minimal effort.  

## Decision

TBD 

## Consequences

TBD
