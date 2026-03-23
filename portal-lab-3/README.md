# Portal Lab 3: Tools & Foundry IQ

In this lab you'll extend your agent with built-in tools — File Search, Code Interpreter, and Web Search — and then connect a Foundry IQ knowledge base grounded in your factory's documentation. No code required.

← [Portal Lab 2](../portal-lab-2/README.md) | **Portal Lab 3**

**Expected duration**: 45 min

**Prerequisites**:

- [Portal Lab 2](../portal-lab-2/README.md) completed (you have a working `TireAssistant` agent)
- Access to the pre-provisioned **Storage Account** in the Azure Portal (set up in Challenge 0)
- Lab data files downloaded to your local machine (see below)

### Download Lab Files from GitHub

This lab requires several data files. Since you're working directly from the Foundry Portal (not in a codespace), download the files from GitHub to your local machine first.

**Option A — Download individual files:**

1. Navigate to the repository on GitHub.
2. Open the file you need (e.g. `portal-lab-3/data/contoso-tires-maintenance-manual.pdf`).
3. Click the **Download raw file** button (↓ icon) in the top-right of the file view.
4. Repeat for each file you need.

**Option B — Download the whole folder as a ZIP:**

1. Navigate to the repository's main page on GitHub.
2. Click the green **Code** button → **Download ZIP**.
3. Extract the ZIP and find the files under `portal-lab-3/data/`.

**Files you'll need during this lab:**

| File | Used in | Purpose |
|------|---------|---------|
| `portal-lab-3/data/contoso-tires-maintenance-manual.pdf` | Task 1 | Maintenance manual for File Search |
| `portal-lab-3/data/defect-rates.csv` | Task 2 (optional) | Defect data for Code Interpreter |
| `portal-lab-3/data/kb-wiki/tire_curing_press.md` | Task 4 | Machine diagnostic guide |
| `portal-lab-3/data/kb-wiki/tire_building_machine.md` | Task 4 | Machine diagnostic guide |
| `portal-lab-3/data/kb-wiki/tire_extruder.md` | Task 4 | Machine diagnostic guide |
| `portal-lab-3/data/kb-wiki/tire_uniformity_machine.md` | Task 4 | Machine diagnostic guide |
| `portal-lab-3/data/kb-wiki/banbury_mixer.md` | Task 4 | Machine diagnostic guide |

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
| **Web Search** | Search the live web for current information and return cited results |
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

### The Three IQ Workloads

Microsoft provides three **IQ workloads** that give agents access to different aspects of your organization:

| IQ Workload | What it does | Data sources |
|-------------|-------------|-------------|
| **Foundry IQ** | Managed knowledge layer for enterprise data | Azure Blob Storage, SharePoint, OneLake, web — structured and unstructured content |
| **Fabric IQ** | Semantic intelligence layer for Microsoft Fabric | OneLake, Power BI — business data, ontologies, semantic models |
| **Work IQ** | Contextual intelligence layer for Microsoft 365 | Documents, meetings, chats, workflows — collaboration signals |

Each IQ workload is standalone, but they can be used together to give agents comprehensive organizational context. In this lab you'll work with **Foundry IQ** — the one focused on grounding agents in enterprise knowledge with permission-aware, cited responses.

> [!TIP]
> Learn more: [What is Foundry IQ?](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/what-is-foundry-iq)

## ✅ Tasks

### Task 1: File Search

In this task you'll upload a maintenance manual PDF and let the agent search it to answer questions.

1. Open your `TireAssistant` agent in the Foundry Portal (or create a new agent for this exercise).
2. In the agent's left-hand configuration panel, find the **Tools** section and click the **Upload files** button.
3. The **Attach files** dialog opens:
   - **Index option**: Leave as **Create a new index** (default).
   - **Vector index name**: A name is auto-generated for you (e.g. `index_honest_floor_8wyjngstpk`). You can keep the generated name or rename it to something meaningful like `contoso-tires-maintenance`.
   - Click **browse for files** (or drag and drop) and select the file [`portal-lab-3/data/contoso-tires-maintenance-manual.pdf`](./data/contoso-tires-maintenance-manual.pdf) from this repository.
4. The file appears in a table showing **Name**, **Size** (~120 KB), and **Status** (Uploading → Uploaded).
5. Click **Attach** to confirm.

> [!NOTE]
> Behind the scenes, this creates a vector index from your PDF and enables the **File Search** tool on the agent. The indexing may take a moment to complete.

6. Once the file is attached and indexed, ask the agent these questions:

<details>
<summary>💬 Sample prompts for File Search</summary>

**Prompt 1** — Maintenance schedule:
> "What is the quarterly maintenance task for the tire building machine TB-200/TB-300?"

**Prompt 2** — Safety procedure:
> "What are the steps in the pre-service safety checklist?"

**Prompt 3** — Part replacement:
> "Walk me through the procedure for replacing heating elements on the tire curing press. What tools and parts do I need?"

</details>

7. Observe the responses:
   - Does the agent cite the uploaded document?
   - Are the answers grounded in the document content (not general knowledge)?
   - Can you see which section of the document the answer came from?

8. **Inspect the response metadata and traces.** After the agent responds, look at the bottom of each chat message — you'll see a metadata bar showing the model used (e.g. `gpt-4.1`), response time, token count, and quality/safety scores (e.g. **AI Quality: 100%**, **Safety: 100%**).

9. Click the **Logs** button at the bottom of a response (or switch to the **Traces** tab at the top of the agent page) to open the trace view:
   - You'll see the full **Conversation** with status (**Completed**), total duration, and token count.
   - Expand a response to see individual steps — notice the **file_search** tool calls followed by the **message** output. This confirms the agent actually searched the uploaded PDF before answering.
   - Click on a specific response row to open the **Input + Output** panel on the right:
     - **Input** shows the user prompt and token count.
     - **Output** shows the full agent response and token count.
     - The **Metadata** tab shows additional details (model, timing, etc.).
     - The **Evaluations** tab shows how many quality checks were run and their results.

> [!TIP]
> The Traces view is invaluable for understanding **how** the agent arrived at its answer. For File Search specifically, you can verify that the tool was invoked and that the response is grounded in the document — not hallucinated from the model's training data.

<details>
<summary>✅ Expected result</summary>

The agent should answer using specific content from the maintenance manual — part numbers (like TCP-HTR-4KW), torque specifications (25–35 Nm), and procedure steps. Answers should include citations or references to the uploaded file. In the Traces view, you should see `file_search` tool calls for each response.

</details>

### Task 2: Code Interpreter

In this task you'll enable the Code Interpreter tool and use it to generate a data visualization from defect rate data.

1. In the agent's left-hand configuration panel, find the **Tools** section and click **Add**.
2. In the **Select a tool** dialog (which has three tabs: **Configured**, **Catalog**, **Custom**), select **Code interpreter**.
3. The tool is added immediately — you'll see it listed under Tools alongside File Search.
4. Click **Save** to create a new agent version with Code Interpreter enabled.
5. Send this prompt to the agent:

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

6. The agent should generate Python code, execute it, and return a chart image.
7. Download the generated image file.

<details>
<summary>✅ Expected result</summary>

A grouped bar chart with 4 machines × 3 months. Bars above 3.0% are red, bars at or below 3.0% are green. A horizontal dashed line marks the quality threshold. The chart should be downloadable as a PNG or similar image file.

</details>

> [!TIP]
> **Optional**: Instead of embedding data in the prompt, try uploading the CSV file at [`portal-lab-3/data/defect-rates.csv`](./data/defect-rates.csv). Ask the agent: "Create a bar chart from this CSV file showing defect rates by machine and month."

### Task 3: Web Search (Before & After)

This task demonstrates the value of web search by showing the same question with and without web access.

**Without web search:**

1. Make sure **Web Search** is **not** enabled on the agent.
2. Ask the agent:
   > "Were there any manufacturing related announcements at Microsoft Ignite 2025?"
3. Observe the response — the agent will either:
   - Decline, saying it doesn't have that information, or
   - Hallucinate generic guesses based on its training data cutoff.

**With web search:**

4. In the agent's **Tools** section, click **Add**.
5. In the **Select a tool** dialog, select **Web search**.
6. The **Add the Web Search Tool** dialog opens:
   - **Search type**: Select **Search the web with Bing Search** (no setup required).
   - The other option — *Search specific domains with Bing Custom Search* — requires a separate Bing Custom Search connection.
   - Click **Add**.
7. Click **Save** to create a new agent version with Web Search enabled.
8. Ask the **exact same question** again:
   > "Were there any manufacturing related announcements at Microsoft Ignite 2025?"
9. Observe the difference:
   - The agent now returns specific, current results.
   - Each fact should include a **citation** linking to the source webpage.

<details>
<summary>💬 What to observe</summary>

This before/after contrast is the clearest demonstration of why tools matter:
- **Without web search**: The model is limited to its training data. It may not have detailed information about specific events.
- **With web search**: The model can access current information and provide cited, verifiable answers.
- The citations are critical for enterprise use — you can verify every claim the agent makes.

</details>

### Task 4: Upload Wiki Files & Create a Foundry IQ Knowledge Base

In this task you'll upload machine diagnostic documents to blob storage and then create a Foundry IQ knowledge base so your agent can answer questions grounded in those documents.

**Step 1 — Upload wiki files to blob storage:**

This repository includes 5 machine diagnostic guides in [`portal-lab-3/data/kb-wiki/`](./data/kb-wiki/):

| File | Content |
|------|---------|
| `tire_curing_press.md` | Curing press faults, diagnostics, corrective actions |
| `tire_building_machine.md` | Building machine vibration and tension issues |
| `tire_extruder.md` | Extruder temperature and pressure faults |
| `tire_uniformity_machine.md` | Uniformity testing and measurement issues |
| `banbury_mixer.md` | Mixing temperature and rotor faults |

You need to upload these to the pre-provisioned **Storage Account** so Foundry IQ can index them.

1. Open the [Azure Portal](https://portal.azure.com) and navigate to your resource group (e.g. `rg-foundry-demo`).
2. Open the **Storage account** (name starts with `msagthack...`).
3. In the left navigation, click **Storage browser**.
4. Expand **Blob containers** — you'll see a `machine-wiki` container (created during Challenge 0 provisioning). Click on it.

> [!NOTE]
> If the `machine-wiki` container doesn't exist yet, click **+ Add container** at the top, name it `machine-wiki`, and click **Create**.

5. Click **Upload** in the toolbar at the top of the container view.
6. In the upload dialog, click **Browse for files** and select all 5 `.md` files from the `portal-lab-3/data/kb-wiki/` folder on your local machine.
7. Click **Upload**. The files should upload in seconds.
8. Verify all 5 files appear in the `machine-wiki` container.

**Step 2 — Create a knowledge base in Foundry IQ:**

1. Go back to the **Foundry Portal** (ai.azure.com) and navigate to **Knowledge** in the left navigation (under the **Build** section). This opens the **Foundry IQ** page with two tabs: **Knowledge bases** and **Indexes**.
2. Click **Create a knowledge base**.
3. The **Choose a knowledge type** dialog opens. You'll see several options:
   - Azure AI Search Index
   - **Azure Blob Storage** ← select this one
   - Web
   - Microsoft SharePoint (Remote)
   - Microsoft SharePoint (Indexed)
   - Microsoft OneLake
4. The **Create a knowledge source** form opens with these fields:
   - **Name**: Enter a name like `machine-wiki-blob` (or keep the auto-generated name).
   - **Description**: Enter something like `Machine diagnostic guides for Contoso Tires factory equipment`.
   - **Storage account**: Select your pre-provisioned storage account from the dropdown.
   - **Container name**: Select `machine-wiki`.
   - **Authentication type**: Change to **API Key**.
   - **Content extraction mode**: Leave as **minimal**.
   - **Include embedding model**: Leave checked (recommended). The **Embedding model** should show `text-embedding-3-large`.
   - **Chat completions model**: Should show `gpt-4.1`.
5. Click **Create** and wait for indexing to complete.

<details>
<summary>✅ You should see something similar to this</summary>

The Foundry IQ page showing your knowledge base with status **Active**, the knowledge source name, and the connection to your AI Search resource.

<!-- TODO: Replace with actual screenshot -->
![Foundry IQ Knowledge Base](./images/placeholder-foundry-iq-kb.png)

</details>

> [!NOTE]
> Indexing may take 2–5 minutes depending on the number of files and their size. You'll see the status change from "Creating" to **Active** when ready.

### Task 5: Connect Knowledge Base to Agent & Test

1. Go back to your `TireAssistant` agent in the Foundry Portal.
2. First, update the agent's **Instructions** to tell it to use the knowledge base. Replace the existing instructions with:

   ```
   You are a tire manufacturing assistant at Contoso Tires.

   Use the knowledge base tool to answer user questions about machine
   diagnostics, fault codes, maintenance procedures, and repair times.
   If the knowledge base doesn't contain the answer, respond with
   "I don't have that information in the maintenance documentation."

   When you use information from the knowledge base, include citations
   to the retrieved sources.

   Always be specific about machine types and part numbers when possible.
   Structure your responses with clear steps.
   ```

   > [!TIP]
   > Explicitly telling the agent to "use the knowledge base tool" increases the chance it will actually call the tool instead of relying on its own training data. See [Optimize agent instructions for knowledge retrieval](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/foundry-iq-connect?tabs=foundry%2Cpython#optimize-agent-instructions-for-knowledge-retrieval) for more guidance.

3. In the left-hand configuration panel, find the **Knowledge** section (marked **Preview**) and click **Add**.
4. From the dropdown, select **Connect to Foundry IQ**.
5. The **Connect to Foundry IQ** dialog opens:
   - **Connection**: Your AI Search connection is pre-selected (e.g. `msagthack-aifoundry-...-aisearch`).
   - **Knowledge base**: Select the knowledge base you just created from the dropdown.
6. Click **Connect** (or **Add**). The knowledge base now appears under the Knowledge section.
7. Click **Save** to create a new agent version with the knowledge base connected.
8. Now test with documentation-grounded questions:

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

9. Verify the responses:
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

1. In the agent playground, click **Add** in the **Tools** section to open the **Select a tool** dialog.
2. Browse the three tabs:

The dialog has three tabs:

| Tab | Description | Examples |
|-----|-------------|----------|
| **Configured** | Tools ready to use with your project's authentication and connections | File search, Code interpreter, Web search, Azure AI search, Grounding with Bing Search, SharePoint, Fabric Data Agent, Browser Automation, plus any custom MCP connections |
| **Catalog** | Browse and add tools from the platform catalog | Additional platform tools and connectors |
| **Custom** | Define your own function tools or OpenAPI-based tools | Custom API calls, data transformations |

3. Explore what's available. Think about which tools could enhance your manufacturing agent:
   - Could a Remote MCP tool connect to a real CMMS (maintenance management system)?
   - Could a Custom tool look up real-time parts inventory?

### Task 7: Explore the Knowledge Base in Azure AI Search

In Task 4 you created a knowledge base through Foundry IQ. Behind the scenes, this created resources in your **Azure AI Search** service. In this task you'll explore those resources directly.

**Step 1 — Browse the knowledge base in Azure AI Search:**

1. Open the [Azure Portal](https://portal.azure.com) and navigate to your resource group.
2. Open the **Azure AI Search** service (name starts with `msagthack-search-...`).
3. In the left navigation, expand **Agentic retrieval** and click **Knowledge bases**.
4. You'll see your knowledge base listed with its name, knowledge source, and chat completion model (`gpt-4.1`).
5. Click on your knowledge base to open its detail page. You'll see:
   - **Basics** section with the knowledge source (status should be **Active**) and last sync time.
   - **Retrieval** section with **Reasoning effort** (Minimal) and **Retrieval instructions** — this is where you could add instructions to guide knowledge source selection and query planning.
   - **Chat completion model** — the model used for agentic retrieval (query decomposition and reranking).
   - **Output configurations** — settings for how results are returned.

**Step 2 — Browse the indexed content:**

1. Still on the knowledge base detail page, notice the main content area shows the **indexed documents** from your wiki files.
2. Scroll through the content — you'll see the full structure of each document: titles, operating thresholds, fault types, diagnostics, corrective actions, and estimated repair times.
3. This confirms all 5 machine wiki files were properly indexed and are searchable.

**Step 3 — Test a query directly:**

1. At the bottom of the knowledge base detail page, find the **"Enter your message..."** input box.
2. Try a query directly against the knowledge base (bypassing the agent):
   > "What are the diagnostic steps for building drum vibration on a tire building machine?"
3. Observe the response — it should return structured results from the indexed documents with source references.
4. Try a cross-document query:
   > "Which machine has the longest estimated repair time for its most critical fault?"

<details>
<summary>💬 Why explore the Search service directly?</summary>

Understanding what's behind Foundry IQ helps you:
- **Debug** issues when the agent doesn't return expected results — is the content indexed? Is the query finding it?
- **Tune** retrieval by adjusting reasoning effort, retrieval instructions, or output configurations.
- **Verify** that your knowledge sources are synced and active.
- **Compare** the raw retrieval results with what the agent returns — this shows you how much the agent adds through its instructions and conversation context.

</details>

> [!TIP]
> You can also explore **Knowledge sources** (under Agentic retrieval) to see the blob storage connection details, and **Indexes** (under Search management) to see the underlying search index created by Foundry IQ.

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

- Verify the Web Search tool is enabled and saved.
- Try a query you know has current web results: "What is today's date?" or "Latest news about Azure."
- Web search may not be available in all regions — ask your coach about your deployment region.

</details>

<details>
<summary>Error: "too_many_requests: Too Many Requests"</summary>

- This means the model deployment has hit its rate limit (tokens per minute).
- **Quick fix**: Switch the agent to a different model. Edit the **Model** dropdown and select another deployment (e.g. switch from `gpt-4.1` to `gpt-4o-mini`, or vice versa).
- Wait 30–60 seconds before retrying — rate limits reset quickly.
- If the error persists across all models, reduce your prompt size or wait a few minutes before continuing.

</details>

<details>
<summary>Foundry IQ indexing seems stuck</summary>

- Indexing 5 small markdown files should complete in 2–5 minutes.
- If you get an error like *"Unable to retrieve blob container... using your managed identity"*, change the **Authentication type** to **API Key** and try again.
- If stuck beyond 10 minutes, verify the 5 `.md` files are actually in the `machine-wiki` container (go to Storage browser and check).
- Try deleting the knowledge base and recreating it with the connection.

</details>

<details>
<summary>Citations don't appear in agent responses</summary>

- Not all question types trigger citations. Try asking a very specific factual question: "What is the normal drum vibration threshold for a tire building machine?"
- Check that the knowledge base shows as "Connected" or "Active" on the agent.
- Some models follow citation instructions more reliably than others — try switching to gpt-4.1.

</details>

## 🧠 Conclusion and Reflection

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
