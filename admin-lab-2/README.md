# Admin Lab 2: Evaluations & Observability

[← Admin Lab 1 — Model Lifecycle](../admin-lab-1/README.md) | **Admin Lab 2** | [Admin Lab 3 — Control Plane →](../admin-lab-3/README.md)

This lab focuses on **monitoring agent behavior** and **evaluating agent quality** — all through the Foundry Portal and Azure Monitor. You'll create a fresh agent, generate trace data, explore dashboards, and run an evaluation.

**Expected duration**: 35 min

**Prerequisites**:

- [Admin Lab 0](../admin-lab-0/README.md) completed (resource access verified)

> [!NOTE]
> This lab is self-contained. You do **not** need agents from the Portal Labs or Coding Challenges. You'll create a new agent from scratch in Task 1.

## 🎯 Objective

- Create a test agent and generate trace data through sample conversations.
- Explore the agent monitoring dashboard — token usage, run counts, errors.
- Drill into individual agent traces to understand execution details.
- Navigate from Foundry Portal into Application Insights for deeper analysis.
- Set up and run an evaluation with built-in quality metrics.
- Interpret evaluation scores and identify areas for improvement.

## 🧭 Context and Background

### Why Observability Matters for Manufacturing Agents

In a tire manufacturing environment, agent errors can have real consequences:
- A **misdiagnosed fault** could lead to the wrong repair, wasting hours of technician time.
- An **incorrect risk score** could deprioritize a critical machine, leading to unplanned downtime.
- **Inconsistent responses** across shifts could confuse maintenance teams.

Observability gives you visibility into **what agents are doing, how well they're doing it, and where they need improvement**.

### The Observability Stack

Azure AI Foundry provides a layered observability architecture:

| Layer | What It Shows | Where to Find It |
|-------|--------------|-----------------|
| **Agent Monitoring Dashboard** | High-level metrics — token usage, run count, tool calls, errors | Foundry Portal → Agent → Monitor tab |
| **Agent Traces** | Individual execution details — message flow, latency, token counts per step | Foundry Portal → Agent → Traces |
| **Application Insights** | Full telemetry — KQL queries, transaction search, custom dashboards | Azure Portal → Application Insights |
| **Evaluations** | Quality scores — groundedness, relevance, coherence, fluency | Foundry Portal → Evaluations |

> [!TIP]
> If you completed Challenge 3 (coding track), your agents already have trace data in Application Insights. The dashboards in this lab will show that data too. If not, don't worry — you'll generate fresh data in Task 1.

---

## ✅ Tasks

### Task 1: Create a Test Agent and Generate Trace Data

First, let's create a simple maintenance advisor agent and send it some test conversations.

1. In the Foundry Portal at [ai.azure.com](https://ai.azure.com), click **Build** in the top navigation bar.
2. Select **Agents** in the left sidebar, then click **+ New agent**.
3. Configure the agent:

| Setting | Value |
|---------|-------|
| **Agent name** | `Contoso Tires Maintenance Advisor` |
| **Model** | `gpt-4.1` |
| **Instructions** | *(copy the system prompt below)* |

```
You are a maintenance advisor for Contoso Tires, a tire manufacturing facility. You help maintenance technicians diagnose machine faults, recommend repair procedures, and identify required parts.

Key machines in the facility:
- Tire Curing Press (TC-100): Temperature threshold 178°C, cycle time threshold 14 min
- Tire Building Machine (TB-200, TB-300): Drum vibration threshold 3.0 mm/s, ply tension threshold 230 N
- Tire Extruder (C1): Throughput minimum 650 kg/h
- Tire Uniformity Machine (D1): Radial force variation threshold 100 N
- Banbury Mixer (E1): Mixing temperature threshold 160°C, vibration threshold 5.5 mm/s

Always reference specific thresholds, part numbers, and estimated repair times when applicable. Prioritize safety — recommend lock-out/tag-out procedures for any physical maintenance tasks.
```

4. Click **Create** to save the agent.

5. Now send **5–6 test conversations** to generate trace data. Use the agent's chat interface and send each of these prompts as separate conversations:

**Prompt 1 — Fault Diagnosis:**
> Machine TC-100 is showing a temperature of 179.2°C. Diagnose the issue and recommend next steps.

**Prompt 2 — Maintenance Procedure:**
> Walk me through the lock-out/tag-out procedure for the tire building machine TB-200 before bearing replacement.

**Prompt 3 — Parts Lookup:**
> What spare parts do I need for a heating element replacement on the curing press? Include part numbers.

**Prompt 4 — Risk Assessment:**
> The Banbury mixer E1 has vibration at 5.8 mm/s and temperature at 162°C. Both thresholds are breached. What's the priority?

**Prompt 5 — Scheduling:**
> Machine D1 has a risk score of 82. Should this be scheduled as routine maintenance or treated as urgent?

**Prompt 6 — Multi-machine Triage:**
> We have simultaneous warnings on TC-100 (temperature) and TB-300 (vibration). How should we prioritize?

> [!NOTE]
> It may take **2–5 minutes** for trace data to appear in the dashboards after you send your conversations. This is normal — telemetry is batched and processed asynchronously.

**✅ Expected result**

The agent responds to each prompt with manufacturing-specific guidance. Each conversation generates trace data.

![Agent Created](./images/agent-created.png)

### Task 2: Explore the Agent Monitoring Dashboard

1. In the Foundry Portal, navigate to your **Contoso Tires Maintenance Advisor** agent.
2. Click the **Monitor** tab (or look for monitoring/analytics in the agent's detail page).
3. Review the dashboard metrics:

| Metric | What to Look For |
|--------|-----------------|
| **Agent runs** | Total count of conversations — should match the 6 you sent |
| **Token usage** | Total tokens consumed (prompt + completion) — gives cost visibility |
| **Average latency** | Response time per run — important for production SLAs |
| **Error rate** | Should be 0% for our test conversations |
| **Tool calls** | 0 for this basic agent (no tools attached) |

4. Note the time range selector — you can filter to the last hour, 24 hours, 7 days, etc.

**💬 What to observe:**
- Token usage varies by prompt complexity. The multi-machine triage prompt likely used the most tokens.
- In production, you'd monitor these dashboards to detect cost spikes, latency degradation, or increasing error rates.

**✅ Expected result**

The monitoring dashboard showing agent run counts and token usage for your test conversations.

![Agent Monitoring Dashboard](./images/agent-monitoring-dashboard.png)

### Task 3: Drill into Agent Traces

1. From the agent's detail page, look for **Traces** or **Tracing** (this may be under the Monitor tab or a separate tab).
2. Select one of your recent agent runs to view its trace details.
3. Examine the **trace tree** — the sequence of operations for that run:
   - **User message** — your input prompt
   - **LLM call** — the model invocation with the full prompt (system + user)
   - **Assistant response** — the generated output
4. For each step, review:
   - **Latency** — how long did the LLM call take?
   - **Token count** — prompt tokens vs. completion tokens
   - **Model** — which deployment was used

**💬 What to observe:**
- The trace shows the full prompt sent to the model, including the system message. This is useful for debugging unexpected responses.
- Prompt tokens (system + user messages) are typically larger than completion tokens. Optimizing system prompts can reduce cost.
- If this agent had tools attached (file search, code interpreter, etc.), you'd see additional steps in the trace tree for each tool invocation.

**✅ Expected result**

An individual trace showing the message flow: user message → LLM call → assistant response, with latency and token counts.

![Agent Trace Detail](./images/agent-trace-detail.png)

### Task 4: Open in Application Insights

For deeper analysis, let's explore the raw telemetry in Azure Application Insights.

1. From the trace view in the Foundry Portal, look for an **"Open in Azure Monitor"** or **"View in Application Insights"** link.
   - Alternatively, open the **Azure portal** at [portal.azure.com](https://portal.azure.com), navigate to your resource group, and click on the **Application Insights** resource directly.
2. In Application Insights, explore **Transaction Search**:
   - Search for recent transactions — you should see entries corresponding to your agent runs.
   - Click on a transaction to see the **end-to-end transaction detail** — a timeline of all operations for that request.
3. Try a simple **KQL query** in the **Logs** section. Click **Logs** in the left sidebar and run:

```kusto
traces
| where timestamp > ago(1h)
| where message contains "Contoso" or message contains "agent"
| project timestamp, message, severityLevel
| order by timestamp desc
| take 20
```

4. Explore the **Performance** view — this shows response times across all agent runs and helps identify outliers.

> [!TIP]
> Application Insights is where production teams spend most of their troubleshooting time. The KQL query language is powerful for filtering, aggregating, and alerting on telemetry data. Bookmark this resource for your project.

**✅ Expected result**

Application Insights showing transaction search results and the KQL query output.

![Application Insights](./images/application-insights.png)

### Task 5: Set Up an Evaluation

Evaluations measure the **quality** of agent responses using built-in metrics. Let's evaluate our maintenance advisor.

1. In the Foundry Portal, navigate to **Operate** in the top navigation bar.
2. Look for **Evaluations** in the left sidebar (it may be under an **Observability** or **Quality** section).
3. Click **+ New evaluation**.
4. Configure the evaluation:

| Setting | Value |
|---------|-------|
| **Target** | Your `Contoso Tires Maintenance Advisor` agent |
| **Evaluation metrics** | Select: **Groundedness**, **Relevance**, **Coherence**, **Fluency** |
| **Test data** | Create or upload test prompts *(see below)* |

5. Use these evaluation prompts (enter them as the test dataset):

| # | Test Prompt | Expected Topic |
|---|-------------|---------------|
| 1 | What should I check if curing press TC-100 temperature reads 179°C? | Fault diagnosis — curing temp |
| 2 | Describe the bearing replacement procedure for tire building machine TB-200. | Maintenance procedure |
| 3 | What parts are needed for bladder replacement on the curing press? Include part numbers. | Parts knowledge |
| 4 | Machine E1 mixer vibration is at 5.8 mm/s. Is this a safety concern? | Safety assessment |
| 5 | How should I prioritize repairs when multiple machines have warnings? | Triage logic |

6. Start the evaluation run.

> [!NOTE]
> Evaluation runs can take a few minutes as each test prompt is sent to the agent and the responses are scored by an evaluator model.

**✅ Expected result**

The evaluation run appears in the list with status "Running" and eventually "Completed".

![Evaluation Created](./images/evaluation-created.png)

### Task 6: Analyze Evaluation Results

1. Once the evaluation completes, click on the run to view results.
2. Review the **overall scores** for each metric:

| Metric | What It Measures | Good Score |
|--------|-----------------|------------|
| **Groundedness** | Are responses based on provided context (not hallucinated)? | ≥ 4.0 / 5.0 |
| **Relevance** | Do responses address the question asked? | ≥ 4.0 / 5.0 |
| **Coherence** | Is the response logically structured and consistent? | ≥ 4.0 / 5.0 |
| **Fluency** | Is the language natural and well-written? | ≥ 4.0 / 5.0 |

3. Drill into **individual prompt results** — click on each test prompt to see its score breakdown.
4. Identify the **lowest-scoring prompt** — what went wrong? Common issues:
   - Low groundedness → Agent may have hallucinated part numbers or thresholds
   - Low relevance → Agent went off-topic or gave a generic answer
   - Low coherence → Response was disorganized or contradictory

**💬 What to observe:**
- Without tools like file search or code interpreter, the agent relies entirely on its training data. Responses about specific part numbers may score lower on groundedness because the model is generating them from memory rather than retrieving them.
- This is exactly the kind of insight that helps you decide: does this agent need RAG tools? Better instructions? Fine-tuning?

**✅ Expected result**

Evaluation results showing per-metric scores and per-prompt breakdowns.

![Evaluation Results](./images/evaluation-results.png)

> [!IMPORTANT]
> Evaluation scores are not absolute quality measures — they're relative indicators. Use them to **compare** across iterations: change the system prompt, add a tool, or fine-tune the model, then re-evaluate to see if scores improve.

## 🚀 Go Further

- **Compare system prompts**: Copy the agent, modify the instructions (e.g., make them shorter or more detailed), and run the same evaluation. Compare scores to see which prompt performs better.
- **Set up Azure Monitor alerts**: In Application Insights, create an alert rule that triggers when agent error rate exceeds 5% or average latency exceeds 10 seconds.
- **Evaluate the fine-tuned model**: If your fine-tuning job from [Admin Lab 1](../admin-lab-1/README.md) has completed, create a new agent using the fine-tuned model and run the same evaluation. Compare scores with the base model agent.
- **Custom KQL dashboards**: Build a custom workbook in Application Insights to visualize agent token usage by time of day, error patterns, or latency percentiles.

## 🧠 Conclusion

You've built a complete observability workflow:

- Created a test agent and generated trace data through realistic conversations
- Explored the built-in monitoring dashboard for high-level metrics
- Drilled into individual traces to understand execution details
- Navigated from Foundry Portal to Application Insights for deep analysis
- Ran an evaluation with built-in quality metrics
- Interpreted scores and identified improvement opportunities

**Next**: [Admin Lab 3 — Control Plane](../admin-lab-3/README.md)
