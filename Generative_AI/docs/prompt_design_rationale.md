# Prompt Design Rationale
## Obesity Level Advice System

---

## 1. Thought Process Behind Each Template

### General Design Philosophy

Before writing any prompt, we grounded our design in a key question: **who is reading this output, and what do they need to do with it?** Four distinct user needs emerged from this exercise, which directly drove the four template designs:

1. A **general user** who just received a prediction and needs a quick, jargon-free summary → **T1**
2. A **healthcare student or caregiver** who needs clinical depth and structured information to act on → **T2**
3. A user whose **lifestyle cluster** has already been identified, and who deserves personalized rather than generic advice → **T3**
4. A user who may feel **anxious, stigmatized, or overwhelmed** by their result, and who needs encouragement before information → **T4**

### T1 — "Start as simple as possible"

The thinking behind T1 was to establish a baseline with the least possible prompt intervention. We wanted to know what GPT would produce without role-setting or structural constraints. This is not laziness, it is good experimental design. If a minimal prompt already gives useful output, that tells us that adding complexity in T2–T4 must genuinely improve something.

We also recognized that T1 might be the most practical for a lightweight chatbot UI where response time and token cost matter.

### T2 — "Structure unlocks evaluation"

The most important insight behind T2 was that **unstructured outputs are hard to compare fairly**. If T1 gives us two paragraphs and T2 gives us four numbered sections, it is much easier to check whether T2 covered more ground. The four sections (Prediction Explanation, Key Risk Factors, Recommended Actions, Warning Signs) were chosen because they map to what a clinical handoff summary typically includes, making the structure medically meaningful, not arbitrary.

We also added a system message for the first time in T2. This was a deliberate escalation from T1. We wanted to test whether a role assignment ("You are a clinical decision-support assistant") changed the factual accuracy and tone of the response.

### T3 — "Personalization requires context injection"

T3 represents the most technically ambitious design. The core challenge was: GPT has no memory of what clustering we ran in Phase 2A. To bridge that gap, we included the cluster profile description directly in the prompt. This technique, often called **context injection** or **retrieval-augmented prompting** — is a standard pattern for grounding LLM outputs in data that was computed outside the model.

The three-section structure (health status → group insights → action plan) was designed to mirror a clinical conversation flow: tell the patient where they are, show them they are not alone, then give them a path forward.

### T4 — "Tone is not decoration, it is a design constraint"

T4 was the most ethically motivated design. We discussed early on that obesity-related health communication has a documented stigma problem, clinical language that emphasizes "risk" and "danger" can cause patients to disengage rather than act. The explicit ban on certain words ("obese", "dangerous", "risk") was not merely stylistic. It was a design decision intended to produce outputs that comply with health communication best practices.

The "Small Steps This Week" section was directly inspired by implementation intention theory: people are more likely to act on plans that specify a concrete action in a concrete timeframe. We deliberately asked for weekly steps rather than long-term goals.

---

## 2. How Domain Knowledge Influenced Design

Our dataset is drawn from an obesity classification study covering lifestyle and biometric features: BMI, physical activity frequency, water intake, caloric food consumption, transportation habits, family history, and technology use time. This domain context shaped our design in several important ways:

**Feature selection for prompt inputs.** Rather than passing raw encoded numbers (e.g., `FAVC_encoded = 1`), we mapped features back to their human-readable labels (e.g., "Frequent high-caloric food: yes"). This was a deliberate design choice to make prompts readable and to prevent GPT from misinterpreting numeric codes.

**Cluster profiles grounded in real data.** In T3, the cluster descriptions we injected were derived from actual cluster analysis results in Phase 2A — not invented. For example, Cluster 3 was characterized by the highest average BMI, lowest physical activity, and highest caloric intake. By grounding the prompt in real profiles, T3 outputs are responsive to actual population patterns rather than generic archetypes.

**Warning sign selection in T2.** The four warning signs mentioned in T2 outputs (e.g., chest pain, swelling in legs, sudden weight gain) align with clinically recognized obesity-related complications. The system message in T2 ("evidence-based explanations") encouraged GPT to draw on this knowledge.

**Avoiding BMI-only framing.** We were aware that BMI is a limited and sometimes misleading metric. Our feature strings include additional context (water intake, activity level, eating habits) to ensure GPT did not anchor advice purely on weight/height ratios.

---

## 3. Lessons Learned During Prompt Testing

### Lesson 1: System messages change tone more than content

The most consistent observation across all test cases was that system messages had a stronger effect on **tone and register** than on factual content. T1 (no system message) and T2 (clinical system message) often covered similar clinical ground, but T2 felt authoritative and structured while T1 felt conversational. This reinforced our decision to use carefully worded system messages for T3 and T4.

### Lesson 2: Word limits are highly effective structure constraints

Setting explicit word limits (T1: 120 words, T4: 200 words) reliably produced shorter, more focused outputs. Without a limit, GPT tends to expand and add caveats. For user-facing applications where screen real estate is limited, word constraints are a practical necessity.

### Lesson 3: Negative constraints ("avoid words like X") work well

T4's instruction to avoid stigmatizing vocabulary produced noticeably different language from T1–T3. GPT respected the constraint across all test cases. This was an unexpected positive result, we had anticipated that GPT might occasionally slip back into clinical framing on a severe case, but it maintained the coaching tone even for the Obesity Type III test case.

### Lesson 4: Cluster injection is effective but sensitive to profile quality

T3 performed best when the cluster profile descriptions were detailed and precise. Vague profiles (e.g., "Cluster 1: healthy users") produced generic advice that was barely more personalized than T1. This highlights that the quality of the unsupervised learning output directly limits the quality of the generative AI output, the two phases are tightly coupled.

### Lesson 5: Structured sections improve evaluability but can feel mechanical

T2's four-section structure made evaluation significantly easier (we could check each section independently for completeness and accuracy). However, for a live user-facing product, the numbered clinical sections could feel cold or impersonal. This confirmed that T2 is best suited as a caregiver reference tool, not a direct patient-facing interface.

### Lesson 6: Test case diversity matters

Using three test cases that span the severity spectrum (Obesity Type III, Normal Weight, Overweight Level I) was essential for stress-testing the templates. Normal Weight cases revealed an important issue: T2 tended to still flag "risk factors" even when the patient was healthy, because the system message primed clinical caution. This is a known issue in LLM risk communication and should be addressed with conditional framing in a production system.

---

## 4. References to Prompt Engineering Best Practices

The following prompt engineering principles informed our template design. We list the principle, its source, and how we applied it.

### 4.1 Role Prompting / Persona Assignment
**Principle:** Assigning a specific role or persona in the system message improves output consistency and relevance.
**Reference:** OpenAI Prompt Engineering Guide — https://platform.openai.com/docs/guides/prompt-engineering
**Applied in:** T2 (clinical decision-support assistant), T3 (personalized health advisor), T4 (wellness coach)

### 4.2 Context Injection / Retrieval Augmentation
**Principle:** LLMs lack access to domain-specific data computed outside the model. Injecting that context into the prompt grounds outputs in real data.
**Reference:** Lewis, P., Perez, E., Piktus, A., et al. (2020). "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks." *Advances in Neural Information Processing Systems (NeurIPS)*, 33.
**Applied in:** T3 (cluster profile descriptions injected into prompt from Phase 2A results)

### 4.3 Few-Shot and Zero-Shot Design
**Principle:** All four of our templates use zero-shot prompting (no examples in the prompt) because we have rich feature inputs that already constrain the output space sufficiently. Few-shot prompting would improve consistency further in a production system.
**Reference:** Brown, T., Mann, B., Ryder, N., et al. (2020). "Language Models are Few-Shot Learners." *Advances in Neural Information Processing Systems (NeurIPS)*, 33.
**Note:** A natural next step is to add 1–2 high-quality examples to T3 and T4 to further reduce variance.

### 4.4 Health Communication Best Practices
**Principle:** Obesity communication that uses stigmatizing or fear-based language reduces patient engagement and self-efficacy.
**References:**
- Rubino, F., Puhl, R. M., Cummings, D. E., et al. (2020). "Joint International Consensus Statement for Ending Stigma of Obesity." *Nature Medicine*, 26, 485–497.
- Gollwitzer, P. M. (1999). "Implementation Intentions: Strong Effects of Simple Plans." *American Psychologist*, 54(7), 493–503. — applied in T4's "small steps this week" design

**Applied in:** T4 system message and negative constraints; T3 empathetic framing

---
