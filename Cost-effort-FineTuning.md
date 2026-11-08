# Clinical Q&A Product: Comparing Three Approaches (Cost & Effort)

*Prices as of November 2026. All figures are estimates; verify current rates before budgeting.*

## Two clarifications

- **Claude Code** is a tool for writing software, not for answering patients. Patient Q&A uses the **Claude API** (directly or through AWS Bedrock). Claude Code can still help developers build the app.
- **MedGemma can't be trained on Tinker.** Tinker only supports models in its own catalog, and Gemma isn't one of them. Approach 2 therefore uses Qwen or gpt-oss.

---

## Comparison

| | **1. Fine-tune MedGemma yourself, host on AWS** | **2. Train with Tinker, host on AWS** | **3. Claude / Gemini API + RAG** |
|---|---|---|---|
| **Model** | MedGemma 1.5 4B (or 27B) | Qwen3.6 or gpt-oss-20b | Claude Sonnet 5 / Haiku, or Gemini Pro / Flash |
| **Who trains it** | Your team, on your AWS GPUs | Your team writes the code; Tinker runs the GPUs | Nobody. Knowledge is added through retrieval (RAG) and prompts |
| **Who hosts it** | You (vLLM on EC2) | You (vLLM on EC2) | The provider |
| **Data leaves your AWS account?** | No | Training data goes to Tinker (US) | Yes, to the provider. Claude via Bedrock can stay inside India |
| **Time to production** (health-info product, no device licence) | 6–9 months | 5–8 months | 3–6 months |
| **Team effort** | 45–65 person-months | 40–60 person-months | 20–35 person-months |
| **Skills needed** | ML training, GPU ops, MLOps | ML training (lighter), GPU ops | Backend, RAG, prompt and evaluation work |
| **One-time training compute** | ~$500–2,000 for all experiments | ~$100–1,000 for all experiments | $0 |
| **Monthly cost, 100k questions** | ~$1,200–2,700 (fixed) | ~$1,200–2,700 (fixed) | ~$400–900 |
| **Monthly cost, 1M questions** | ~$1,200–5,400 | ~$1,200–5,400 | ~$4,000–9,000 |
| **Ongoing maintenance** | High: GPUs, scaling, retraining, security patches | High: same as Approach 1 | Low–medium: re-test when the provider changes models |
| **Answer quality (starting point)** | Good medical base, small model | General base; medical knowledge comes from your data | Strongest reasoning and safety behavior out of the box |
| **Control and lock-in** | Full control, no vendor lock-in | Full control after export | Dependent on the vendor's pricing and model changes |

**If the product counts as a medical device under CDSCO:** add 6–15 months and about 60–140 person-months to every approach.

---

## How the numbers were worked out

**Workload per question:** about 2,000 input tokens (instructions + retrieved medical content + question) and 400 output tokens. No caching discount assumed, so these are conservative.

**API prices (Approach 3), per million input / output tokens:**

| Model | Price | Approx. cost per question |
|---|---|---|
| Claude Sonnet 5 / 5.5 | $2 / $10 | $0.008 |
| Claude Haiku 4.5 | $1 / $5 | $0.004 |
| Claude Haiku 5.5 | $0.10 / $0.50 | $0.0004 |
| Gemini 3.1 Pro | $2 / $12 | $0.009 |
| Gemini 3.8 Flash (2027 price) | $1.50 / $7.50 | $0.006 |

Notes:
- Gemini Flash rates are introductory and double on January 1, 2027. Budget at the 2027 price.
- Gemini output prices include thinking tokens, so real bills can run higher.
- Bedrock sets its own regional prices; check the India rates.
- As of September 2026, the Bedrock India in-country profile covers Claude Opus 5, Sonnet 5, and Haiku 4.5.

**Self-hosting (Approaches 1 and 2):**
- Two L4 instances (g6.xlarge) for redundancy: ~$1,175/month.
- Two L40S instances (g6e.xlarge) for a larger model: ~$2,715/month.
- At 1M questions/month, you may need 3–4 instances.

**Tinker training:** a typical supervised fine-tune of Qwen3-8B over 50M training tokens costs around $20 per run.

**Excluded (same for all approaches):** app, database, vector store, and logging, roughly $300–800/month.

---

## What the comparison means

1. **The API route is fastest and cheapest to build.** Development effort drops by roughly half because there's no training or GPU infrastructure. Data residency is solved for Claude through Amazon Bedrock's India profile (Mumbai and Hyderabad), and Bedrock doesn't store inputs or outputs by default. For Gemini, confirm Vertex AI's India region options separately.

2. **Self-hosting only saves money at high volume.** The GPU bill is fixed, so it beats the API only above roughly **150,000–350,000 questions/month**. That ignores the 1–2 engineers needed to keep the GPUs running. Counting them, the API usually stays cheaper until millions of questions a month.

3. **Tinker saves weeks, not months.** It removes training-infrastructure setup (about 2–4 weeks). Hosting, safety testing, and compliance stay the same. The trade-offs: you lose MedGemma's medical head start, and training data leaves India.

4. **The non-model work is the same in every approach.** About 60% of the effort goes to clinician-reviewed evaluation sets, safety guardrails, DPDP compliance, and monitoring. That's why no approach drops below roughly 3 months.

5. **Model updates are a hidden cost of the API route.** Providers retire model versions on their own schedule. Each switch means re-running your clinical evaluation, and if you're licensed as a medical device, each switch is also a regulated change.

---

## Recommendation

**Launch with Approach 3:** Claude Sonnet through Bedrock India, with RAG over trusted clinical sources. This reaches production fastest and spends least while you test whether the product works. Real usage also gives you the clinician-reviewed Q&A data you'd need for fine-tuning later.

**Move to Approach 1** (self-hosted fine-tuned MedGemma) only when:
- volume passes roughly 500,000+ questions/month,
- data must never leave your own AWS account, or
- API costs become a meaningful share of revenue.

A hybrid is also possible: the self-hosted model handles routine questions, and the API handles complex or red-flag cases.

---

## Sources to re-check before budgeting

- Claude API pricing: https://platform.claude.com/docs/en/about-claude/pricing
- Gemini API pricing: https://ai.google.dev/gemini-api/docs/pricing
- Tinker pricing: https://thinkingmachines.ai/tinker/pricing/
- Claude on Bedrock in India: https://aws.amazon.com/blogs/machine-learning/amazon-bedrock-expands-claude-model-availability-to-india-cross-region-inference/
- AWS EC2 on-demand pricing: https://aws.amazon.com/ec2/pricing/on-demand/