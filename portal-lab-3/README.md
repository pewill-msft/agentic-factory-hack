# Portal Lab 3: Tools

In this lab you'll extend your agent with built-in tools — File Search, Code Interpreter, and Web Search. No code required.

← [Portal Lab 2](../portal-lab-2/README.md) | **Portal Lab 3** | [Portal Lab 4](../portal-lab-4/README.md) →

**Expected duration**: 45 min

**Prerequisites**:

- [Portal Lab 2](../portal-lab-2/README.md) completed
- Access to the pre-provisioned **Storage Account** in the Azure Portal (set up in Challenge 0)
- Lab data files downloaded to your local machine (see below)

### Download Lab Files from GitHub

This lab requires several data files. Since you're working directly from the Foundry Portal (not in a codespaces), download the files from GitHub to your local machine first.

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

## 🎯 Objective

The goals for this lab are:

- Use **File Search** to let the agent query an uploaded document.
- Use **Code Interpreter** to have the agent generate data visualizations and return files.
- See the contrast of **Web Search** — before and after enabling live web access.

## 🧭 Context and Background

In Portal Lab 2, you created agents with custom instructions and memory. Now you'll give those agents access to **tools** — additional capabilities beyond the base model.

The Foundry Portal offers several built-in tools:

| Tool | What it does |
|------|-------------|
| **File Search** | Upload files (PDF, DOCX, etc.) and let the agent search their content to answer questions |
| **Code Interpreter** | Run Python code in a sandboxed environment — generate charts, perform calculations, process files |
| **Web Search** | Search the live web for current information and return cited results |
| **Memory** | Persist facts across conversations (you used this in Lab 2) |

## ✅ Tasks

### Task 1: File Search

In this task you'll create a fresh agent and upload a maintenance manual PDF so the agent can search it to answer questions.

1. In the Foundry Portal, navigate to **Build** → **Agents** and click **Create agent**.
2. Name the agent `TireToolsAgent` and click **Create**.
3. Configure it:
   - **Model**: Select `gpt-4.1` from the dropdown.
   - **Instructions**: Paste the following:

     ```
     You are a tire manufacturing assistant at Contoso Tires. You help
     technicians troubleshoot machine issues, recommend maintenance procedures,
     and answer questions about tire building, curing, and mixing processes.
     Always be specific about machine types and part numbers when possible.
     ```

4. Notice that **Web Search** is added as a tool by default. For this task, **remove it** — click the three-dot menu (⋮) next to **Web search** and select **Remove**. We want the agent to rely solely on the uploaded document, not the web. You'll re-enable Web Search later in Task 3.
5. Click **Save**.
6. In the agent's left-hand configuration panel, find the **Tools** section and click the **Upload files** button.
7. The **Attach files** dialog opens:
   - **Index option**: Leave as **Create a new index** (default).
   - **Vector index name**: A name is auto-generated for you (e.g. `index_honest_floor_8wyjngstpk`). You can keep the generated name or rename it to something meaningful like `contoso-tires-maintenance`.
   - Click **browse for files** (or drag and drop) and select the file [`portal-lab-3/data/contoso-tires-maintenance-manual.pdf`](./data/contoso-tires-maintenance-manual.pdf) from this repository.
8. The file appears in a table showing **Name**, **Size** (~120 KB), and **Status** (Uploading → Uploaded).
9. Click **Attach** to confirm.

> [!WARNING]
> After attaching the file, the portal creates a vector index in the background. This can take **several minutes**. You may see a yellow warning banner saying *"Tools not configured. The agent might not run as expected."* and the **Save** button may not work until the index is ready.
>
> **If this happens**: Navigate away from the agent (e.g. click **Models** in the left navigation), wait 2–3 minutes, then return to your `TireToolsAgent` and select **Add** **File Tool** under **Tools** again. The **File search** tool should now show with your index listed. Click **Save** to confirm the configuration.
>


10. Once the file is attached and indexed, ask the agent these questions:

<details>
<summary>💬 Sample prompts for File Search</summary>

**Prompt 1** — Maintenance schedule:
> "What is the quarterly maintenance task for the tire building machine TB-200/TB-300?"

**Prompt 2** — Safety procedure:
> "What are the steps in the pre-service safety checklist?"

**Prompt 3** — Part replacement:
> "Walk me through the procedure for replacing heating elements on the tire curing press. What tools and parts do I need?"

</details>

11. Observe the responses:
   - Does the agent cite the uploaded document?
   - Are the answers grounded in the document content (not general knowledge)?
   - Can you see which section of the document the answer came from?

12. **Inspect the response metadata and traces.** After the agent responds, look at the bottom of each chat message — you'll see a metadata bar showing the model used (e.g. `gpt-4.1`), response time, token count, and quality/safety scores (e.g. **AI Quality: 100%**, **Safety: 100%**).

13. Click the **Logs** button at the bottom of a response (or switch to the **Traces** tab at the top of the agent page) to open the trace view:
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


### Task 3: Web Search (Before & After)

This task demonstrates the value of web search by showing the same question with and without web access.

**Without web search:**

1. Make sure **Web Search** is **not** enabled on `TireToolsAgent` (you removed it in Task 1).
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

## 🚀 Go Further

> [!NOTE]
> Finished early? These optional exercises let you explore additional tool types.

### Task 4: Explore the Tool Catalog

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

## 🧠 Conclusion and Reflection

In this lab you learned to:
- **File Search**: Upload documents and let the agent answer questions from their content
- **Code Interpreter**: Generate visualizations and perform calculations within a sandboxed environment
- **Web Search**: Add live web access and see the clear before/after difference

Together, these tools transform a basic chat model into a capable assistant — grounded in your data, able to compute and visualize, and connected to the live web when needed.

Next up: [Portal Lab 4](../portal-lab-4/README.md) — where you'll build an enterprise knowledge base with **Foundry IQ** and learn the difference between tool-based search and RAG.
