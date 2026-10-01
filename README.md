# AI Defense Matrix — Maturity Assessment Guide

A working guide for scoring AI security controls against the [AI Defense Matrix](https://aidefensematrix.com), used here to drive a maturity assessment and roadmap.

## What is the AI Defense Matrix?

The AI Defense Matrix (AIDM) was co-authored by Lenny Zeltser and Sounil Yu as the "security for AI" companion to Yu's original **Cyber Defense Matrix**. Where the Cyber Defense Matrix organizes traditional security controls across five asset classes (Devices, Applications, Networks, Data, Users), the AI Defense Matrix extends that same organizing logic to AI-specific assets that traditional controls don't cover.

**Structure:**
- **Rows (8 asset classes):** things in your environment that need AI-specific defense
- **Columns (6 functions):** the NIST CSF 2.0 functions — Govern, Identify, Protect, Detect, Respond, Recover
- **Cells:** example control categories or technologies that defend that asset class within that function

## The matrix

| Asset Class | Govern | Identify | Protect | Detect | Respond | Recover |
|---|---|---|---|---|---|---|
| **AI-Workload Platforms** | AI-platform standards | AI security posture management | AI-workload hardening; model-loading supply-chain verification | AI-workload runtime detection | Generic container IR | Generic platform restore |
| **AI Coding and Orchestration Tools** | AI tool governance | Coding agent, orchestration harness, and agent-framework inventory; plugin, skill, and MCP server discovery | Sandboxing; harness hardening; plugin, skill, and MCP server allowlisting | Prompt-injection testing; agent anomaly detection | Agent runtime IR; plugin, skill, and MCP server disable | Harness config reset; plugin, skill, and MCP server restore; prompt rollback |
| **AI-Generated Code** | AI coding standards; code-review policy; license and provenance policy | AI-code provenance; origin tracking | AI-aware SAST | Hallucinated dependency; insecure-pattern detection | PR block; revert of AI-generated commits | Code rewrite; replacement of flagged artifacts |
| **AI Gateways and Routers** | AI egress policy; approved-service registry | AI traffic discovery | AI gateways for egress; MCP gateways for tool gating | Anomalous AI traffic; RAG-leakage egress detection | AI traffic blocking; shadow AI takedown | Generic network failover |
| **AI Model** | Model selection; provider evaluation | Model inventory; AIBOM | Model firewalls; weight protection | Model drift; integrity monitoring | Model rollback; provider coordination for consumed models | Model version restore; provider re-selection |
| **Training Data** | Dataset provenance; licensing policy | Dataset inventory; lineage | Data access control | Poisoning; backdoor detection | Dataset quarantine; retraining trigger | Dataset restore from golden copies; model retraining |
| **Runtime AI Data** | Prompt and RAG policy; memory-retention governance; interaction-history policy | RAG source; LLM-oversharing inventory | Prompt-injection defense, RAG sanitization, memory-poisoning defense, AI-content DLP | Prompt anomaly, jailbreak attempts, RAG leakage, memory tampering | Session termination; RAG source isolation | Vector DB restore; re-indexing |
| **AI Agent Identities** | AI agent identity policy, authorization standards, OAuth for agents | AI agent; non-human principal inventory | Agent OAuth; capability scoping, short-lived credentials | Agent behavioral monitoring; runtime authorization drift | Credential revocation, agent quarantine, session termination | Agent identity re-provisioning |

## How to use the matrix — walkthrough

The matrix is read one cell at a time: pick a row (asset class), pick a column (function), and ask *"what do we have for this intersection?"* Here are three worked examples showing the full process.

### Example 1: AI Agent Identities × Respond

**Cell content says:** Credential revocation, agent quarantine, session termination

**How to apply it:**
1. Ask: "If an Entra Agent Identity or Copilot agent starts behaving maliciously right now, what happens?"
2. Check for an actual artifact — a containment playbook, a documented revocation procedure, a SOAR workflow.
3. If a playbook exists that disables the agent's credentials and quarantines it, this cell is covered — score it high (e.g., 3/Mature).
4. If the answer is "someone would probably go disable it manually," this cell is weak — score it low (e.g., 1/Ad hoc) and flag it as a gap.

### Example 2: Runtime AI Data × Detect

**Cell content says:** Prompt anomaly, jailbreak attempts, RAG leakage, memory tampering

**How to apply it:**
1. Ask: "Would we know if someone successfully jailbroke a chatbot or exfiltrated data through a RAG pipeline?"
2. Check for evidence — an AI-content DLP tool, prompt-anomaly alerting, a SIEM rule tuned for jailbreak signatures.
3. No tooling exists for this today in most orgs — this is usually one of the weakest cells across the industry, so don't be surprised if it scores a 0 or 1.

### Example 3: AI-Generated Code × Protect

**Cell content says:** AI-aware SAST

**How to apply it:**
1. Ask: "Does our code-scanning pipeline know the difference between AI-generated and human-written code, and catch AI-specific issues like hallucinated dependencies?"
2. Check whether your SAST tool has AI-code-aware rules enabled, or if it's just running generic static analysis against AI output.
3. Many orgs score this a 1–2: they run SAST, but it isn't tuned for AI-specific failure modes (e.g., a package name that doesn't exist, confidently invented by the model).

### The repeatable process

1. Pick a row.
2. Walk across all 6 columns left to right, asking "what do we have here?" for each cell.
3. Score 0–3 using the rubric below, backed by a named artifact (policy ID, tool name, log entry).
4. Repeat for all 8 rows (48 cells total).
5. Sort your scored grid by lowest score — that's your prioritized gap list.

## Maturity scoring rubric

Score every cell **0–3**:

| Score | Label | Definition | The tell |
|---|---|---|---|
| **0** | None | No control exists; nobody owns it | Asking "who handles this?" gets silence |
| **1** | Ad hoc | Control exists but is manual, undocumented, or person-dependent | It only works because one specific person remembers to do it |
| **2** | Partial | Documented and/or partially automated, but has gaps in coverage or enforcement | Exists in policy or in one environment/tool, not consistently everywhere |
| **3** | Mature | Automated, enforced, monitored, and consistently applied | Runs without a human trigger, with logs/evidence proving it works |

**The single test:** *If the person who built this left tomorrow, would it keep working?*
No control → 0. No, it'd break → 1. Mostly, with cracks → 2. Yes, entirely → 3.

## What to investigate per function (before scoring)

| Function | The question | Where to look |
|---|---|---|
| **Govern** | Is there a written, approved policy? Does it have enforcement teeth or is it aspirational? | Internal policy/standards docs, AI governance approval registry |
| **Identify** | Do we have a current, accurate inventory of this asset class? | CSPM tool inventory, cloud resource graph queries, identity provider's app/agent registry |
| **Protect** | Are preventive controls actually blocking bad configs/access, not just flagging them? | Policy assignments with deny (not audit-only) effects, conditional access, RBAC scoping |
| **Detect** | Would we know within a reasonable window if this asset class was compromised or drifted? | CSPM/SIEM alerting rules tied specifically to this asset class |
| **Respond** | Is there a documented, *tested* playbook? | Incident response runbooks, SOAR workflows — confirm it's been run for real, not just written |
| **Recover** | Can we restore to a known-good state, and has that restore been tested? | Backup/restore procedures, golden configs, infrastructure-as-code re-provisioning |

### Investigation checklist

1. Pull the inventory first (Identify) — nothing else can be honestly scored without knowing what exists.
2. Search for a written policy or standard (Govern).
3. Check for deny/enforce policy effects, not just audit-only (Protect).
4. Check for an active alert rule tied to this asset class (Detect) — "we'd probably notice" is not evidence.
5. Find the runbook and confirm whether it has actually been executed (Respond).
6. Ask whether a restore has ever been tested (Recover) — untested caps the score at 1–2 regardless of documentation.

**Avoid the intent trap:** score based on evidence ("here's the policy ID / log entry / alert rule name"), not plans ("we're going to..."). If you can't point to an artifact justifying a 2 or 3, drop the score by one until you can.

## Worked scoring examples (full rows)

These show the full "mark it" process — one row where coverage is strong, one where it's weak — so you can see the full range of how scoring plays out in practice.

### Row example 1: AI Agent Identities (mixed maturity)

| Function | Score | Evidence found | Why this score |
|---|---|---|---|
| Govern | 1 | No written AI agent identity policy or OAuth-for-agents standard exists | Nothing documented — scores at the "ad hoc" ceiling by default |
| Identify | 2 | Informal agent inventory tracked through governance work, no formal AIBOM-style registry | Exists and is used, but not a systematic, queryable inventory — capped at Partial |
| Protect | 1 | No capability scoping or short-lived credential enforcement in place | A preventive control is described in the matrix cell but nothing enforces it today |
| Detect | 1 | No dedicated behavioral monitoring for agent runtime drift | No alert rule exists specifically for this — can't claim higher than Ad hoc |
| Respond | 3 | A tested containment playbook exists to disable rogue agent identities | Documented, automatable, and has actually been exercised — this earns Mature |
| Recover | 1 | No formal re-provisioning process defined | No documented or tested restore path |

**Row takeaway:** Strong Respond, weak everywhere else — a classic "we can react but can't prevent or detect" pattern. The fix priority here is Govern and Protect, since those are what would have stopped the incident Respond is cleaning up after.

### Row example 2: Training Data (low maturity, for contrast)

| Function | Score | Evidence found | Why this score |
|---|---|---|---|
| Govern | 0 | No dataset provenance or licensing policy exists | Nobody owns this — a true gap |
| Identify | 1 | Datasets are known informally by the team that built them, no central lineage tracking | Exists only in people's heads, not a system |
| Protect | 2 | Data access control exists via standard IAM/RBAC on storage, but not AI-specific | Partial — generic access control, not purpose-built for training data sensitivity |
| Detect | 0 | No poisoning or backdoor detection tooling in place | No control exists at all |
| Respond | 0 | No dataset quarantine or retraining-trigger process defined | No control exists at all |
| Recover | 1 | Golden-copy backups exist for some datasets but haven't been tested for restore | Exists but unverified — caps it at Ad hoc |

**Row takeaway:** Near-zero maturity across the board — typical for Training Data in orgs that consume models rather than train their own, since this row matters most if you fine-tune or train in-house.

### How to read the scored grid once all 8 rows are done

- **Column-wide low scores** (e.g., every row scores 0–1 on Detect) point to a missing *capability* — you likely need a tool or platform investment.
- **Row-wide low scores** (e.g., Training Data scores low across all 6 functions) point to a missing *owner* — nobody has been assigned that asset class yet.
- **A single weak cell in an otherwise strong row** (e.g., AI Agent Identities' Govern/Protect/Detect gaps next to a strong Respond) is your most actionable finding — it's a specific, scoped project rather than a program-wide initiative.

## Framework and standards basis

- **NIST Cybersecurity Framework (CSF) 2.0** — provides the six-function column structure (Govern, Identify, Protect, Detect, Respond, Recover), the same language most security programs and auditors already use.
- **Cyber Defense Matrix** (Sounil Yu) — the parent framework; the AI Defense Matrix follows its row × function grid pattern applied to AI-specific assets instead of traditional IT assets.
- **Cross-mapped to adjacent AI security frameworks**, including NIST IR 8596 (AI component identification), the OWASP LLM Top 10 (application-layer risks), ISO/IEC 42001 (AI management system controls), and the SANS Critical AI Security Guidelines — each of these covers one slice of AI security; the AIDM is designed to give a single consolidated view across all of them.
- The framework is free to use and licensed under **CC BY-SA 4.0**.

## Why organizations use this matrix

- **No existing single framework covers the whole picture.** NIST IR 8596, OWASP LLM Top 10, and ISO 42001 each address a slice of AI risk. The AIDM gives security leaders one grid to see across all of them at once.
- **Gap-finding.** Filling out the grid quickly reveals which asset-class/function combinations have zero coverage — these are usually the highest-priority items for a roadmap.
- **Ownership clarity.** AI security work tends to be split across teams (platform, data, identity, appsec). The matrix forces an explicit answer to "who owns this cell?" instead of leaving it assumed.
- **Common language with leadership and auditors.** Because it's built on NIST CSF 2.0, findings translate directly into terms risk committees and auditors already understand.
- **Vendor and tool evaluation.** Security leaders use it to see where a proposed tool actually fits (and where it doesn't), and to spot when many vendors are competing for the same crowded cell while others go unaddressed.

## Benefits of using this matrix

- Turns a vague goal ("secure our AI systems") into 48 specific, answerable questions (8 asset classes × 6 functions).
- Surfaces blind spots — most organizations are strong in a couple of cells (often Identify/Protect for AI-Workload Platforms) and have near-zero coverage in newer areas like AI Agent Identities or Runtime AI Data.
- Creates a defensible, evidence-based basis for budget and headcount requests, since each score should be backed by a named artifact (a policy, a log, an alert rule).
- Produces a repeatable baseline — score it today, re-score in two quarters, show measurable progress.

## License

The AI Defense Matrix is © 2026 Lenny Zeltser and Sounil Yu, licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/). Source: [aidefensematrix.com](https://aidefensematrix.com).
