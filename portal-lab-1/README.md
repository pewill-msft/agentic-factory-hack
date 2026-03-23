# Portal Lab 1: Models

In this lab you'll explore the Foundry model catalog, deploy a new model, and test it in the playground — all through the Foundry Portal. No code required.

← [Portal Lab 0](../portal-lab-0/README.md) | **Portal Lab 1** | [Portal Lab 2 →](../portal-lab-2/README.md)

**Expected duration**: 45 min

**Prerequisites**:

- [Portal Lab 0](../portal-lab-0/README.md) completed (environment validated, 3 models confirmed)

## 🎯 Objective

The goals for this lab are:

- Navigate the model catalog and use filters to discover models by provider, capability, and deployment type.
- Understand model cards: details, benchmarks, and existing deployments.
- Deploy a new model and test it in the playground with a manufacturing-themed system prompt.
- Experiment with model parameters and compare models side by side.

## 🧭 Context and Background

The Foundry Portal provides a curated **model catalog** with hundreds of models from Microsoft, OpenAI, Meta, Mistral, and other providers. Each model has a **model card** with documentation, benchmarks, and deployment options.

You already have three models deployed from Challenge 0. In this lab you'll explore what's available, deploy a fourth model, and learn to test models in the **playground** before integrating them into agents or applications.

### Key concepts

| Concept | Description |
|---------|-------------|
| **Model catalog** | Searchable directory of all available models, with filters for provider, task type, and licensing |
| **Model card** | Detail page for a model: description, benchmarks, use cases, limitations, and pricing |
| **Deployment** | An instance of a model provisioned in your project, with a specific endpoint and rate limits |
| **Playground** | Interactive chat UI to test a deployed model with system prompts, parameters, and file attachments |
| **Benchmarks** | Performance comparisons across models on standardized tasks (quality, speed, cost) |

## ✅ Tasks

### Task 1: Browse the Model Catalog

1. In the Foundry Portal, navigate to **Discover** → **Models**.
2. You'll see the full model catalog. Explore the available **filters**:
   - **Provider / Collection**: OpenAI, Meta, Microsoft, Mistral, Cohere, and others
   - **Capabilities**: Chat completion, Embeddings, Image generation, Vision, Audio
   - **Deployment options**: Global Standard, Standard, Provisioned, Serverless API
3. Try these filter combinations and note the results:
   - Filter by **OpenAI** provider → see all OpenAI models available
   - Filter by **Vision** capability → see which models support image input
   - Filter by **Serverless API** deployment → see pay-per-token models

<details>
<summary>💬 What to look for</summary>

- The catalog contains models from multiple providers — not just OpenAI.
- Different models support different deployment types and capabilities.
- Some models are free to try, others require specific deployment types.
- Notice the model family groupings (e.g., GPT-4.1, GPT-4o, Phi-4, Llama).

</details>

### Task 2: Explore a Model Card

1. Search for or click on **gpt-4.1** in the catalog.
2. On the model card page, explore the tabs:
   - **Details**: Description, capabilities, supported regions, context window size, pricing
   - **Benchmarks**: Quality/speed/cost comparisons against other models
   - **Existing Deployments**: See the deployment you already have in your project
3. Take note of the context window size and the benchmark scores.

<details>
<summary>💬 Things to notice</summary>

- The model card shows the maximum context window (input + output tokens).
- Benchmarks compare the model against peers on standardized tasks.
- The "Existing Deployments" tab shows deployments already in your project — useful to avoid duplicates.

</details>

### Task 3: Compare & Benchmark Models

1. From the model catalog, click **Browse leaderboard** (or navigate to the benchmarks section).
2. The leaderboard ranks models on quality, speed, and cost metrics.
3. Select two models (e.g., **gpt-4.1** and **gpt-4o-mini**) and click **Compare**.
4. Review the side-by-side comparison: benchmark scores, pricing, context window, and supported capabilities.

<details>
<summary>💬 What to observe</summary>

- Larger models typically score higher on quality benchmarks but cost more per token.
- Smaller models (like gpt-4o-mini) are faster and cheaper, with a quality trade-off.
- The comparison view helps you choose the right model for your use case: balance quality, speed, and cost.

</details>

### Task 4: Deploy a New Model

1. Navigate to **Discover** → **Models** and search for **gpt-5-mini**.
2. Click on the model card, then click **Deploy**.
3. In the deployment wizard:
   - Leave the **deployment name** as the suggested default (or customize it).
   - Select the **deployment type** (Global Standard is fine for this lab).
   - Review the rate limits and pricing summary.
4. Click **Deploy** and wait for provisioning to complete.
5. Once the status shows **Succeeded**, note the deployment name — you'll use it in the playground.

<details>
<summary>✅ You should see something similar to this</summary>

The deployment page shows your new model with status "Succeeded", along with the endpoint URL and rate limits.

<!-- TODO: Replace with actual screenshot -->
![Model Deployment](./images/placeholder-model-deployment.png)

</details>

> [!TIP]
> Deployment typically takes 1–3 minutes. If it takes longer, refresh the page. If it fails due to quota, ask your coach about region capacity.

### Task 5: Test in the Playground

1. From the deployment page, click **Open in playground** (or navigate to **Build** → **Playgrounds** → **Chat playground** and select your new deployment).
2. In the **System prompt** box, enter:

   ```
   You are a manufacturing quality expert at Contoso Tires. You help engineers
   diagnose tire defects, recommend quality improvements, and explain
   manufacturing processes. Be specific about machine types, part numbers,
   and procedures when possible. Use clear, structured responses.
   ```

3. Click **Apply** to save the system prompt.
4. In the chat input, try these test messages one at a time:

<details>
<summary>💬 Sample prompts to try</summary>

**Prompt 1** — General knowledge:
> "What are common causes of sidewall separation in radial tires?"

**Prompt 2** — Process explanation:
> "Explain the vulcanization process and how temperature affects tire quality."

**Prompt 3** — Troubleshooting:
> "Our tire building machine TB-200 is showing drum vibration readings of 4.2 mm/s. Normal threshold is 3.0 mm/s. What should we check and in what order?"

</details>

5. Now experiment with the **parameters panel** (usually on the right side or in settings):
   - **Temperature**: Try 0.2 (more focused) vs. 1.0 (more creative). Send the same question with each setting.
   - **Top P**: Adjust from 0.9 to 0.5 and observe the difference.
   - **Max tokens**: Set a low limit (e.g., 100) and see how the model truncates its response.

<details>
<summary>💬 What to observe about parameters</summary>

- **Low temperature** (0.1–0.3): More deterministic, repeatable, focused answers. Good for factual/technical responses.
- **High temperature** (0.8–1.2): More varied, creative responses. Each run may differ.
- **Max tokens**: Hard limit on response length. The model stops mid-sentence if it hits the limit.
- For manufacturing/technical use cases, **lower temperature** (0.2–0.5) is typically preferred.

</details>

## 🚀 Go Further

> [!NOTE]
> Finished early? These optional exercises let you explore additional playground features.

### Task 6: Compare Models in Playground

1. In the playground, look for the **Compare** button (or open a second playground tab).
2. Select your newly deployed **gpt-5-mini** on one side and **gpt-4o-mini** on the other.
3. Set the same system prompt on both sides.
4. Send the same question to both models simultaneously:
   > "Create a step-by-step root cause analysis procedure for inconsistent cure times on a tire curing press."
5. Compare the responses: depth, structure, and specificity.

### Task 7: Multimodal Input (Vision)

1. In the playground, select a vision-capable model deployment (e.g., **gpt-4.1** supports vision).
2. In the chat input, click the **attachment** icon to upload an image.
3. Upload a tire defect image (or any manufacturing-related image you have available).
4. Ask: "Analyze this image. What type of tire defect do you see, and what manufacturing process is most likely responsible?"
5. Observe how the model interprets visual input alongside text.

> [!TIP]
> If you don't have a tire defect image handy, try uploading any photo and asking the model to describe it in a manufacturing context. The goal is to experience multimodal input.

## 🛠️ Troubleshooting and FAQ

<details>
<summary>Model deployment fails with a quota error</summary>

- Your subscription may have reached its deployment quota for the region.
- Try deploying with a lower rate limit (e.g., reduce tokens-per-minute).
- Ask your coach if an alternative region or deployment type is available.

</details>

<details>
<summary>The model I want isn't in the catalog</summary>

- Not all models are available in all regions. Check the model card for supported regions.
- Some models require specific subscription types or enrollment in a preview program.

</details>

<details>
<summary>Playground responses are slow or timing out</summary>

- Try reducing the **max tokens** parameter.
- Check if the model is still provisioning (status should be "Succeeded").
- High demand on shared quota can cause temporary slowdowns.

</details>

<details>
<summary>Compare feature is not visible in the playground</summary>

- The Compare feature may appear as a button in the playground toolbar or under a menu.
- If not available, you can manually open two browser tabs — one for each model — and send the same prompt in both.

</details>

## 🧠 Conclusion

In this lab you learned to:
- **Discover** models through the catalog with filters and benchmarks
- **Deploy** a model to your project with a specific configuration
- **Test** models interactively in the playground with system prompts and parameter tuning
- **Compare** models to understand quality, style, and cost trade-offs

These skills form the foundation for the next lab, where you'll create agents that use these models.

**Next**: [Portal Lab 2 — Agents](../portal-lab-2/README.md)
