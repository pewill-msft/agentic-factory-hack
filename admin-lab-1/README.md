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

> [!TIP]
> Fine-tuning is most valuable when you need the model to consistently use domain-specific language, formats, or knowledge — even without retrieval tools attached. For Contoso Tires, this means the model "knows" that the curing press warning threshold is 178°C without needing to look it up.

It's also important to understand what fine-tuning **doesn't** do. A base LLM is effectively a **fixed snapshot** of the model as it existed when that version was trained and released. It doesn't keep getting smarter after release, and it doesn't automatically learn from your day-to-day prompts, chats, or new factory events. In other words, the model won't "pick up" a newly introduced Contoso machine, a changed spare-parts policy, or last week's maintenance bulletin unless you:

- provide that information at runtime through **RAG** or another tool,
- fine-tune a new model with updated training data, or
- move to a newer base model version and fine-tune again if needed.

This is why fine-tuning works best for **stable patterns** like response style, threshold interpretation, standard procedures, and recurring terminology. For **fast-changing facts**, such as current inventory, newly added machines, or the latest maintenance advisories, retrieval is usually the better fit.

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
- There are approximately **30 examples** covering all five machine types

> [!TIP]
> For production fine-tuning, you'd typically want 50–100+ high-quality examples. The quality and consistency of your training data matters more than quantity. Each example should represent the exact response style and depth you want the model to produce.

### Task 2: Create a Fine-Tuning Job

1. In the Foundry Portal, click **Build** in the top navigation bar.
2. Select **Fine-tuning** in the left sidebar.
3. Click **+ Fine-tune a model**.
4. Configure the fine-tuning job:

| Setting | Value |
|---------|-------|
| **Base model** | `gpt-4o-mini` |
| **Training data** | Upload [admin-lab-1/data/training-data.jsonl](admin-lab-1/data/training-data.jsonl) |
| **Validation data** | Leave empty (optional for this lab) |
| **Epochs** | `3` (number of passes through the training data) |
| **Learning rate multiplier** | `1.0` (default — scales the base learning rate) |
| **Batch size** | `auto` (let the system optimize) |

5. Give the fine-tuning job a descriptive name, e.g., `contoso-tires-maintenance-ft`.
6. Click **Start** to launch the fine-tuning job.

**✅ Expected result**

The fine-tuning job appears in the list with status "Running" or "Queued".

![Fine-Tuning Job Created](./images/fine-tuning-job-created.png)

> [!NOTE]
> Fine-tuning typically takes **30–60 minutes** depending on the data size and selected hyperparameters. While the job runs, continue with the remaining tasks in this lab and come back to deploy the fine-tuned model later.

### Task 3: Monitor Training Progress

1. Click on your fine-tuning job to open its detail page.
2. While the job is running, you can monitor:
   - **Training loss** — should decrease over epochs (the model is learning)
   - **Validation loss** (if validation data was provided) — monitors overfitting
   - **Job status** — Queued → Running → Succeeded
   - **Estimated completion time**

**💬 What to look for in the training metrics:**

| Metric | Good Sign | Warning Sign |
|--------|-----------|--------------|
| Training loss | Steadily decreasing | Flat or increasing |
| Validation loss | Decreasing alongside training loss | Increasing while training loss decreases (overfitting) |
| Final loss value | Below 1.0 | Above 2.0 |

> [!TIP]
> If you don't want to wait, here's what a completed fine-tuning job typically looks like. The training loss curve should show a downward trend across the 3 epochs.

**✅ Completed job example**

A completed fine-tuning job showing the training loss curve decreasing over 3 epochs.

![Fine-Tuning Complete](./images/fine-tuning-complete.png)

### Task 4: Inspect Deployed Model Versions

While the fine-tuning job runs, switch to the model deployment view and review the currently deployed versions.

1. Open the **Foundry Portal** at [ai.azure.com](https://ai.azure.com) if you aren't already there.
2. Click **Operate** in the top navigation bar, then select **Assets** → **Models** in the left sidebar.
3. Click on the **gpt-4.1** deployment to open its details.
4. Review the following information:
   - **Model version** — the specific version number currently deployed
   - **Version upgrade policy** — is it set to auto-upgrade or pinned?
   - **Deprecation date** and **Retirement date** (if shown)
   - **Rate limits** — tokens per minute (TPM) and requests per minute (RPM)
5. Repeat for **gpt-4o-mini** and **text-embedding-3-large**.

**💬 What to observe:**
- Note which version each model is running. Write these down — you'll compare after upgrading.
- Check if any model shows an upcoming deprecation date. This tells you how much time you have before an upgrade is required.

**✅ Expected result**

The deployment detail page showing the model version, version policy, and rate limits.

![Model Version Details](./images/model-version-details.png)

### Task 5: Explore Model Update Policies

1. While viewing a deployment's details, look for the **version upgrade policy** setting.
2. Understand the two options:
   - **Auto-upgrade (default)** — Foundry will automatically move your deployment to the latest stable version when available. You receive a notification but don't need to take action.
   - **Pinned** — Your deployment stays on the current version until you manually upgrade. You're responsible for upgrading before the retirement date.

> [!IMPORTANT]
> In production environments, many organizations prefer **pinned** versions so they can test new versions before upgrading. Auto-upgrade is convenient for development but can introduce unexpected behavior changes.

3. Consider: which policy would you recommend for Contoso Tires' production agents that diagnose machine faults? Why?

**💬 Things to consider:**
- A version upgrade could change how the model interprets fault thresholds or diagnostic procedures.
- Pinning gives you control but requires monitoring deprecation timelines.
- A good practice is to pin production deployments and auto-upgrade a separate test deployment for validation.

### Task 6: Upgrade a Model Deployment

1. Navigate back to **Operate** → **Assets** → **Models**.
2. Click on the **gpt-4o-mini** deployment.
3. Look for an **Upgrade** or **Update version** option.
   - If a newer version is available, you'll see it listed as an upgrade option.
   - If the deployment is already on the latest version, the option may be grayed out.
4. If an upgrade is available:
   a. Click **Upgrade** and review the version comparison (current vs. new).
   b. Note any documented changes between versions.
   c. Confirm the upgrade.
5. After the upgrade completes, verify the new version number in the deployment details.

> [!NOTE]
> If no newer version is available for gpt-4o-mini, that's fine — the key learning is understanding where the upgrade workflow lives and what information it provides. You can also try this on gpt-4.1 instead.

**✅ Expected result**

The deployment now shows the updated model version.

![Model Upgrade](./images/model-upgrade.png)

### Task 7: Verify in the Playground

Let's confirm the upgraded model still produces good results for our manufacturing domain.

1. Navigate to **Build** → **Playgrounds** → **Chat playground** in the Foundry Portal.
2. Select the **gpt-4o-mini** deployment (the one you just upgraded).
3. Set the **System message** to:

```
You are a manufacturing maintenance expert at Contoso Tires. Provide concise, accurate diagnostic and maintenance guidance. Reference specific thresholds, part numbers, and procedures when applicable.
```

4. Send this test prompt:

> What are the most common causes of excessive drum vibration in a tire building machine, and what maintenance steps should be taken?

5. Review the response — it should mention vibration thresholds, bearing inspection, and alignment procedures.

**💬 What to observe:**
- Does the response quality match what you'd expect? Compare with your memory of pre-upgrade behavior.
- This is the kind of validation you'd do before approving a version upgrade for production.

**✅ Expected result**

The model responds with relevant maintenance guidance including vibration thresholds and diagnostic steps.

![Playground Verification](./images/playground-verification.png)

### Task 8: Deploy and Test the Fine-Tuned Model

Once the fine-tuning job completes (or if a pre-fine-tuned model has been provided):

1. From the completed fine-tuning job page, click **Deploy**.
2. Give the deployment a name, e.g., `contoso-tires-ft`.
3. Configure rate limits as needed and click **Deploy**.
4. Once deployed, go to **Build** → **Playgrounds** → **Chat playground**.
5. Select your new fine-tuned deployment (`contoso-tires-ft`).
6. Send the **same prompt** you used in Task 7:

> What are the most common causes of excessive drum vibration in a tire building machine, and what maintenance steps should be taken?

7. Compare the response with the base model response from Task 7.

**💬 What to compare:**

| Aspect | Base Model (gpt-4o-mini) | Fine-Tuned Model |
|--------|--------------------------|------------------|
| Mentions specific thresholds (3.0 mm/s) | Sometimes, if prompted | Consistently includes thresholds |
| References part numbers (TBM-BRG-6220) | Rarely | Frequently includes part numbers |
| Uses standard fault type names | No | Yes (building_drum_vibration) |
| Response style | General manufacturing advice | Contoso Tires-specific procedures |

8. Try a few more prompts to see the difference:

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
- Understood the trade-offs between auto-upgrade and version pinning
- Performed (or explored) a model version upgrade
- Validated model behavior after an upgrade
- Deployed and compared the fine-tuned model against the base model

**Next**: [Admin Lab 2 — Evaluations & Observability](../admin-lab-2/README.md)
