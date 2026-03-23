# Portal Lab 2: Agents / Agent Service

In this lab you'll create AI agents, have multi-turn conversations, work with agent versioning, enable persistent memory, and build a declarative multi-agent workflow — all through the Foundry Portal. No code required.

← [Portal Lab 1](../portal-lab-1/README.md) | **Portal Lab 2** | [Portal Lab 3 →](../portal-lab-3/README.md)

**Expected duration**: 45 min

**Prerequisites**:

- [Portal Lab 1](../portal-lab-1/README.md) completed (at least one model deployed and tested in the playground)

## 🎯 Objective

The goals for this lab are:

- Create a prompt-based agent with a manufacturing persona and test it in conversation.
- Understand agent versioning — modify instructions and compare behavior.
- Enable persistent memory so the agent remembers facts across conversations.
- Build a declarative workflow that chains two agents together.

## 🧭 Context and Background

In Portal Lab 1, you tested models directly in the playground. That's useful for experimentation, but in production scenarios you want a **reusable agent** — a model combined with instructions, tools, and memory that forms a consistent assistant.

The Foundry Portal lets you create and manage agents without writing code. An agent wraps a model deployment with:

| Component | Description |
|-----------|-------------|
| **Instructions** | System prompt that defines the agent's persona, capabilities, and response style |
| **Model** | The underlying model deployment the agent uses for inference |
| **Tools** | Optional capabilities: file search, code interpreter, web search, custom functions |
| **Memory** | Persistent knowledge the agent retains across conversations |
| **Versions** | Snapshots of the agent configuration — compare behavior across versions |

### Declarative Workflows

A **declarative workflow** lets you connect multiple agents in sequence. The output of one agent flows as input to the next. This is useful for complex tasks that benefit from specialization — e.g., one agent researches, another summarizes.

## ✅ Tasks

### Task 1: Create a Prompt Agent

1. In the Foundry Portal, navigate to **Build** → **Agents**.
2. Click **+ New agent**.
3. Configure the agent:
   - **Name**: `TireAssistant`
   - **Model**: Select `gpt-4o-mini` (pre-deployed from Challenge 0)
   - **Instructions**: Paste the following:

     ```
     You are a tire manufacturing assistant at Contoso Tires. You help
     technicians troubleshoot machine issues, recommend maintenance procedures,
     and answer questions about tire building, curing, and mixing processes.

     Always be specific about machine types and part numbers when possible.
     When diagnosing issues, ask clarifying questions if the technician hasn't
     provided enough detail. Structure your responses with clear steps.
     ```

4. Click **Create** (or **Save**) to create the agent.

<details>
<summary>✅ You should see something similar to this</summary>

The agent detail page showing the name "TireAssistant", the selected model, and the instructions you entered.

<!-- TODO: Replace with actual screenshot -->
![Agent Created](./images/placeholder-agent-created.png)

</details>

### Task 2: Multi-Turn Conversation

1. With `TireAssistant` selected, open the **chat** (or **test**) panel.
2. Have a multi-turn conversation. Try this sequence:

<details>
<summary>💬 Sample conversation</summary>

**You**: "The tire building drum on machine TB-200 is vibrating excessively. What should I check first?"

*Wait for the response, then follow up:*

**You**: "The vibration reading is 4.2 mm/s. Our threshold is 3.0 mm/s. Is that critical?"

*Wait for the response, then follow up:*

**You**: "What parts might I need to replace, and what's the estimated repair time?"

</details>

3. Notice how the agent maintains context across turns — it remembers you're discussing the TB-200 and the vibration reading without you repeating it.

### Task 3: Create Agent Versions

1. Go back to the agent's **Instructions** and modify them. Add the following lines at the end:

   ```
   Always respond in bullet points. Include an estimated repair time for every
   recommendation. End each response with a one-line safety reminder.
   ```

2. **Save** the agent. The portal automatically creates a new version each time you save — you don't need to include a version number in the name.
3. Note the version number the portal assigned (visible in the version history or agent details).
4. Ask the **same question** you asked in Task 2:
   > "The tire building drum on machine TB-200 is vibrating excessively. What should I check first?"
5. Compare the new version's response with the previous version's response:
   - Does the updated version use bullet points as instructed?
   - Does it include repair time estimates?
   - Does it end with a safety reminder?

<details>
<summary>💬 What to observe</summary>

- The **original version** likely gave paragraph-style responses without repair time estimates.
- The **new version** should produce structured bullet points, include time estimates, and add a safety reminder.
- This demonstrates how instruction changes directly alter agent behavior — the "art" of instruction engineering.

</details>

### Task 4: Enable Memory

1. In the agent configuration, look for the **Tools** section.
2. Enable the **Memory** tool (it may appear as a toggle or an add-on tool).
3. Once enabled, open a **new conversation** with the agent.
4. Teach the agent a fact:
   > "Remember this: I always work on machine TB-200 in Building A, Line 3. My name is Alex and I'm a Level 2 mechanical technician."
5. The agent should acknowledge that it will remember this information.
6. Now **start a completely new conversation** (not just a new message — a fresh session).
7. In the new conversation, ask:
   > "Which machine do I usually work on, and what's my certification level?"
8. Verify the agent recalls the information from the previous conversation.

<details>
<summary>✅ Expected result</summary>

The agent should respond with something like: "You usually work on machine TB-200 in Building A, Line 3, and you're a Level 2 mechanical technician."

If the agent doesn't recall, check that the Memory tool is enabled and that you saved the agent configuration before starting the new conversation.

</details>

> [!IMPORTANT]
> Memory allows the agent to build persistent knowledge about the user across sessions. This is powerful for scenarios where technicians interact with the same agent over days or weeks — the agent learns their role, preferences, and common tasks.

### Task 5: Create a Declarative Workflow

In this task you'll create a simple two-agent workflow where a **Researcher** agent investigates a problem and a **Summarizer** agent condenses the findings into an executive summary.

**Step 1 — Create the Researcher agent:**

1. Navigate to **Build** → **Agents** and create a new agent:
   - **Name**: `ManufacturingResearcher`
   - **Model**: `gpt-4o-mini`
   - **Instructions**:

     ```
     You are a manufacturing research analyst. When given a manufacturing
     problem, investigate thoroughly by:
     1. Identifying possible root causes (at least 3)
     2. Listing relevant machine types and components involved
     3. Describing potential impact on production quality and throughput
     4. Suggesting diagnostic tests to confirm each root cause

     Be detailed and technical. Your output will be read by another agent
     that will summarize your findings.
     ```

2. Save the agent.

**Step 2 — Create the Summarizer agent:**

1. Create another new agent:
   - **Name**: `ExecutiveSummarizer`
   - **Model**: `gpt-4o-mini`
   - **Instructions**:

     ```
     You are an executive communications specialist for a tire manufacturing
     company. You receive detailed technical analysis and produce a concise
     executive summary that includes:
     1. A one-paragraph situation overview
     2. Top 3 findings (bullet points, plain language)
     3. Recommended immediate actions (prioritized list)
     4. Estimated business impact (production hours, cost if available)

     Keep the summary under 200 words. Avoid jargon — write for a plant
     manager audience.
     ```

2. Save the agent.

**Step 3 — Create the workflow:**

1. Navigate to **Build** → **Agents** and look for an option to create a **Workflow** (or **Multi-agent workflow** / **Declarative workflow**).
2. Add the two agents in sequence:
   - **Step 1**: `ManufacturingResearcher`
   - **Step 2**: `ExecutiveSummarizer`
3. Configure the workflow so the output of the Researcher flows as input to the Summarizer.
4. Save the workflow.

**Step 4 — Test the workflow:**

1. Open the workflow and start a conversation.
2. Send this prompt:
   > "Investigate why our tire curing press TC-100 is producing inconsistent cure times across batches. Recent data shows cure time variance has increased from ±2 seconds to ±8 seconds over the past month."
3. Observe the two-stage output:
   - First, the Researcher produces a detailed technical analysis.
   - Then, the Summarizer condenses it into a brief executive summary.

<details>
<summary>💬 What to observe</summary>

- The Researcher should produce a thorough, technical analysis with multiple root causes and diagnostic suggestions.
- The Summarizer should distill this into a concise, business-friendly summary.
- This demonstrates how agent specialization through workflows can handle complex tasks more effectively than a single agent.

</details>

## 🚀 Go Further

> [!NOTE]
> Finished early? These optional exercises let you experiment further with agents.

### Task 6: Swap Models and Compare

1. Edit `TireAssistant` and change the model from `gpt-4o-mini` to `gpt-4.1`.
2. Ask the same troubleshooting question from Task 2.
3. Compare the responses:
   - Is the `gpt-4.1` response more detailed or higher quality?
   - Does it follow the instructions more precisely?
   - How does response speed compare?

### Task 7: Instruction Engineering Experiments

Try creating variations of the TireAssistant with different instruction styles and compare their effectiveness:

| Style | Example instruction snippet |
|-------|---------------------------|
| **Persona-heavy** | "You are Marcus, a 20-year veteran tire manufacturing expert who learned the trade on the factory floor..." |
| **Constraint-focused** | "RULES: 1. Never exceed 100 words per response. 2. Always start with the most critical step. 3. Use numbered lists only..." |
| **Few-shot** | "Here is an example of a good response: Q: 'Press is overheating' → A: '1. Check thermocouple readings...'" |
| **Chain-of-thought** | "Think through the problem step by step. First identify the symptom, then the subsystem, then possible causes, then rank by likelihood..." |

Create 2–3 variants and test them with the same question. Note which style produces the most useful, actionable responses for a maintenance technician.

## 🛠️ Troubleshooting and FAQ

<details>
<summary>Agent creation fails or I can't find the Agents section</summary>

- Ensure you're in the correct project scope in the Foundry Portal.
- The Agents section is under **Build** → **Agents**. If you don't see it, your project may need the Agent Service enabled — ask your coach.

</details>

<details>
<summary>Memory doesn't persist across conversations</summary>

- Verify the Memory tool is enabled in the agent's tool configuration.
- Make sure you saved the agent configuration after enabling memory.
- Try teaching a simple fact ("My name is Alex") and then starting a fresh conversation to test recall.
- Memory may take a moment to index — wait 10–15 seconds before starting the new conversation.

</details>

<details>
<summary>Workflow builder is not available</summary>

- Declarative workflows may be in preview and not available in all regions.
- As a workaround: run the Researcher agent manually, copy its output, and paste it as input to the Summarizer agent. You'll still experience the two-agent pattern, just with a manual handoff step.

</details>

<details>
<summary>Agent responses don't follow the instructions</summary>

- Check that the instructions were saved (not just typed).
- Try making the instructions more explicit: add "ALWAYS" or "NEVER" for critical rules.
- Smaller models (gpt-4o-mini) may follow complex instructions less reliably than larger models (gpt-4.1). Try switching to a larger model if instruction compliance is an issue.

</details>

## 🧠 Conclusion

In this lab you learned to:
- **Create** agents with custom instructions and personas
- **Version** agents to compare instruction changes side by side
- **Enable memory** for persistent knowledge across conversations
- **Build workflows** that chain specialized agents for complex tasks

You've seen how agents go beyond simple chat — they combine models with instructions, memory, and workflows to create consistent, specialized assistants.

**Next**: [Portal Lab 3 — Tools & Foundry IQ](../portal-lab-3/README.md)
