# Admin Lab 3: Control Plane

[← Admin Lab 2 — Evaluations & Observability](../admin-lab-2/README.md) | **Admin Lab 3** | [Admin Lab 4 — Identity, Security & Publishing →](../admin-lab-4/README.md)

This lab explores the **Management Center** — the fleet-level control plane in Azure AI Foundry. You'll learn how administrators manage resources, agents, and quotas across projects from a single pane of glass.

**Expected duration**: 30 min

**Prerequisites**:

- [Admin Lab 0](../admin-lab-0/README.md) completed (resource access verified)
- Helpful (but not required): [Admin Lab 2](../admin-lab-2/README.md), which created an agent with monitoring data

## 🎯 Objective

- Navigate the Management Center and understand the organizational hierarchy (hubs → projects → resources).
- Explore fleet-level monitoring and cross-project dashboards.
- Manage agent lifecycle from the control plane — view agents, versions, and status.
- Review quotas, rate limits, and capacity planning.

## 🧭 Context and Background

### From Per-Agent to Fleet-Level

In [Admin Lab 2](../admin-lab-2/README.md), you explored monitoring for a **single agent** — token usage, traces, and evaluations. That's the right view for a developer or a specific maintenance team.

But as a **product owner or platform administrator**, you need a broader view:
- How many agents are running across all projects?
- Which projects consume the most tokens?
- Are we approaching our quota limits?
- Which agents need attention (errors, version deprecation)?

This is where the **Management Center** (Control Plane) comes in.

### The Organizational Hierarchy

Azure AI Foundry organizes resources in a hub-and-spoke model:

```
Azure Subscription
└── Resource Group
    └── AI Foundry Hub
        ├── Project A (Contoso Tires Maintenance)
        │   ├── Model deployments (gpt-4.1, gpt-4o-mini, ...)
        │   ├── Agents (Maintenance Advisor, ...)
        │   └── Connections (Storage, AI Search, ...)
        ├── Project B (could be another team/use case)
        └── Shared resources (compute, storage, quotas)
```

| Level | Who Manages It | What They See |
|-------|---------------|---------------|
| **Subscription** | IT / Cloud admin | Cost, compliance, policies |
| **Hub** | Platform admin / Product owner | Shared resources, quotas, cross-project metrics |
| **Project** | Developer / Team lead | Models, agents, tools, evaluations |

> [!NOTE]
> In this workshop you likely have one hub with one project. In production environments, a hub typically serves multiple projects for different teams or use cases. The Management Center is designed for that multi-project reality.

**Useful references:**
- [Control Plane overview](https://learn.microsoft.com/en-us/azure/foundry/control-plane/overview)
- [Monitoring across the fleet](https://learn.microsoft.com/en-us/azure/foundry/control-plane/monitoring-across-fleet)
- [Managing agents](https://learn.microsoft.com/en-us/azure/foundry/control-plane/how-to-manage-agents)

---

## ✅ Tasks

### Task 1: Navigate the Management Center

1. Open the **Foundry Portal** at [ai.azure.com](https://ai.azure.com).
2. Click **Operate** in the top navigation bar.
3. Look for **Management Center** in the left sidebar (it may also appear as **Management** or under a **Govern** section).
4. Explore the Management Center overview:
   - **Hubs and Projects** — see the organizational structure. How many hubs and projects exist in your environment?
   - **Resource overview** — a summary of connected resources (compute, storage, model deployments)
   - **Navigation** — note the different sections available (Resources, Agents, Quotas, Monitoring, etc.)

**💬 What to observe:**
- The Management Center gives you a **top-down view** — instead of navigating into each project individually, you see everything at once.
- In a production environment with multiple projects (e.g., maintenance agents, quality agents, scheduling agents), this is where you'd get a consolidated view.

**✅ Expected result**

The Management Center overview page showing hubs, projects, and connected resources.

![Management Center Overview](./images/management-center-overview.png)

### Task 2: Explore Fleet-Level Monitoring

1. In the Management Center, look for a **Monitoring** or **Metrics** section.
2. Explore the fleet-level dashboards — these aggregate data across all projects and agents:

| Dashboard Metric | What It Shows |
|-----------------|--------------|
| **Total agent runs** | Combined run count across all agents in all projects |
| **Token usage by project** | Which projects consume the most tokens — helps with cost allocation |
| **Error rate trends** | Cross-project error trends — helps identify systemic issues |
| **Active agents** | How many agents are deployed and running |

3. Compare this view with the per-agent monitoring you explored in [Admin Lab 2](../admin-lab-2/README.md).

**💬 What to notice:**
- Fleet-level monitoring answers **"how is the platform performing?"** while per-agent monitoring answers **"how is this specific agent performing?"**
- Token usage by project is particularly useful for **cost allocation** — you can charge back to teams based on actual usage.
- Error rate trends at the fleet level can reveal systemic issues (e.g., a model deprecation affecting multiple agents simultaneously).

> [!TIP]
> If you see limited data in the dashboards, that's expected — we only have one project with a few test conversations. In production, these dashboards become essential for capacity planning and cost management.

**✅ Expected result**

Fleet-level monitoring showing aggregate metrics across projects.

![Fleet Monitoring](./images/fleet-monitoring.png)

### Task 3: Manage Agents from the Control Plane

1. In the Management Center, look for an **Agents** section.
2. Review the list of agents across your project(s). You should see:
   - The **Contoso Tires Maintenance Advisor** you created in [Admin Lab 2](../admin-lab-2/README.md) (if completed)
   - Any other agents created during the workshop

3. For each agent, note the available information:

| Field | Description |
|-------|-------------|
| **Agent name** | The display name you gave the agent |
| **Project** | Which project the agent belongs to |
| **Model** | Which model deployment the agent uses |
| **Version** | The agent's version number (increments with changes) |
| **Status** | Active, Paused, or Deleted |
| **Last active** | When the agent was last invoked |

4. Explore the lifecycle operations available:
   - Can you **pause** an agent from here?
   - Can you **delete** an agent?
   - Can you see the agent's **version history**?

**💬 What to observe:**
- Managing agents from the control plane lets you perform bulk operations — for example, pausing all agents that use a model version that's about to be deprecated.
- Version tracking helps you understand which agents have been updated recently and which may be running outdated configurations.

> [!IMPORTANT]
> In production, **do not delete agents without checking** if they're actively serving users. Use the "Last active" timestamp and run count to determine if an agent is still in use.

**✅ Expected result**

The agent list showing agents across projects with their status, model, and version.

![Agent Management](./images/agent-management.png)

### Task 4: Review Quotas and Capacity

1. In the Management Center, look for a **Quotas** or **Capacity** section.
2. Review the quota allocations for your model deployments:

| Quota Metric | Description |
|-------------|-------------|
| **Tokens per Minute (TPM)** | Maximum tokens the deployment can process per minute |
| **Requests per Minute (RPM)** | Maximum API calls per minute |
| **Allocated vs. Used** | How much of your quota is currently consumed |
| **Region availability** | Which regions have capacity for your model |

3. Check the quotas for each of your model deployments (gpt-4.1, gpt-4o-mini, text-embedding-3-large).
4. Consider these questions:
   - If you deployed three agents all using gpt-4.1, how would the TPM quota be shared among them?
   - What happens when you hit the RPM limit? (Answer: requests get throttled with HTTP 429 responses)
   - If your token usage is trending upward, how would you request more capacity?

**💬 What to think about:**
- Quotas are **shared across all agents** using the same model deployment. If agent A uses 80% of the TPM quota, agents B and C only have 20% to share.
- For production, consider deploying the same model multiple times with separate quotas for different teams or priority levels — critical maintenance diagnostic agents could get a higher-quota deployment than general Q&A agents.
- **Rate limit planning** is essential before scaling to production. A single test agent uses minimal quota, but 50 concurrent maintenance technicians querying the system will need significantly more.

**✅ Expected result**

The quotas view showing TPM and RPM allocations per model deployment.

![Quotas and Capacity](./images/quotas-capacity.png)

## 🚀 Go Further

- **Set up quota alerts**: In Azure Monitor, create an alert that triggers when token usage exceeds 80% of the allocated TPM for any deployment.
- **Explore Azure Policy**: Investigate how Azure Policy can enforce governance rules — for example, requiring that all model deployments use version pinning, or restricting which models can be deployed in a project.
- **Cost estimation**: Using the token usage data from the monitoring dashboards, estimate the monthly cost for running your maintenance advisor agent at production scale (e.g., 100 queries per hour, 8 hours per day).
- **Multi-project scenario**: Discuss with your team how you'd organize a production environment — separate projects for maintenance, quality, and scheduling agents? Or one project with multiple agents?

## 🧠 Conclusion

You've explored the Management Center — the admin's single pane of glass:

- Navigated the hub-and-spoke organizational hierarchy
- Viewed fleet-level monitoring and cross-project dashboards
- Managed agent lifecycle from the control plane — status, versions, operations
- Reviewed quotas and capacity planning for model deployments

**Next**: [Admin Lab 4 — Identity, Security & Publishing](../admin-lab-4/README.md)
