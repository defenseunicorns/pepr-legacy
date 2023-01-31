# Contributing to Pepr
First off, thanks so much for wanting to help out! :tada:

## Developer Experience
More to follow soon! 

### Developer Workflow
Placeholder for more details.

## Testing
More to follow soon!

### Code Testing
Placeholder for more details.

## Documentation

### Architecture Decision Records (ADR)

We've chosen to use ADRs to document architecturally significant decisions. We primarily use the guidance found in [this article by Michael Nygard](http://thinkrelevance.com/blog/2011/11/15/documenting-architecture-decisions) with a couple of tweaks:

- The criteria for when an ADR is needed is undefined. The team will decide when the team needs an ADR.
- We will use the tool [adr-tools](https://github.com/npryce/adr-tools) to make it easier on us to create and maintain ADRs.
- We will keep ADRs in the repository under `adr/NNNN-name-of-adr.md`. `adr-tools` is configured with a dotfile to automatically use this directory and format.

### How to use `adr-tools`

```bash
# Create a new ADR titled "Use Bisquick for all waffle making"
adr new Use Bisquick for all waffle making
# Create a new ADR that supersedes a previous one. Let's say, for example, that the previous ADR about Bisquick was ADR number 9.
adr new -s 9 Use scratch ingredients for all waffle making
# Create a new ADR that amends a previous one. Let's say the previous one was ADR number 15
adr new -l "15:Amends:Amended by" Use store-bought butter for all waffle making
# Get full help docs. There are all sorts of other helpful commands that help manage the decision log.
adr help
```