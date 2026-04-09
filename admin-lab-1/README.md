# Admin Lab 1: Model Lifecycle — Versioning & Fine-Tuning

[← Admin Lab 0 — Environment](../admin-lab-0/README.md) | **Admin Lab 1** | [Admin Lab 2 — Evaluations & Observability →](../admin-lab-2/README.md)

This lab covers two essential model lifecycle activities: **fine-tuning a model** for the Contoso Tires manufacturing domain and **managing model versions and upgrades**. You'll start the fine-tuning job first, then use the waiting time to review versioning and upgrade workflows.

**Expected duration**: 45 min

**Prerequisites**:

- [Admin Lab 0](../admin-lab-0/README.md) completed (resource access verified)

> [!NOTE]
> These Admin Labs are **self-contained**. You do not need to have completed the Portal Labs or Coding Challenges. If this is your first time using the Foundry Portal model catalog and playground, don't worry — each task includes the navigation steps you need.

## 🎯 Objective

- Prepare training data and create a fine-tuning job from scratch.
- Deploy and compare a fine-tuned model with its base model.
- Understand model version policies — pinning, auto-upgrade, and deprecation timelines.
- Perform a model version upgrade on a live deployment.
- Verify model behavior before and after an upgrade.

## 🧭 Context and Background

### Model Versioning

Azure AI Foundry deploys specific **versions** of a model. Over time, model providers release newer versions, and older ones move through a lifecycle: **general availability**, **legacy**, **deprecated**, and finally **retired**. As an admin, this matters because the operational impact changes at each stage.

- In **Legacy**, the model is on a path toward retirement, but **existing deployments continue to work and you can still create new deployments**.
- In **Deprecated**, **you can no longer create any new deployments** for that model, but **existing deployments continue to work** until the retirement date.
- In **Retired**, the model is no longer usable. **New deployments are blocked and existing deployments stop working**, typically returning `404` errors.

This means the **deprecation date is the last point at which you can provision a new instance of that model**. If you wait until after deprecation, you can keep running what is already deployed, but you can't create new deployments for scale-out, disaster recovery, or new environments.

As an admin, you need to understand:

| Concept | Description |
|---------|-------------|
| **Version pinning** | Your deployment stays on a specific version until you manually upgrade |
| **Auto-upgrade** | Foundry automatically moves your deployment to the latest stable version |
| **Legacy date** | The model is marked for future retirement, but you can still create new deployments |
| **Deprecation date** | After this date, you can't create any new deployments, although existing deployments still run |
| **Retirement date** | After this date, the model is no longer usable and existing deployments stop working |

> [!IMPORTANT]
> For production planning, treat the **deprecation date** as your last safe date to provision a new deployment of that model. After that point, your migration work is limited to moving existing traffic off the old deployment before the retirement date arrives.

### Fine-Tuning

Fine-tuning creates a **custom version** of a base model by training it on your own data. This is one of three strategies for specializing model behavior:

| Strategy | When to Use | Contoso Tires Example |
|----------|-------------|----------------------|
| **Prompt engineering** | General tasks, quick iteration | *"You are a manufacturing maintenance expert…"* system prompt |
| **RAG (Retrieval-Augmented Generation)** | Access to specific documents at query time | Retrieving the tire curing press troubleshooting guide on demand |
| **Fine-tuning** | Consistent domain-specific behavior baked in | Model always responds with Contoso-specific thresholds, part numbers, and procedures without needing retrieval |
| **Memory** | Persistent personalization across conversations | Agent remembers that a technician specializes in curing presses and adjusts responses accordingly |

### Customization Methods

When you create a fine-tuning job in the Foundry Portal, you choose a **customization method**. The available methods depend on the base model you select:

| Method | What it does | When to use it |
|--------|-------------|----------------|
| **Supervised (SFT)** | Trains the model on labeled input/output pairs. The model learns to reproduce the assistant responses in your JSONL examples. | Best for most scenarios including task specialization, domain adaptation, and consistent response style — **this is the method we use in this lab**. |
| **Direct Preference Optimization (DPO)** | Aligns the model with human-preferred responses by training on *chosen* vs. *rejected* answer pairs. | Ideal for improving response quality, tone, or safety after an initial SFT pass. Supported on `gpt-4o`, `gpt-4.1`, `gpt-4.1-mini`, and `gpt-4.1-nano`. |
| **Reinforcement Fine-Tuning (RFT)** | Uses reward signals from model graders to optimize complex behaviors, rather than fixed answer pairs. | Best for reasoning-heavy tasks where correct outputs are hard to enumerate upfront. Currently supported on `o4-mini`. |

See the [Azure fine-tuning guide](https://learn.microsoft.com/azure/foundry/openai/how-to/fine-tuning?pivots=programming-language-studio) for DPO and RFT walkthroughs.

> [!TIP]
> Fine-tuning is most valuable when you need the model to consistently use domain-specific language, formats, or knowledge — even without retrieval tools attached. For Contoso Tires, this means the model "knows" that the curing press warning threshold is 178°C without needing to look it up.

It's also important to understand what fine-tuning **doesn't** do. A base LLM is effectively a **fixed snapshot** of the model as it existed when that version was trained and released. It doesn't keep getting smarter after release, and it doesn't automatically learn from your day-to-day prompts, chats, or new factory events. In other words, the model won't "pick up" a newly introduced Contoso machine, a changed spare-parts policy, or last week's maintenance bulletin unless you:

- provide that information at runtime through **RAG** or another tool,
- fine-tune a new model with updated training data, or
- move to a newer base model version and fine-tune again if needed.

This is why fine-tuning works best for **stable patterns** like response style, threshold interpretation, standard procedures, and recurring terminology. For **fast-changing facts**, such as current inventory, newly added machines, or the latest maintenance advisories, retrieval is usually the better fit.

**How to reason about training data vs. RAG/tools/memory:**

- Put examples in **training data** when you want the model's default behavior to stay consistent over time: response style, diagnostic flow, fault taxonomy, threshold interpretation, and standard safety procedures.
- Use **RAG** for reference material that may evolve but still benefits from document grounding at runtime: manuals, policies, troubleshooting guides, and approved procedures.
- Use **tools** for live operational state that should come from a current source of truth: inventory levels, supplier lead times, maintenance windows, technician availability, work orders, and scheduler scoring logic.
- Use **[memory](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/what-is-memory?tabs=conversational-agent)** for user-specific or session-specific context that should persist across conversations: a technician's role, their assigned machines, preferences, and prior diagnostic history. Memory is managed per-agent and builds up over time through interactions — see [Portal Lab 2](../portal-lab-2/README.md) for a hands-on walkthrough.
- If a fact becoming stale would lead to a bad operational decision, it should be retrieved or looked up at runtime rather than baked into the fine-tuned model.

### Pre-Deployed Models

Your project has three models already deployed:

| Model | Type | Typical Use |
|-------|------|-------------|
| `gpt-4.1` | Chat completion (flagship) | High-quality reasoning, function calling, structured output |
| `gpt-4o-mini` | Chat completion (cost-efficient) | High-volume tasks, low-latency responses |
| `text-embedding-3-large` | Embedding | Vector search, similarity matching |

---

## ✅ Tasks

### Task 1: Review the Training Data

Fine-tuning requires a dataset of example conversations that teach the model your domain-specific patterns. We've prepared a dataset of Contoso Tires manufacturing Q&A pairs.

1. Open the training data file: [`training-data.jsonl`](./data/training-data.jsonl)
2. Review the format — each line is a JSON object with a `messages` array containing:
   - A `system` message (the manufacturing expert persona)
   - A `user` message (a question about tire manufacturing)
   - An `assistant` message (the ideal response with Contoso-specific details)

Here's an example entry:

```json
{
  "messages": [
    {"role": "system", "content": "You are a Contoso Tires manufacturing maintenance expert..."},
    {"role": "user", "content": "Our tire curing press TC-100 is showing temperatures of 179°C. What could be wrong?"},
    {"role": "assistant", "content": "A reading of 179°C on the curing press exceeds the 178°C threshold and indicates a curing_temperature_excessive fault. Likely causes include: heating element malfunction, temperature sensor drift..."}
  ]
}
```

**💬 What to notice about the training data:**
- Responses include **specific thresholds** (178°C, 3.0 mm/s, 230 N)
- Responses reference **part numbers** (TCP-HTR-4KW, TBM-BRG-6220)
- Responses use **standard fault type names** (curing_temperature_excessive, building_drum_vibration)
- Some examples intentionally teach the model to **consult runtime systems** for live inventory, scheduling, or supplier facts instead of memorizing brittle operational values
- There are approximately **30 examples** covering all five machine types

> [!TIP]
> For production fine-tuning, you'd typically want 50–100+ high-quality examples. The quality and consistency of your training data matters more than quantity. Each example should represent the exact response style and depth you want the model to produce.

### Task 2: Create a Fine-Tuning Job

First, download the training data file to your local machine so you can upload it to the Foundry Portal:

1. Navigate to [`admin-lab-1/data/training-data.jsonl`](./data/training-data.jsonl) in the GitHub repository.
2. Click the **Download raw file** button (↓ icon) in the top-right of the file view.
3. Save it somewhere you can find it (e.g., your Downloads folder).

Now create the fine-tuning job:

1. In the Foundry Portal, click **Build** in the top navigation bar.
2. Select **Fine-tune** in the left sidebar.
3. Click the **Fine-tune** button.
4. Fill in the **Basic details** section:

| Setting | Value |
|---------|-------|
| **Select model** | `gpt-4o-mini` |
| **Customization method** | `Supervised` (see [Customization Methods](#customization-methods) above) |
| **Training type** | `Standard` (keeps data in your resource region) |
| **Select data source** | Change from *Curated sample datasets* to **Upload new dataset**, then upload [`training-data.jsonl`](./data/training-data.jsonl) |
| **Validation data** | Leave collapsed (optional for this lab) |
| **Suffix** | Enter a descriptive suffix, e.g., `contoso-tires-maintenance` — this is appended to the model name to identify your fine-tuned version |

5. Leave **Automatically deploy model after job completion** toggled **off**. You'll deploy the model manually in Task 7.
6. Expand **Additional configuration** and set:

| Setting | Value |
|---------|-------|
| **Seed** | Leave as `Random` (controls reproducibility; a fixed integer gives deterministic results across runs) |
| **Batch size** | Leave on `Default` |
| **Number of epochs** | Select `Custom` and enter `1` (number of passes through the training data; range 1–10. We use `1` in this lab to keep training time shorter) |
| **Learning rate multiplier** | Leave on `Default` |

7. Click **Submit** to launch the fine-tuning job.

**✅ Expected result**

The fine-tuning job appears in the list with status "Running" or "Queued".

> [!NOTE]
> Fine-tuning typically takes **30–45 minutes** depending on queue time and data size, and the job can sit in "Queued" state for several minutes before it starts running. The estimated finish time shown in the portal may underestimate — plan for the full 30–45 minutes. **Move on to Task 4 now** and come back periodically to check progress.

### Task 3: Monitor Training Progress

This task is something you'll **check back on periodically** while working through the remaining tasks. There's no need to wait here — just revisit the fine-tuning job page every 10–15 minutes to track progress.

1. Click on your fine-tuning job to open its detail page.
2. Check the **Logs** tab to see the job's event history — this shows timestamped status messages such as "Job enqueued", "Preprocessing completed", "Finetuning started", etc. This is helpful when the job appears stuck or metrics haven't appeared yet.
3. On the **Monitor** tab, you can track:
   - **Current train loss** — a measure of how wrong the model is on the current training batch; lower is better and it should generally decrease over time
   - **Current train mean token accuracy** — the percentage of tokens in the current training batch that the model predicted correctly; higher is better and it should generally increase over time
   - **Job status** — Queued → Running → Succeeded

> [!NOTE]
> It can take **several minutes** after the job enters `Running` before any training metrics appear. During that time, values such as current train loss or token accuracy may show as blank (`--`). This is normal — check the **Logs** tab to confirm the job is progressing.

**💬 What to look for in the training metrics:**

| Metric | Good Sign | Warning Sign |
|--------|-----------|--------------|
| Training loss | Steadily decreasing | Flat or increasing |
| Train mean token accuracy | Steadily increasing | Flat or decreasing |
| Validation loss | Decreasing alongside training loss | Increasing while training loss decreases (overfitting) |
| Final loss value | Below 1.0 | Above 2.0 |

> [!TIP]
> If you don't want to wait, here's what a completed fine-tuning job typically looks like. Even with a single epoch, you should still expect to see the loss trend in the right direction as training progresses.

**✅ Completed job example**

A completed fine-tuning job showing the training metrics and a successful completion status.

![Fine-Tuning Complete](./images/fine-tuning-complete.png)

### Task 4: Inspect Deployed Model Versions

While the fine-tuning job runs, let's review the currently deployed model versions.

1. Open the **Foundry Portal** at [ai.azure.com](https://ai.azure.com) if you aren't already there.
2. Click **Operate** in the top navigation bar, then select **Assets** → **Models** in the left sidebar.
3. Click on the **gpt-4.1** deployment to open its details.
4. Review the following information:
   - **Deployment type** — for example `GlobalStandard`
   - **Provisioning state** — confirm it shows `Succeeded`
   - **Version upgrade policy** — for example `OnceNewDefaultVersionAvailable`
   - **Tokens per Minute Rate Limit** and **Requests per Minute Rate Limit**
   - **Model name** and **Model version** — for example `gpt-4.1` and `2025-04-14`
   - **Lifecycle status** — for example `GenerallyAvailable`
   - **Retirement date**
5. Repeat for **gpt-4o-mini** and **text-embedding-3-large**.

**💬 What to observe:**
- Note which **model version** each deployment is running. This helps you understand exactly which release is currently deployed in your environment.
- Note the **lifecycle status** and **retirement date**. These tell you how long the current deployment remains supported.
- Note the **version upgrade policy** value exactly as shown in the portal. You'll interpret what that means in the next task.
- Check the **TPM** and **RPM** limits so you understand the current capacity allocated to the deployment.

**✅ Expected result**

The deployment detail page showing deployment type, provisioning state, version upgrade policy, rate limits, model version, lifecycle status, and retirement date.

![Model Version Details](./images/model-version-details.png)

### Task 5: Explore Model Update Policies

1. While viewing a deployment's details, look for the **version upgrade policy** setting.
2. In this environment, you may see a policy value such as **`OnceNewDefaultVersionAvailable`**.
3. Understand the practical meaning of the common policies:
   - **`OnceNewDefaultVersionAvailable`** — the deployment is configured to upgrade when a new default version becomes available for that model family.
   - **`Once the current version expires`** — the deployment stays on the current version longer and only moves when that version reaches expiry.
   - **`Opt out of automatic model version upgrades`** — the deployment stays on the current version until you explicitly change it. In that case, you're responsible for upgrading before retirement.

> [!IMPORTANT]
> In production environments, many organizations prefer pinned or tightly controlled upgrade behavior so they can test new versions before upgrading. Automatic upgrade policies are convenient for development, but they can introduce unexpected behavior changes if you don't validate first.

4. Consider: which policy would you recommend for Contoso Tires' production agents that diagnose machine faults? Why?

**💬 Things to consider:**
- A version upgrade could change how the model interprets fault thresholds or diagnostic procedures.
- Pinning gives you control but requires monitoring deprecation timelines.
- A good practice is to use a controlled policy for production deployments and a more automatic policy on a separate test deployment for validation.

### Task 6: Change the Version Upgrade Policy

In this task you'll edit the `gpt-4.1` deployment to understand the available settings and pin it to its current version — the recommended approach for production workloads.

1. While viewing the `gpt-4.1` deployment details page, click **Edit**.
2. In the **Update deployment** panel, review the available fields.

#### Deployment types

The **Deployment type** dropdown controls where and how inference requests are processed. The deployment types you may see include:

| Deployment Type | Description |
|----------------|-------------|
| **Global Standard** | Pay-per-token. Requests are routed globally for the highest available rate limits. Data storage stays in your resource's region. |
| **Data Zone Standard** | Pay-per-token. Requests stay within the geographic data zone (e.g., US, EU) for data residency requirements. |
| **Standard** | Pay-per-token. Requests are processed in the region where the resource is deployed. |
| **Global Provisioned Throughput** | Reserved capacity billed per provisioned throughput unit (PTU). Routed globally for best availability. |
| **Data Zone Provisioned Throughput** | Reserved PTU capacity within a geographic data zone. |

> [!TIP]
> For this lab, leave the deployment type as **Global Standard**. In production, choose based on your data residency requirements and whether you need guaranteed throughput (provisioned) or flexible pay-per-token (standard). See [Deployment types](https://learn.microsoft.com/en-us/azure/foundry/foundry-models/concepts/deployment-types) for details.

#### Change the upgrade policy

3. Expand **Model version settings** and note the current **Model version** (e.g., `2025-04-14`).
4. Open the **Model version upgrade policy** dropdown. You'll see three options:
   - **Upgrade once new default version becomes available** — the deployment auto-upgrades when a new default version is released
   - **Once the current version expires** — the deployment stays on the current version and only upgrades when it reaches expiry
   - **Opt out of automatic model version upgrades** — the deployment stays on the current version until you manually change it
5. Select **Opt out of automatic model version upgrades**.

#### Adjust the rate limit

6. Scroll down to the **Tokens per Minute Rate Limit** slider. This controls the maximum number of tokens the deployment can process per minute. The slider shows your current allocation out of the total quota available for your subscription and region.
7. Lower the rate limit to **30,000** tokens per minute. In a shared workshop environment with multiple deployments, it's good practice to avoid allocating more capacity than you need — this leaves room for other deployments (including the fine-tuned model you'll deploy in Task 7).

> [!TIP]
> In production, set rate limits based on expected traffic. Over-allocating TPM on one deployment can starve other deployments sharing the same quota. Monitor actual usage and adjust as needed.

8. Click **Save** to apply both changes.

> [!IMPORTANT]
> For production agents like Contoso Tires' fault diagnosis agents, opting out of automatic upgrades is the recommended practice. A model version change could alter how the model interprets fault thresholds or diagnostic procedures. By pinning the version, you can test new versions on a separate deployment before upgrading production.

**✅ Expected result**

The deployment's version upgrade policy now shows **Opt out of automatic model version upgrades**. The model version remains unchanged — you've only changed when (and whether) it auto-upgrades.

> [!NOTE]
> When you opt out, you're responsible for monitoring deprecation timelines and upgrading before the retirement date. A good practice is to keep a separate test deployment with automatic upgrades enabled so you can validate new versions early.

### Task 7: Deploy and Test the Fine-Tuned Model

Once the fine-tuning job from Task 2 has completed, come back here to deploy and test the model.

1. In the Foundry Portal, click **Build** in the top navigation bar.
2. Select **Fine-tune** in the left sidebar.
3. Click on your completed fine-tuning job — its status should show **Succeeded**. If it still shows **Running**, continue with the other tasks and check back in a few minutes.
4. Click **Deploy**.
5. Give the deployment a name, e.g., `contoso-tires-ft`, and configure the **Tokens per Minute Rate Limit** to **50,000**. Click **Deploy**.
6. Once deployed, select the **Playground** tab.
7. Click **Compare models** in the upper-right corner of the playground.
8. In the comparison panel, add the base deployment **`gpt-4o-mini`** so you can see both models side by side.
9. In the **Setup** tab, make sure both sides use the same instructions.
10. Select **Setup** and use this prompt:

   ```
   You are a manufacturing maintenance expert at Contoso Tires. Provide concise, accurate diagnostic and maintenance guidance. Reference specific thresholds, part numbers, and procedures when applicable.
   ```

11. In the **Chat** tab, send the following prompt once and compare the two responses side by side.
> What are the most common causes of excessive drum vibration in a tire building machine, and what maintenance steps should be taken?

> [!NOTE]
> It might take a few minutes for the fine-tuned model to get ready.

> [!TIP]
> With **Sync chat input and setup** enabled in setup the same instructions and prompts are applied to both models.

**💬 What to compare:**

| Aspect | Base Model (gpt-4o-mini) | Fine-Tuned Model |
|--------|--------------------------|------------------|
| Mentions specific thresholds (3.0 mm/s) | Sometimes, if prompted | Consistently includes thresholds |
| References part numbers (TBM-BRG-6220) | Rarely | Frequently includes part numbers |
| Uses standard fault type names | No | Yes (building_drum_vibration) |
| Response style | General manufacturing advice | Contoso Tires-specific procedures |

12. Try a few more prompts to see the difference:

> Machine TC-100 curing cycle is taking 16 minutes. What should I check?

> What spare parts should we always keep minimum stock for?

**✅ Expected result**

The fine-tuned model provides responses with Contoso Tires-specific details (thresholds, part numbers, fault types) even without those details in the system prompt.

![Fine-Tuned Comparison](./images/fine-tuned-comparison.png)

## 🚀 Go Further

- **Experiment with hyperparameters**: Create a second fine-tuning job with different settings (e.g., 5 epochs, learning rate multiplier 0.5) and compare the results.
- **Create a validation dataset**: Split the training data — use 80% for training and 20% for validation. This helps you detect overfitting.
- **A/B deploy model versions**: Keep both the base and fine-tuned model deployed side-by-side. Send the same prompts to both and track which gives better results for your use case.
- **Combine fine-tuning with RAG**: A fine-tuned model that also has access to retrieval tools can provide both consistent domain knowledge (from fine-tuning) and up-to-date information (from retrieval).

## 🧠 Conclusion

You've completed two critical model lifecycle tasks:

- Reviewed a domain-specific training dataset in JSONL format
- Created a fine-tuning job from scratch in the portal
- Monitored training progress and interpreted loss metrics
- Inspected deployed model versions and their upgrade policies
- Reviewed the practical trade-offs between automatic upgrade policies and controlled/manual upgrade behavior
- Reviewed the deployment edit experience and the available version upgrade policy options
- Deployed and compared the fine-tuned model against the base model

**Next**: [Admin Lab 2 — Evaluations & Observability](../admin-lab-2/README.md)
