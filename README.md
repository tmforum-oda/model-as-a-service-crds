# MoDaaS CRDs — Proposed Contribution to the AI-Native ODA Canvas

> **Status: PROPOSED CONTRIBUTION INPUT — not an adopted standard.**
> These artifacts are submitted for the TM Forum AI-Native Canvas working group to
> evaluate. Nothing here is endorsed or ratified by TM Forum. Where this proposal
> diverges from an existing ADR, the divergence is called out for the working group
> to decide.

## What this is

MoDaaS (Model-as-a-Service) is a governance contract for AI models, tools, and
agents running on the ODA Canvas. It expresses that contract as four
namespace-scoped Kubernetes Custom Resource Definitions plus a discovery Registry,
reconciled by operators.

This folder contains the **open contract only** — the declarative schemas and the
interaction flows. It contains no operator source code.

The contract answers a question the Canvas, a model-serving backend, and an agent
runtime individually do not: which models, tools, and agents is a given agent
allowed to use, are they approved for this data class, and can they all be revoked
at once?

## Contents

| Path | What |
|------|------|
| `crds/` | The four v1beta1 CRD schemas: ModelConfig, ToolConfig, AgentConfig, Registry |
| `LICENSE` | Apache-2.0 |

## Vendor neutrality

The contract is provider-neutral by construction:

- The root spec carries no provider name. `spec.provider` is an open string
  pattern, not a closed enum.
- Provider-specific configuration lives only inside typed sub-blocks (for example
  `spec.awsBedrock`). A new provider adds one typed block and one CEL rule pair.
  No CRD fork, no second CRD.
- Admission-time CEL rules enforce exactly one provider block per resource, so
  multiple provider operators coexist on one cluster.

**Honest disclosure.** The shipped ModelConfig schema defines typed provider blocks
for seven providers: `awsBedrock`, `azureOpenAI`, `googleVertex`, `databricks`,
`nvidiaNIM`, `ollama`, and `custom`. The contract is multi-provider in the schema
itself, not just in principle. What differs today is the *operator*: AWS is the one
provider with a wired reference operator; the other blocks are realized by sibling
operators under the same one-block-plus-CEL convention (no CRD change). The
prominence of AWS in the reference implementation reflects which operator exists
today, not a design preference in the contract.

## Three CRDs, not one — a proposal that diverges from ADR-005

ADR-005 (AI-Native Configuration Resources), currently Proposed, takes a single-CRD
position. MoDaaS deliberately splits the contract into three asset CRDs (Model,
Tool, Agent) plus a Registry. The rationale:

- Models, tools, and agents have different lifecycles, different owners, and
  different governance surfaces. A model is approved for a data class; a tool
  exposes an MCP surface; an agent declares dependencies on models and tools.
  Collapsing them into one resource forces unrelated fields to share a schema and a
  reconciliation loop.
- Independent lifecycle means independent revocation. Pausing a model must
  fail-close every agent that depends on it, which the dependency edges between the
  three CRDs express directly.

This is offered as input to the ADR-005 discussion, not as a replacement decision.
The working group decides.

## What is deliberately not here

The operator implementation (reconciliation logic, IAM/IRSA integration, gateway
data plane), cluster-specific configuration, and provider credentials are the AWS
reference implementation. They are not part of the open contract and are not
published here. Adopters who want a deployable artifact can consume a Helm chart
(CRDs plus RBAC plus an operator Deployment that references a published image),
distributed separately from this contract folder.

## Conformance

A conformance test kit (BDD/Gherkin, aligned to the ODA component CTK L1/L2
convention) is in progress as a joint deliverable with TM Forum. Until it lands,
these artifacts define the contract's schema, semantics, lifecycle state machine,
and admission rules; the CTK will define how an implementation proves it satisfies
them.

## Alignment with existing TMF frameworks

- The Registry projects into TMF639 Resource Inventory as a read-only view, so
  governed assets appear in the Canvas inventory.
- Asset CRDs align to the ODA Component ownership model.
- Governance realizes GB1085 section 2.2 (service functionality) and section 4
  (Canvas Operator Requirements). TMF639/SID references in the schemas are optional
  standards metadata.

## License

Apache-2.0.
