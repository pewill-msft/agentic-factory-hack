# Portal Lab 3: Tools & Foundry IQ

In this lab you'll extend your agent with built-in tools — File Search, Code Interpreter, and Web Search — and then connect a Foundry IQ knowledge base grounded in your factory's documentation. No code required.

← [Portal Lab 2](../portal-lab-2/README.md) | **Portal Lab 3**

**Expected duration**: 45 min

**Prerequisites**:

- [Portal Lab 2](../portal-lab-2/README.md) completed (you have a working `TireAssistant` agent)
- The pre-provisioned **Storage Account** contains wiki markdown files (set up in Challenge 0)

## 🎯 Objective

The goals for this lab are:

- Use **File Search** to let the agent query an uploaded document.
- Use **Code Interpreter** to have the agent generate data visualizations and return files.
- See the contrast of **Web Search** — before and after enabling live web access.
- Create a **Foundry IQ** knowledge base from existing blob storage documents and connect it to your agent.
- Understand how Foundry IQ performs agentic retrieval: query decomposition, parallel search, reranking, and cited responses.

## 🧭 Context and Background

In Portal Lab 2, you created agents with custom instructions and memory. Now you'll give those agents access to **tools** — additional capabilities beyond the base model.

The Foundry Portal offers several built-in tools:

| Tool | What it does |
|------|-------------|
| **File Search** | Upload files (PDF, DOCX, etc.) and let the agent search their content to answer questions |
| **Code Interpreter** | Run Python code in a sandboxed environment — generate charts, perform calculations, process files |
| **Web Search** (Bing Grounding) | Search the live web for current information and return cited results |
| **Memory** | Persist facts across conversations (you used this in Lab 2) |

### Foundry IQ

**Foundry IQ** is an enterprise knowledge retrieval system that goes beyond simple file search. It connects to data sources (blob storage, SharePoint, etc.), indexes the content, and provides **agentic retrieval** — a multi-step process:

```
User question
    ↓
❶ Query decomposition — break complex questions into sub-queries
    ↓
❷ Parallel search — search across all connected knowledge sources simultaneously
    ↓
❸ Reranking — score and prioritize the most relevant results
    ↓
❹ Cited response — generate an answer with inline citations to source documents
```

This is more sophisticated than uploading a single file — Foundry IQ handles large knowledge bases with multiple documents and provides traceable answers.

## ✅ Tasks

### Task 1: File Search

In this task you'll upload a maintenance manual and let the agent search it to answer questions.

1. Open your `TireAssistant` agent in the Foundry Portal (or create a new agent for this exercise).
2. In the agent's **Tools** configuration, enable **File Search**.
3. Upload the maintenance manual:
   - You can use the file at [`portal-lab-3/data/contoso-tires-maintenance-manual.md`](./data/contoso-tires-maintenance-manual.md) in this repository.
   - Download it and convert it to PDF, or upload the markdown file directly (if supported).

> [!TIP]
> To convert the markdown to PDF: open it in any markdown viewer (VS Code, GitHub) and print to PDF. Or upload the `.md` file directly — the file search tool can process plain text.

4. Once the file is uploaded and indexed, ask the agent these questions:

<details>
<summary>💬 Sample prompts for File Search</summary>

**Prompt 1** — Maintenance schedule:
> "What is the quarterly maintenance task for the tire building machine TB-200/TB-300?"

**Prompt 2** — Safety procedure:
> "What are the steps in the pre-service safety checklist?"

**Prompt 3** — Part replacement:
> "Walk me through the procedure for replacing heating elements on the tire curing press. What tools and parts do I need?"

</details>

5. Observe the responses:
   - Does the agent cite the uploaded document?
   - Are the answers grounded in the document content (not general knowledge)?
   - Can you see which section of the document the answer came from?

<details>
<summary>✅ Expected result</summary>

The agent should answer using specific content from the maintenance manual — part numbers (like TCP-HTR-4KW), torque specifications (25–35 Nm), and procedure steps. Answers should include citations or references to the uploaded file.

</details>

### Task 2: Code Interpreter

In this task the agent will generate a data visualization from defect rate data.

1. Ensure **Code Interpreter** is enabled in the agent's tools.
2. Send this prompt to the agent:

   ```
   Create a bar chart showing these monthly defect rates by machine:
   - TB-200: Jan 2.1%, Feb 2.6%, Mar 1.9%
   - TB-300: Jan 3.4%, Feb 3.8%, Mar 3.0%
   - TC-100: Jan 4.2%, Feb 3.5%, Mar 4.5%
   - BM-500: Jan 2.7%, Feb 2.3%, Mar 2.8%

   Use red color for any bar where the defect rate exceeds 3.0%.
   Use green for bars at or below 3.0%.
   Add a horizontal dashed line at 3.0% labeled "Quality Threshold".
   Title: "Contoso Tires — Monthly Defect Rates by Machine (Q1 2025)"
   ```

3. The agent should generate Python code, execute it, and return a chart image.
4. Download the generated image file.

<details>
<summary>✅ Expected result</summary>

A grouped bar chart with 4 machines × 3 months. Bars above 3.0% are red, bars at or below 3.0% are green. A horizontal dashed line marks the quality threshold. The chart should be downloadable as a PNG or similar image file.

</details>

> [!TIP]
> **Optional**: Instead of embedding data in the prompt, try uploading the CSV file at [`portal-lab-3/data/defect-rates.csv`](./data/defect-rates.csv). Ask the agent: "Create a bar chart from this CSV file showing defect rates by machine and month."

### Task 3: Web Search (Before & After)

This task demonstrates the value of web grounding by showing the same question with and without web access.

**Without web search:**

1. Make sure **Web Search** (Bing Grounding) is **not** enabled on the agent.
2. Ask the agent:
   > "What were the latest major announcements from Microsoft Build 2026?"
3. Observe the response — the agent will either:
   - Decline, saying it doesn't have that information, or
   - Hallucinate generic guesses based on its training data cutoff.

**With web search:**

4. Now enable the **Web Search** tool (Bing Grounding) in the agent's tool configuration.
5. Ask the **exact same question** again:
   > "What were the latest major announcements from Microsoft Build 2026?"
6. Observe the difference:
   - The agent now returns specific, current results.
   - Each fact should include a **citation** linking to the source webpage.

<details>
<summary>💬 What to observe</summary>

This before/after contrast is the clearest demonstration of why tools matter:
- **Without web search**: The model is limited to its training data. It cannot know about events after training.
- **With web search**: The model can access current information and provide cited, verifiable answers.
- The citations are critical for enterprise use — you can verify every claim the agent makes.

</details>

### Task 4: Create a Foundry IQ Knowledge Base

1. In the Foundry Portal, navigate to **Build** → **Knowledge** (or **Knowledge bases**).
2. Click **+ New knowledge base** (or **+ Add knowledge source**).
3. Configure the knowledge base:
   - **Name**: `ContosoTiresDiagnostics`
   - **Data source**: Select **Azure Blob Storage**
   - **Connection**: Connect to the pre-provisioned storage account from Challenge 0
   - **Container/path**: Select the container that holds the wiki markdown files (the 5 machine diagnostic guides)
4. Click **Create** and wait for indexing to complete.

The following files should be indexed:

| File | Content |
|------|---------|
| `tire_curing_press.md` | Curing press faults, diagnostics, corrective actions |
| `tire_building_machine.md` | Building machine vibration and tension issues |
| `tire_extruder.md` | Extruder temperature and pressure faults |
| `tire_uniformity_machine.md` | Uniformity testing and measurement issues |
| `banbury_mixer.md` | Mixing temperature and rotor faults |

<details>
<summary>✅ You should see something similar to this</summary>

The knowledge base detail page showing "ContosoTiresDiagnostics" with 5 indexed documents and status "Ready."

<!-- TODO: Replace with actual screenshot -->
![Foundry IQ Knowledge Base](./images/placeholder-foundry-iq-kb.png)

</details>

> [!NOTE]
> Indexing may take 2–5 minutes depending on the number of files and their size. You'll see a progress indicator.

### Task 5: Connect KB to Agent & Test

1. Go back to your `TireAssistant` agent.
2. In the **Tools** or **Knowledge** configuration, connect the `ContosoTiresDiagnostics` knowledge base.
3. Now test with documentation-grounded questions:

<details>
<summary>💬 Sample prompts for Foundry IQ</summary>

**Prompt 1** — Specific diagnostics:
> "What are the diagnostic steps for excessive curing temperature on a tire curing press?"

**Prompt 2** — Threshold values:
> "What is the normal vibration threshold for a tire building machine, and what happens when it's exceeded?"

**Prompt 3** — Cross-document query:
> "Compare the estimated repair times for drum bearing replacement on the tire building machine versus rotor blade replacement on the Banbury mixer."

**Prompt 4** — Specific fault lookup:
> "What are the likely causes and corrective actions for the fault type 'ply_tension_excessive'?"

</details>

4. Verify the responses:
   - Are **citations** included, showing which source document was used?
   - Do the answers match the content in the wiki markdown files?
   - Try the cross-document query (Prompt 3) — Foundry IQ should pull from multiple documents and combine the information.

<details>
<summary>💬 How Foundry IQ works behind the scenes</summary>

When you ask a question:
1. **Query decomposition**: The system breaks your question into targeted sub-queries.
2. **Parallel search**: Each sub-query searches across all 5 indexed documents simultaneously.
3. **Reranking**: Results are scored for relevance and the top matches are selected.
4. **Cited response**: The agent generates an answer that includes inline citations like `[tire_building_machine.md]`.

This is fundamentally different from File Search (Task 1), which works with individual uploaded files. Foundry IQ handles enterprise-scale knowledge bases with multiple documents and provides traceable, cited answers.

</details>

## 🚀 Go Further

> [!NOTE]
> Finished early? These optional exercises let you explore additional tool types and advanced knowledge configurations.

### Task 6: Explore the Tool Catalog

1. In the Foundry Portal, navigate to **Build** → **Tools**.
2. Browse the available tool categories:

| Category | Description | Examples |
|----------|-------------|---------|
| **Built-in** | Tools provided by the platform | File Search, Code Interpreter, Web Search, Memory |
| **Remote MCP** | MCP servers hosted externally | Custom APIs, third-party services |
| **Local MCP** | MCP servers running locally | Local tools for development/testing |
| **Custom tools** | Function definitions you create | Custom API calls, data transformations |

3. Explore what's available. Think about which tools could enhance your manufacturing agent:
   - Could a Remote MCP tool connect to a real CMMS (maintenance management system)?
   - Could a Custom tool look up real-time parts inventory?

### Task 7: Hybrid Knowledge — Enterprise + Web

1. Go back to your `ContosoTiresDiagnostics` knowledge base.
2. Look for an option to add **Bing** (web) as an additional knowledge source.
3. If available, enable it and test with a hybrid query:
   > "Compare our documented maintenance schedule for the tire curing press with industry best practices for curing press maintenance."
4. The response should combine:
   - **Enterprise data**: Your internal documentation (with citations to the wiki files)
   - **Web data**: Current industry best practices (with citations to web URLs)

<details>
<summary>💬 Why hybrid knowledge matters</summary>

In real manufacturing environments, teams need both:
- **Internal knowledge**: SOPs, machine-specific procedures, tribal knowledge captured in runbooks
- **External knowledge**: Industry standards, manufacturer recommendations, regulatory updates

Foundry IQ's hybrid approach lets you combine both in a single agent, with full citations so users know whether they're reading internal docs or external sources.

</details>

## 🛠️ Troubleshooting and FAQ

<details>
<summary>File Search doesn't find content from the uploaded document</summary>

- Ensure the file was fully processed (check for an "indexed" or "ready" status next to the file).
- Large files may take a minute to index. Wait and try again.
- Try asking a question that uses exact words from the document — this confirms the index is working.

</details>

<details>
<summary>Code Interpreter doesn't generate a chart</summary>

- Make sure Code Interpreter is enabled in the agent's tool configuration.
- The model may occasionally return code instead of executing it. Ask: "Please execute the code and return the chart as a downloadable file."
- Try simplifying the request: "Create a simple bar chart with these values: A=10, B=20, C=15."

</details>

<details>
<summary>Web Search returns no results or generic responses</summary>

- Verify the Bing Grounding / Web Search tool is enabled and saved.
- Try a query you know has current web results: "What is today's date?" or "Latest news about Azure."
- Web search may not be available in all regions — ask your coach about your deployment region.

</details>

<details>
<summary>Foundry IQ indexing seems stuck</summary>

- Indexing 5 small markdown files should complete in 2–5 minutes.
- If stuck beyond 10 minutes, check the storage account connection — the system needs read access to the blob container.
- Try deleting the knowledge base and recreating it with the connection.

</details>

<details>
<summary>Citations don't appear in agent responses</summary>

- Not all question types trigger citations. Try asking a very specific factual question: "What is the normal drum vibration threshold for a tire building machine?"
- Check that the knowledge base shows as "Connected" or "Active" on the agent.
- Some models follow citation instructions more reliably than others — try switching to gpt-4.1.

</details>

## 🧠 Conclusion

In this lab you learned to:
- **File Search**: Upload documents and let the agent answer questions from their content
- **Code Interpreter**: Generate visualizations and perform calculations within a sandboxed environment
- **Web Search**: Add live web access and see the clear before/after difference
- **Foundry IQ**: Build an enterprise knowledge base from blob storage and get cited, multi-document answers

Together, these tools transform a basic chat model into a capable enterprise assistant — grounded in your data, able to compute and visualize, and connected to the live web when needed.

---

## 🎉 Congratulations!

You've completed all four Portal Labs. Here's what you've accomplished:

| Lab | What you learned |
|-----|-----------------|
| **Portal Lab 0** | Validated your environment and confirmed resource access |
| **Portal Lab 1** | Discovered, deployed, and tested models in the playground |
| **Portal Lab 2** | Created agents with memory and built multi-agent workflows |
| **Portal Lab 3** | Extended agents with tools and enterprise knowledge |

All of this was done through the Foundry Portal — no code required. These same capabilities can also be accessed programmatically through the Foundry SDK (Python, .NET) as demonstrated in Challenges 1–4 of this hackathon.

> [!TIP]
> Interested in the code-first approach? Check out [Challenge 1](../challenge-1/README.md) to see how to build agents programmatically in Python, or [Challenge 2](../challenge-2/README.md) for the .NET/C# approach.
