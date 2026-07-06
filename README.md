# skill-microsoft-foundry

An Agent Skill for deploying, evaluating, and managing **Microsoft Azure AI Foundry** resources and agents end-to-end.

This repository packages Microsoft Foundry expertise as a structured skill (`SKILL.md` + sub-skills and references) that an OpenCode agent can load to carry out Foundry workflows: model discovery and deployment, the full AI-agent development lifecycle, evaluation, tracing, and troubleshooting.

## Overview

The skill is defined by [`SKILL.md`](SKILL.md) and orchestrates a set of specialized sub-skills. Rather than embedding one-off commands, it drives workflows through Azure MCP tools and CLI tooling (`docker`, `az acr`, `az`), and reads workflow-specific reference documents before executing any step.

Each agent's Foundry state is kept under a per-project `.foundry/` workspace, with `agent-metadata.yaml` as the source of truth for environments, project endpoints, agent names, container registry details, and evaluation test cases.

## Capabilities

The skill covers the following areas (see [`SKILL.md`](SKILL.md) for the full routing table):

### Agent lifecycle (`foundry-agent/`)
- **create** — Scaffold new hosted-agent applications from starter samples (Microsoft Agent Framework, LangGraph, or custom frameworks, in Python or C#).
- **deploy** — Containerize a project, build and push images to Azure Container Registry, and create/update/clone/start/stop agent deployments. Supports prompt (LLM-based) and hosted (container-based, ACA and vNext) agents across .NET, Node.js, Python, Go, and Java.
- **invoke** — Send single- or multi-turn messages to an agent for testing and chat.
- **observe** — Evaluate agent quality, run batch evaluations, analyze failures, optimize prompts and agent instructions, compare versions, and set up CI/CD monitoring.
- **trace** — Query traces and analyze latency/failures, correlating evaluation results to specific responses via Application Insights `customEvents`.
- **troubleshoot** — Inspect container logs, query telemetry, and diagnose deployment failures.
- **eval-datasets** — Harvest production traces into evaluation datasets, manage dataset versions and splits, track metrics over time, detect regressions, and maintain lineage from trace to deployment.

### Models (`models/deploy-model/`)
Unified, intent-based model deployment with intelligent routing:
- **preset** — Quick deployments with no customization.
- **customize** — Full control over version, SKU, capacity, and RAI (content filter) policy.
- **capacity** — Discover model availability and capacity across regions, backed by helper scripts (`query_capacity`, `discover_and_rank`) provided in both Bash and PowerShell.

### Infrastructure & governance
- **project/create** — Create a new Azure AI Foundry project for hosting agents and models.
- **resource/create** — Provision an Azure AI Services multi-service (Foundry) resource via the Azure CLI.
- **quota** — Check quota usage, plan capacity, and troubleshoot deployment failures caused by insufficient quota.
- **rbac** — Manage role assignments, managed identities, and service principals for access control and CI/CD setup.

## Repository Structure

```text
SKILL.md                 # Skill entry point and workflow router
foundry-agent/           # Agent lifecycle sub-skills
  create/  deploy/  invoke/  observe/  trace/  troubleshoot/  eval-datasets/
models/deploy-model/     # Model deployment (preset / customize / capacity) + scripts
project/create/          # Create a Foundry project
resource/create/         # Provision a Foundry (AI Services) resource
quota/                   # Quota and capacity management
rbac/                    # RBAC and identity management
references/              # Shared references (metadata contract, setup, auth, SDK)
```

## Installation

This repository is an Agent Skill, not a standalone application. Make it available to your OpenCode agent by placing it where the agent discovers skills:

```bash
git clone https://github.com/doge-liang/skill-microsoft-foundry.git
```

Then point your agent at the cloned directory (or copy it into your agent's skills folder). The agent loads [`SKILL.md`](SKILL.md), whose frontmatter `name` and `description` govern when the skill activates.

### Prerequisites
- Azure CLI (`az`) and an authenticated Azure subscription
- The `azure` MCP server available to the agent
- `docker` and `az acr` for hosted-agent deployment scenarios

## Usage

Once loaded, the skill activates on Foundry-related requests (for example: "deploy my agent to Foundry", "evaluate this agent", "deploy gpt-4o", "check model capacity", "create a Foundry project", "fix a quota error"). The agent then:

1. Reads the matching sub-skill document before executing any workflow.
2. Resolves project context from `.foundry/agent-metadata.yaml` (agent root, environment, project endpoint, agent name).
3. Runs the workflow using Azure MCP tools and CLI commands, with required pre-checks, validation, and status polling.

A typical onboarding flow is `project/create` → `deploy` → `invoke`. Full intent-to-workflow mappings are documented in [`SKILL.md`](SKILL.md).

## Documentation

Start with **[`SKILL.md`](SKILL.md)** for the complete skill definition, sub-skill routing table, agent development lifecycle, the `.foundry` workspace standard, and setup references.

## License

MIT
