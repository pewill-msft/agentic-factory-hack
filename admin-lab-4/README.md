# Admin Lab 4: Identity, Security & Publishing

[← Admin Lab 3 — Control Plane](../admin-lab-3/README.md) | **Admin Lab 4** | ← Final Lab

This capstone lab ties together identity, security, and publishing. You'll trace the identity journey of an agent — from the **shared project identity** it uses during development, through **tool access security**, to the **dedicated identity** it receives after publishing to production. Along the way, you'll inspect service principals in **Entra ID** to see exactly how Foundry connects to Azure resources.

**Expected duration**: 40 min

**Prerequisites**:

- [Admin Lab 0](../admin-lab-0/README.md) completed (resource access verified)
- Helpful (but not required): [Admin Lab 2](../admin-lab-2/README.md), which created an agent you can publish

## 🎯 Objective

- Understand the agent identity model — project managed identity vs. published agent identity.
- Find and inspect the Foundry project's service principal in Entra ID.
- Review IAM role assignments that connect Foundry to dependent Azure resources.
- Understand how to secure tool and connection access for agents.
- Publish an agent and observe the identity change in Entra ID.
- Test a published agent endpoint and manage published versions.

## 🧭 Context and Background

### The Agent Identity Journey

An agent's identity determines **what resources it can access** and **under whose authority it acts**. This changes as the agent moves from development to production:

```
Development (Unpublished)              Production (Published)
┌─────────────────────┐                ┌─────────────────────┐
│  Agent A             │                │  Agent A (published) │
│  Agent B             │  ──Publish──►  │                     │
│  Agent C             │                │  Dedicated identity  │
│                      │                │  (new service        │
│  Shared project      │                │   principal in Entra)│
│  managed identity    │                └─────────────────────┘
│  (one SP for all)    │
└─────────────────────┘
```

| Phase | Identity Used | Who Controls Access |
|-------|--------------|-------------------|
| **Development** | Project's **managed identity** (system-assigned) — shared by all agents in the project | Platform admin configures IAM roles on the project |
| **Published** | A **dedicated service principal** created for this specific agent | Admin configures permissions for the published agent independently |
| **On-Behalf-Of (OBO)** | Agent acts under the **end user's identity** — using delegated permissions | Requires custom Entra app registration and consent flow |

> [!NOTE]
> "SP" = Service Principal — the identity object in Microsoft Entra ID (formerly Azure Active Directory) that represents an application or service.

### Why This Matters

- **During development**, all agents in the project share the same identity. If one agent can access Cosmos DB, they all can. This is convenient but not suitable for production isolation.
- **After publishing**, the agent gets its own identity. You can grant it only the specific permissions it needs — following the **principle of least privilege**.
- **OBO (On-Behalf-Of)** is important when the agent needs to access resources **as the end user** — for example, if different users should see different data based on their own permissions.

**Useful references:**
- [Authentication and Authorization in Foundry](https://learn.microsoft.com/en-us/azure/foundry/concepts/authentication-authorization-foundry)
- [Agent Identity](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/agent-identity)
- [Publish an Agent](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/publish-agent)

---

## ✅ Tasks

## Part A — Identity & Security

### Task 1: Find the Foundry Project Identity in Entra ID

Every AI Foundry project has a **system-assigned managed identity** — this is the service principal that agents use to authenticate when calling tools and accessing resources during development.

1. Open the **Entra admin center** at [entra.microsoft.com](https://entra.microsoft.com) in your browser.
2. In the left sidebar, navigate to **Identity** → **Applications** → **Enterprise applications**.
3. In the search/filter bar, search for your AI Foundry project name or resource group name.
   - The managed identity typically has the same name as your AI Foundry resource.
   - Set the **Application type** filter to "Managed Identities" to narrow the results.
4. Click on the service principal to open its details.
5. Review the following:

| Field | What It Shows |
|-------|-------------|
| **Display name** | The name of your AI Foundry resource |
| **Application ID** | The unique client ID for this identity |
| **Object ID** | The directory object ID (used in IAM role assignments) |
| **Managed identity type** | System-assigned |
| **Permissions** | What API permissions have been granted |

6. Click on **Permissions** in the left sidebar to see what APIs this identity has been granted access to.

**💬 What to observe:**
- This single service principal is used by **all agents** in the project. When Agent A calls a tool that accesses Cosmos DB, it authenticates as this identity.
- The Object ID you see here corresponds to the principal entries in IAM role assignments — you'll verify this in Task 2.

**✅ Expected result**

The Entra enterprise application page showing the managed identity for your AI Foundry project.

![Entra Managed Identity](./images/entra-managed-identity.png)

> [!TIP]
> If you can't find the managed identity, try the Azure portal instead: navigate to your **AI Foundry resource** → **Identity** (left sidebar) → **System assigned** tab. This shows the Object ID, which you can then look up in Entra.

### Task 2: Review IAM Role Assignments

The managed identity needs specific roles on dependent resources to function. Let's trace the permission chain.

1. In the **Azure portal** at [portal.azure.com](https://portal.azure.com), navigate to your **AI Foundry resource**.
2. Click **Access control (IAM)** in the left sidebar.
3. Click the **Role assignments** tab.
4. Look for entries where the **principal** matches the Object ID from Task 1 (the managed identity).

5. Now check IAM on dependent resources. Navigate to each and check **Access control (IAM)** → **Role assignments**:

| Resource | Expected Role for Managed Identity | Why |
|----------|-----------------------------------|-----|
| **Storage Account** | Storage Blob Data Contributor (or similar) | Agents access knowledge base files |
| **AI Search** | Search Index Data Reader (or similar) | Agents query the search index |
| **Application Insights** | Monitoring Metrics Publisher (or similar) | Agents send telemetry/traces |

6. For each resource, confirm the managed identity's Object ID appears in the role assignments.

**💬 What to observe:**
- The managed identity needs **specific roles on each connected resource**. If a role is missing, the agent will get a permission error when trying to access that resource.
- In production, you'd want to follow the **principle of least privilege** — grant only the minimum roles needed. For example, "Storage Blob Data Reader" if agents only need to read files, not write them.

**✅ Expected result**

The IAM blade showing role assignments on the AI Foundry resource, with the managed identity visible.

![IAM Role Assignments](./images/iam-role-assignments.png)

### Task 3: Secure Tool and Connection Access

Agents in Foundry access external resources through **connections**. Each connection represents a link to a service (Storage, AI Search, Cosmos DB, etc.). Let's review how access to these connections is controlled.

1. In the **Foundry Portal**, navigate to your project.
2. Look for **Connections** or **Connected resources** (typically under **Operate** → **Project settings** or **Assets**).
3. Review the list of connections:

| Connection | Type | What It Accesses |
|-----------|------|-----------------|
| Storage | Azure Blob Storage | Knowledge base files, training data |
| AI Search | Azure AI Search | Search indexes for RAG |
| (Others) | Various | Cosmos DB, API Management, etc. |

4. Click on a connection to examine its configuration:
   - **Authentication type** — API key, managed identity, or service principal?
   - **Access scope** — Who can use this connection? All agents in the project, or restricted to specific agents?

5. Consider the security implications:

**💬 Things to think about:**
- If a connection uses an **API key**, that key is shared by all agents in the project. Rotating the key affects all agents simultaneously.
- **Managed identity** authentication is generally preferred — no secrets to manage, and permissions are controlled via IAM roles (which you reviewed in Task 2).
- In production, you may want to **restrict which agents can use which connections** — for example, a diagnostic agent shouldn't need access to the parts ordering API.

> [!IMPORTANT]
> Connection-level access control is critical for production security. An agent with access to a write-enabled Cosmos DB connection could potentially modify factory data. Ensure agents only have connections to the resources they need, with the minimum required permissions.

**✅ Expected result**

The connections page showing linked resources and their authentication types.

![Connections](./images/connections.png)

---

## Part B — Publishing & Post-Publish Identity

### Task 4: Create and Publish an Agent

Let's publish an agent and observe what happens to its identity.

1. First, ensure you have an agent to publish. Either:
   - Use the **Contoso Tires Maintenance Advisor** from [Admin Lab 2](../admin-lab-2/README.md) (if you completed it), or
   - Create a new simple agent: **Build** → **Agents** → **+ New agent** with:

| Setting | Value |
|---------|-------|
| **Agent name** | `Contoso Tires Advisor` |
| **Model** | `gpt-4.1` |
| **Instructions** | `You are a maintenance advisor for Contoso Tires. Help technicians diagnose faults and plan repairs. Reference specific thresholds and part numbers.` |

2. Navigate to the agent's detail page.
3. Look for a **Publish** button or option.
4. Walk through the publish workflow:
   - Review the publishing configuration — endpoint URL, authentication settings
   - Confirm the publish operation

> [!NOTE]
> Publishing creates a **production-ready endpoint** for the agent with its own identity and authentication. The development version of the agent continues to exist in the project for iteration.

**✅ Expected result**

The agent shows a "Published" status with an endpoint URL.

![Agent Published](./images/agent-published.png)

### Task 5: Review the Published Agent's Identity in Entra

This is the key identity moment — let's see what happened in Entra ID after publishing.

1. Return to [entra.microsoft.com](https://entra.microsoft.com) → **Identity** → **Applications** → **Enterprise applications**.
2. Search for your published agent name (e.g., "Contoso Tires Advisor" or similar).
3. You should find a **new service principal** — this is the published agent's **dedicated identity**.
4. Open this service principal and review:

| Field | Project Identity (Task 1) | Published Agent Identity (New) |
|-------|--------------------------|-------------------------------|
| **Display name** | AI Foundry project name | Published agent name |
| **Type** | Managed Identity | Application |
| **Scope** | Shared by all agents in project | Dedicated to this published agent |
| **Permissions** | Broad project-level access | Can be restricted to specific resources |

5. Compare the two service principals:
   - The **project identity** serves all development agents — broad permissions.
   - The **published agent identity** is dedicated — you can grant it only what this specific agent needs.

**💬 This is the identity journey:**
1. During development → agent uses the **project's managed identity** (shared)
2. After publishing → agent receives a **dedicated service principal** (isolated)
3. Optionally → configure **OBO** so the agent acts under the **end user's identity** (delegated)

> [!TIP]
> In production, review the published agent's identity carefully. Grant it the minimum IAM roles needed for its specific tools and connections. This ensures that even if one agent is compromised, it can't access resources outside its scope.

**✅ Expected result**

Two service principals visible in Entra — the project-level managed identity and the published agent's dedicated identity.

![Entra Published Identity](./images/entra-published-identity.png)

### Task 6: Manage Published Versions

1. Back in the **Foundry Portal**, navigate to your published agent.
2. Look for a **Versions** or **Publishing history** section.
3. Review the version information:
   - **Version number** — increments with each publish
   - **Published date** — when this version was deployed
   - **Status** — Active, Previous, or Rolled back
4. If you make a change to the agent (e.g., update the instructions) and publish again, a **new version** is created.
5. Explore the **rollback** capability — can you revert to a previous version?

**💬 What to observe:**
- Version management is essential for production safety. If a new version of the agent introduces a regression (e.g., gives incorrect maintenance advice), you want to roll back quickly.
- Each version is a snapshot of the agent's configuration at publish time — instructions, model, tools, and settings.

**✅ Expected result**

The publishing history showing version numbers, dates, and status.

![Published Versions](./images/published-versions.png)

### Task 7: Test the Published Endpoint

Finally, let's invoke the published agent through its production endpoint.

1. From the published agent's page, copy the **endpoint URL**.
2. Note the **authentication method** — typically an API key or OAuth token.
3. Test the endpoint using one of these methods:

**Option A — Using the browser/portal test tool** (if available in the published agent's page):
   - Look for a "Test" or "Try it" button
   - Send a simple prompt: `What should I check if curing press temperature reads 179°C?`

**Option B — Using curl** (from a terminal or Codespace):

```bash
curl -X POST "<your-published-endpoint-url>" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <your-api-key>" \
  -d '{
    "messages": [
      {"role": "user", "content": "What should I check if curing press temperature reads 179°C?"}
    ]
  }'
```

4. Verify you get a meaningful response from the published agent.

**✅ Expected result**

A successful response from the published agent endpoint with manufacturing-specific guidance.

![Endpoint Test](./images/endpoint-test.png)

> [!WARNING]
> The API key for a published agent is a **production credential**. In a real deployment, store it in Azure Key Vault, not in code or environment variables. Rotate keys regularly and monitor for unauthorized usage.

## 🚀 Go Further

- **On-Behalf-Of (OBO) flow**: Discuss with your team how you'd configure an agent to execute under the end user's identity. This requires a custom Entra app registration with delegated permissions and a consent flow. When would this be necessary? (Answer: when different users should have different data access levels through the same agent.)
- **Approval workflows**: For a production deployment, how would you implement an approval step before publishing? Consider using Azure DevOps or GitHub Actions to gate the publish action behind a code review and approval.
- **A/B testing**: Deploy two versions of the same agent and route a percentage of traffic to each. Compare metrics from [Admin Lab 2](../admin-lab-2/README.md) to determine which version performs better.
- **Least-privilege audit**: Review all IAM role assignments from Task 2. For each role, determine if a more restrictive role would work. Document a "least privilege" configuration for your production agents.

## 🧠 Conclusion

You've traced the complete identity journey of a Foundry agent:

**Part A — Identity & Security:**
- Found the project's managed identity in Entra ID and understood its shared scope
- Traced IAM role assignments from the managed identity to dependent resources
- Reviewed connection-level access control and tool security

**Part B — Publishing & Identity:**
- Published an agent and observed the creation of a dedicated service principal
- Compared project-level vs. published agent identities in Entra
- Managed published versions with rollback capability
- Tested the published agent's production endpoint

### The Big Picture

```
Admin Lab 0    →  Environment foundations
Admin Lab 1    →  Model lifecycle (versioning + fine-tuning)
Admin Lab 2    →  Quality assurance (evaluations + observability)
Admin Lab 3    →  Fleet governance (control plane + quotas)
Admin Lab 4    →  Production readiness (identity + security + publishing)
```

You now have the knowledge to manage AI models and agents across their full lifecycle — from deployment and customization, through monitoring and evaluation, to secure production publishing.

← [Admin Lab 3 — Control Plane](../admin-lab-3/README.md)
