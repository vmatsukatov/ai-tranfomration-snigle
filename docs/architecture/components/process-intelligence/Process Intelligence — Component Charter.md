# Process Intelligence — Component Charter

**Status:** Proposed  
**Scope:** Business/product and logical architecture; no model training or production deployment  
**Principle:** Begin with a narrow, read-only, evidence-grounded assistant. Increase autonomy only when evaluation evidence, governance, and explicit customer approval justify it.

## 1. Purpose

Process Intelligence is the explainable assistance and insight component of a GDPR-compliant, multi-tenant AI process platform. It turns authorized company context, governed knowledge, and process state into useful answers and proposed actions without becoming the system of record or bypassing process controls.

It consumes three governed inputs:

1. **Company Structure** — tenants, legal entities, business units, teams, roles, responsibilities, reporting relationships, and authorization attributes.
2. **Knowledge Data** — approved policies, procedures, work instructions, definitions, templates, evidence, and metadata such as owner, validity, jurisdiction, classification, and retention.
3. **Process Graph State** — process definitions, instances, tasks, dependencies, events, decisions, controls, owners, SLAs, status, and history.

Its initial job is to help a person understand and navigate work. Its long-term job may include executing narrowly defined, reversible steps, but only through existing platform commands, policy checks, and explicit approval mechanisms.

### 1.1 Desired outcomes

- People get fast, grounded answers about how work should proceed and what is happening now.
- Process owners discover gaps, ambiguity, bottlenecks, control failures, and improvement opportunities.
- Workers receive relevant next-step guidance with evidence and clear uncertainty.
- Auditors and administrators can reconstruct what context, policy, model, rule, and approval produced an output or action.
- Tenants can adopt capabilities incrementally without surrendering control of data, models, or automation.

### 1.2 Non-goals for the first stages

- A general-purpose enterprise chatbot.
- A new source of truth for organization, knowledge, or process state.
- Unreviewed generation or publication of policy.
- Autonomous changes to process definitions, permissions, retention, legal decisions, or high-impact business outcomes.
- Training foundation models on tenant data.
- Selecting a permanent model provider before workload-specific evidence exists.
- Replacing deterministic workflow, authorization, validation, or compliance controls with probabilistic reasoning.

## 2. Users and jobs to be done

| User | Jobs to be done | Representative questions or outcomes |
|---|---|---|
| Process participant | Understand assigned work and complete it correctly | “What should I do next?”, “Why is this required?”, “Show the applicable procedure.” |
| Team lead / supervisor | Unblock work and manage exceptions | “Which cases are at risk?”, “What is blocking this task?”, “Who can approve it?” |
| Process owner | Improve process design and performance | Find missing ownership, recurring loops, bottlenecks, ambiguous handoffs, and control gaps. |
| Knowledge owner | Maintain trustworthy guidance | Detect stale, conflicting, unused, or uncovered guidance and propose updates for review. |
| Risk, privacy, legal, or compliance role | Verify adherence and evidence | Trace decisions, identify deviations, check lawful use, and export auditable evidence. |
| Executive / analyst | Understand aggregate performance | Obtain permission-filtered trends, causes, and improvement hypotheses without exposing personal data unnecessarily. |
| Platform / tenant administrator | Configure safe use | Choose enabled capabilities, data domains, providers, residency, budgets, policies, and approval thresholds. |
| Developer / integration owner | Extend tools safely | Register typed tools, test policies, inspect traces, and evaluate changes before release. |

The interface must preserve the distinction between fact, retrieved evidence, deterministic calculation, model inference, and recommendation.

## 3. Capability ladder and autonomy contract

Capability is enabled per tenant, use case, process, role, and action—not as one global “AI on” switch. A higher level does not remove controls from a lower level.

| Level | Capability | Allowed behavior | Required guardrails | Example |
|---|---|---|---|---|
| L0 | Search and retrieval | Return permission-filtered records and documents | Exact provenance; no synthesized claims | Find the current onboarding procedure. |
| L1 | Grounded read-only Q&A | Summarize and answer from authorized context | Citations; uncertainty; abstention; no state mutation | Explain why a task is blocked. |
| L2 | Guided work | Explain requirements, compare state with rules, produce checklists or drafts | Separate rules from inference; user verifies output | Guide a worker through an exception path. |
| L3 | Insight and detection | Identify gaps, anomalies, bottlenecks, and likely causes | Reproducible queries where possible; thresholds; human validation | Flag cases likely to breach SLA. |
| L4 | Recommendations | Rank next actions and explain expected effect and trade-offs | Policy validation; confidence; alternatives; approval target | Recommend reassignment to an eligible role. |
| L5 | Controlled automation | Execute an explicitly allowed action through a typed command | Fresh authorization, preview, approval, idempotency, audit, rollback/compensation | After approval, reassign a task and notify the new owner. |
| L6 | Bounded delegated automation | Execute a pre-approved plan inside strict scope and stop conditions | Tenant opt-in, action budget, continuous policy enforcement, monitoring, kill switch | Resolve a low-risk queue using an approved playbook. |

**Initial ceiling:** L1 for production use. A narrow L2 pilot may follow after L1 exit criteria are met. L5–L6 are architectural horizons, not launch commitments.

### 3.1 Automation invariant

The reasoning component never mutates authoritative state directly. It submits a typed command to the platform command layer. The command layer independently performs authentication, authorization, purpose and policy checks, schema validation, concurrency control, approval verification, execution, and audit logging. A generated statement such as “approved” is never itself an approval.

## 4. Responsibility boundary

### 4.1 Process Intelligence owns

- Use-case orchestration and capability-level enforcement.
- Authorized context requests and context assembly.
- Retrieval, reranking, prompt construction, and model routing.
- Tool selection proposals and typed tool invocation within policy.
- Evidence linking, confidence signals, explanations, and safe fallback.
- Evaluation, quality telemetry, cost/latency budgets, and feedback capture.
- AI-specific audit records: context manifest, prompt/template version, provider/model version, tool trace, policy decisions, output, approval, and outcome.

### 4.2 It does not own

- Tenant identity, user authentication, or authoritative access control.
- Company Structure, Knowledge Data, or Process Graph persistence.
- Core workflow execution and business invariants.
- Approval records or legally significant signatures.
- Source-document governance, retention schedules, or legal holds.
- The authoritative process analytics store.
- Enterprise-wide observability and incident response, though it emits required signals.

### 4.3 Dependencies and contracts

| Dependency | Read contract | Write/action contract |
|---|---|---|
| Identity and policy | Principal, tenant, roles/attributes, purpose, consent or legal-basis signals where relevant | Policy decision and authorization token; Process Intelligence cannot grant access. |
| Company Structure | Versioned, tenant-scoped graph/query API | Changes remain in the owning service and require its workflow. |
| Knowledge Data | Permission-aware hybrid retrieval plus document/version metadata | Draft suggestions only until approved through knowledge governance. |
| Process Graph | Versioned definitions and state/query API, including event time | Commands go through workflow APIs with preconditions and idempotency keys. |
| Approval service | Approval policy, approver set, state, expiry | Approval request references a frozen action preview and context digest. |
| Audit service | Append-only event interface | Tamper-evident AI and action records with tenant-controlled retention. |

## 5. Decision allocation: rules, LLMs, and conventional ML

Use the least complex mechanism that reliably solves the problem.

| Mechanism | Best suited to | Must not be the sole authority for |
|---|---|---|
| Deterministic workflow, queries, and rules | Permissions, required fields, eligibility, deadlines, process conformance, calculations, approval routing, command validation | Ambiguous language interpretation or open-ended explanation |
| LLM reasoning | Natural-language Q&A, synthesis across sources, explanation, intent classification, draft generation, tool-plan proposals | Access control, legal conclusions, policy enforcement, numeric truth, irreversible or high-impact decisions |
| Conventional ML / statistics | Forecasting duration or breach risk, anomaly detection, clustering, ranking, capacity patterns | Decisions without validated labels, drift monitoring, calibrated thresholds, and human/policy controls |

Preferred execution pattern:

1. Resolve identity, tenant, purpose, and allowed capability.
2. Obtain facts and deterministic findings from owning services.
3. Use an LLM to explain those findings or propose a bounded plan.
4. Validate structured output against schemas and deterministic policy.
5. Present evidence, uncertainty, and required approval.
6. For enabled automation, execute only through the command layer and record the outcome.

Where a database query or graph algorithm can answer exactly, use it and let the model narrate—not calculate—the result.

## 6. Logical architecture

```text
User / API client
       |
       v
Interaction API -- identity, tenant, purpose, capability level
       |
       v
PI Orchestrator
  |-- Policy Enforcement Point <---- central authorization/policy service
  |-- Context Assembler
  |     |-- Company Structure adapter
  |     |-- Knowledge retrieval adapter
  |     `-- Process Graph query adapter
  |-- Deterministic analysis/rules engine
  |-- Retrieval + reranking
  |-- Model gateway / router -------- hosted | private | local/open model
  |-- Tool gateway (typed, allowlisted MCP/native tools)
  |-- Evidence + explanation composer
  |-- Output/action validator
  `-- Evaluation, telemetry, and feedback
       |
       +--> Read-only response with citations
       `--> Action proposal --> Approval service --> Platform command layer
                                                    |
                                                    v
                                           Authoritative domain service

Every boundary carries tenant_id, principal, purpose, authorization context,
correlation_id, data classification, and version/freshness metadata.
```

Deploy the orchestration layer stateless where practical. Store conversation state, traces, embeddings, caches, evaluation artifacts, and feedback only in explicitly governed, tenant-scoped stores.

## 7. Context assembly

Context assembly is an authorization-sensitive data product, not a prompt concatenation step.

### 7.1 Request envelope

Every request includes:

- tenant and principal identity;
- role/attribute claims and delegated authority;
- declared use case and processing purpose;
- capability level and action scope;
- process definition/instance/task identifiers where applicable;
- locale, timezone, jurisdiction, and desired response form;
- correlation ID, timestamp, and client/channel;
- data-classification ceiling and applicable residency/provider restrictions.

### 7.2 Assembly pipeline

1. **Authorize before retrieval.** Apply row-, field-, document-, edge-, and purpose-level filters at the source or trusted retrieval boundary.
2. **Resolve the work anchor.** Identify the exact process version, instance, task, organization scope, and relevant time point.
3. **Collect structured facts.** Prefer typed fields and graph queries over prose extraction.
4. **Retrieve governed knowledge.** Filter by tenant, ACL, jurisdiction, validity interval, status, language, classification, and process link before semantic ranking.
5. **Add deterministic findings.** Rules, deadlines, conformance deviations, and metrics include rule/query version and inputs.
6. **Minimize.** Include only fields needed for the declared purpose; redact or pseudonymize personal and special-category data where possible.
7. **Rank and budget.** Deduplicate, rerank, and fit the context window without discarding mandatory policy evidence.
8. **Create a context manifest.** Record source IDs, versions, timestamps, authorization decision IDs, transformations, hashes, and freshness.
9. **Bind instructions by trust level.** System policy outranks tenant configuration; source content is evidence and never executable instruction.

### 7.3 Freshness and consistency

- Responses display an “as of” time and identify stale or superseded sources.
- Action proposals bind to source versions and state preconditions; execution fails closed when material state changed.
- Conversation memory never overrides current authoritative state.
- Cached retrieval and model outputs are partitioned by tenant, authorization scope, purpose, model/template version, and source version. Sensitive shared caching is prohibited.

## 8. Knowledge and integration strategies

### 8.1 RAG

RAG is the default for changing, tenant-specific facts and governed knowledge because it supports freshness, revocation, and citations.

- Use hybrid lexical/semantic retrieval with metadata and ACL filtering before reranking.
- Chunk along semantic/document boundaries and retain document, section, version, owner, validity, and classification metadata.
- Keep embeddings tenant-isolated logically at minimum; use stronger physical isolation for tenant tier, regulation, or threat model where justified.
- Treat embedding vectors as personal/confidential data when their source is such data.
- Re-index, tombstone, and invalidate caches when source access, version, retention, or deletion changes.
- Protect against prompt injection in retrieved content through trust labels, instruction isolation, content scanning, tool restrictions, and tests.

RAG is not proof of truth: the answer must remain faithful to sources, and conflicting or insufficient evidence must be surfaced.

### 8.2 MCP and tool calls

MCP can provide a standard integration boundary, but protocol compatibility does not confer trust.

- Register only approved servers/tools with a tenant-aware catalog.
- Expose narrow, typed, versioned tools rather than general database, filesystem, browser, or code execution.
- Classify tools as read, draft, reversible write, irreversible write, or external communication.
- Validate arguments and results; constrain resource identifiers to the active tenant and authorized scope.
- Use short-lived delegated credentials; do not place secrets in prompts or model-visible context.
- Require previews and approval for writes; attach idempotency keys, preconditions, timeouts, rate limits, and compensation behavior.
- Log tool selection, sanitized arguments, policy decisions, result metadata, and side effects.
- Treat tool output as untrusted data and prevent it from redefining system policy.

Native internal APIs remain acceptable where they provide stronger typing or controls; MCP is an adapter choice, not an architectural mandate.

### 8.3 Fine-tuning

Do not fine-tune initially. Prompting, deterministic tools, and RAG should establish the task and evaluation baseline first.

Consider fine-tuning only if repeated evaluation shows a stable, high-volume behavior gap—such as output structure, domain terminology, or routing—that cannot be solved reliably and economically with simpler methods. Fine-tuning is not the primary mechanism for current tenant facts or access control.

Any future tenant-data tuning requires a documented lawful basis and purpose, minimization, provenance, consent/contract analysis where applicable, deletion strategy, isolation decision, memorization/leakage testing, data-subject-rights handling, and a clear prohibition on cross-tenant learning without explicit lawful agreement. Prefer de-identified or synthetic examples.

### 8.4 Model deployment options

| Option | Strengths | Costs/risks | Evidence needed |
|---|---|---|---|
| Hosted frontier model | Strong general reasoning, rapid iteration, managed scaling | Data-transfer terms, residency, provider dependence, variable cost/latency | Task quality, contractual controls, retention/training policy, regional availability, exit plan |
| Privately hosted open model | Greater infrastructure and data-path control; customizable | Operations, security patching, capacity, potentially lower quality | Quality at target workload, total cost, skills, throughput, security lifecycle |
| Local/edge model | Minimal external transfer; low latency for small tasks | Device variance, limited context/reasoning, update governance | Hardware coverage, acceptable quality, secure distribution, supportability |
| Hybrid/router | Match sensitivity and complexity to model; resilience and cost control | More evaluation, routing, observability, and consistency complexity | Routing policy performance, fallback quality, portability, cross-provider regression results |

The model gateway must normalize authentication, regional routing, timeouts, retries, streaming, structured outputs, safety settings, usage metering, and audit metadata. Provider-specific features stay behind adapters. No provider receives tenant data for its own training by default.

Selection is made per workload using a weighted scorecard: grounded quality, privacy and contractual posture, residency, latency, availability, structured/tool-use reliability, context capacity, safety, portability, operability, and total cost. Re-evaluate periodically and on material model changes.

## 9. Trustworthy responses and actions

### 9.1 Citations

- Every material factual claim derived from tenant context links to the exact accessible source, section/field, version, and “as of” time.
- Deterministic results cite the rule/query and version, not a fabricated document citation.
- Citations are verified after generation: the cited evidence must entail the claim and remain authorized for the requesting user.
- If the user cannot open the underlying source, do not reveal it through a citation snippet.
- Clearly label general model knowledge and avoid it when the use case requires tenant-grounded answers.

### 9.2 Confidence and uncertainty

Do not present raw model token probability as business confidence. Produce separate, interpretable signals:

- retrieval coverage and source authority/freshness;
- citation entailment/faithfulness;
- deterministic-rule result certainty;
- conventional-ML calibrated probability, when used;
- ambiguity, conflict, or missing-data flags;
- overall response status: **supported**, **partially supported**, **insufficient evidence**, or **blocked by policy**.

Calibration is measured on task-specific labeled data. The UI explains why confidence is limited and what evidence or action would improve it.

### 9.3 Explainability

Show a concise answer first, then:

- evidence and source versions;
- known facts versus inference;
- applicable rule or process path;
- assumptions and missing information;
- alternatives and expected trade-offs for recommendations;
- for actions, the exact proposed change, affected records, authority, risks, reversibility, and approver.

Expose a decision trace, not hidden chain-of-thought. The trace contains structured inputs, retrieved evidence, rules/tools used, versions, validation outcomes, and approvals.

### 9.4 Approvals

Approval requirements are deterministic policy based on action class, impact, role, data sensitivity, process, and tenant settings.

- The approver sees a stable preview of the exact action and evidence.
- Approval is scoped, attributable, time-limited, and bound to an action/context digest.
- Segregation of duties and multi-party approval are supported.
- Material state change, expired approval, or altered arguments invalidates approval.
- Bulk actions show item count, sample/diff, exception behavior, and rollback/compensation plan.
- Emergency stop and tenant-wide capability disablement are always available.

### 9.5 Safe fallback

When evidence is missing, conflicting, stale, unauthorized, or below threshold:

1. Do not guess or call mutating tools.
2. State the limitation in plain language.
3. Return verified partial facts when useful.
4. Ask for the minimum missing information or route to the responsible human/owner.
5. Offer the canonical procedure or search path.
6. Record the failure mode for evaluation without storing unnecessary prompt content.

## 10. Evaluation and release gates

### 10.1 Evaluation datasets

Build a versioned evaluation suite before broad pilot use:

- **Golden Q&A:** representative questions with acceptable answers, required/forbidden claims, and authoritative citations.
- **Process scenarios:** valid paths, exceptions, loops, parallel work, stale state, role changes, SLA cases, and ambiguous ownership.
- **Recommendation cases:** expert-ranked actions, unacceptable actions, trade-offs, and required approvals.
- **Tool/action cases:** schemas, policy outcomes, state preconditions, idempotency, rollback/compensation, and injected failures.
- **Privacy/security adversarial set:** cross-tenant requests, unauthorized fields, inference attacks, prompt injection, tool-output injection, secrets, special-category data, deletion, and membership changes.
- **Robustness set:** conflicting, stale, missing, multilingual, noisy, and out-of-domain inputs.
- **Production shadow set:** sampled and minimized real cases with governance approval, redaction/pseudonymization, restricted access, and retention limits.

Datasets are stratified by tenant archetype, role, language, process complexity, source type, risk tier, and capability. Avoid transferring one tenant's content into another tenant's evaluation. Synthetic data supports coverage but does not replace governed real-world validation.

### 10.2 Metrics

| Dimension | Example measures |
|---|---|
| Accuracy and grounding | Claim correctness; citation precision/recall; citation entailment; hallucination/unsupported-claim rate; structured-output validity |
| Relevance and usefulness | Task completion; expert/user relevance rating; top-k retrieval recall; answer completeness; unnecessary-content rate |
| Process adherence | Correct next-step/path rate; rule-conformance rate; forbidden-action rate; correct approval routing; successful/compensated action rate |
| Privacy and isolation | Unauthorized disclosure rate; cross-tenant leakage rate (target zero); data-minimization violations; deletion/ACL propagation time; prompt-injection success rate |
| Confidence and fallback | Calibration error/Brier score where applicable; abstention precision/recall; unsafe-answer rate under insufficient evidence |
| Latency and reliability | p50/p95/p99 end-to-end and component latency; timeout/error rate; availability; tool success rate |
| Cost and efficiency | Cost per request/completed job; tokens and retrieval/tool calls per job; cache benefit; human minutes saved; rework rate |
| Product impact | Adoption, repeat use, resolution time, SLA breaches, escalation quality, recommendation acceptance and realized outcome |

Metrics must be segmented; averages can hide high-risk failure modes. Privacy and forbidden-action tests are hard release gates, not metrics that can be traded against convenience.

### 10.3 Test layers and change control

- Unit/contract tests for policies, adapters, schemas, filters, and citations.
- Offline evaluation for every prompt, retriever, model, rule, and tool change.
- Adversarial and tenant-isolation testing in CI and before release.
- Human expert review for high-risk scenarios.
- Shadow and canary runs with no writes, then tightly scoped pilots.
- Regression comparison against the current production baseline, including cost and latency.
- Version pinning and rapid rollback for prompts, models, indexes, policies, and tools.

Release thresholds are use-case-specific and approved by product, domain, security, and privacy owners. Any material model/provider update is treated as a component change and re-evaluated.

## 11. Feedback and learning loops

Capture structured feedback at the point of use:

- helpful/not helpful with reason codes;
- incorrect claim or citation;
- missing or stale knowledge;
- wrong process interpretation;
- recommendation accepted, edited, rejected, or escalated;
- action outcome, rollback, exception, and downstream impact.

Feedback does not flow directly into prompts, retrieval ranking, rules, or training. It enters a governed triage process:

1. Separate product friction, retrieval failure, source-quality problem, rule defect, model failure, and policy block.
2. Route source issues to knowledge/process owners.
3. Redact and minimize examples; control evaluator access and retention.
4. Add reviewed cases to versioned evaluation sets.
5. Change one or more components through normal review and release gates.
6. Measure whether accepted recommendations produced the desired outcome, not merely clicks.

Tenant-specific feedback remains tenant-scoped. Cross-tenant aggregate learning uses only lawfully permitted, minimized data and cannot expose tenant content.

## 12. GDPR, privacy, and tenant isolation

### 12.1 Privacy principles

- Define controller/processor roles and subprocessors per deployment and contract.
- Record purpose and lawful basis for each use case; prohibit incompatible secondary use.
- Apply data protection by design/default, purpose limitation, minimization, accuracy, storage limitation, integrity, confidentiality, and accountability.
- Complete a DPIA before uses likely to create high risk, particularly employee monitoring, profiling, sensitive data, or consequential automated decisions.
- Maintain processing records, data-flow/subprocessor inventory, transfer safeguards, incident procedures, and data residency controls.
- Support access, rectification, erasure, restriction, objection, and portability across source records, indexes, embeddings, caches, conversations, feedback, logs, and training artifacts where applicable.
- Avoid solely automated decisions with legal or similarly significant effects unless a validated lawful exception and meaningful safeguards exist. Provide human intervention and contestability.
- Configure retention by artifact and purpose; legal holds override deletion only through governed policy.

This charter is a design baseline, not legal advice; tenant-specific use cases require privacy and legal review.

### 12.2 Isolation controls

- Tenant identity is derived from trusted authentication, never from model/user-supplied text.
- Every storage key, query, event, cache key, index namespace, tool credential, and audit record is tenant-scoped.
- Authorization filters execute before retrieval and are rechecked at citation rendering and action execution.
- Use per-tenant encryption keys where required; central secrets management and rotation are mandatory.
- Separate tenant data planes, indexes, or model deployments when risk tier, regulation, contract, or scale warrants it.
- Prohibit cross-tenant joins in online paths. De-identified aggregate analytics require a separate governed pipeline and minimum-group protections.
- Test confused-deputy, identifier substitution, cache bleed, index bleed, log leakage, support access, backup/restore, and deletion paths.
- Provider routing enforces tenant residency, model allowlist, retention, and data-use policy before any content leaves the platform boundary.

### 12.3 Logging and observability

Default logs contain metadata and hashes rather than full prompts or documents. Content capture is opt-in for a defined debugging/evaluation purpose, access-controlled, redacted where possible, time-limited, and auditable. Operational alerts detect anomalous access, cross-tenant policy failures, repeated injection attempts, runaway tool loops, cost spikes, and shifts in quality or abstention.

## 13. Key risks and treatments

| Risk | Treatment |
|---|---|
| Plausible but false guidance | Grounding, claim-level citations, deterministic checks, abstention, expert evaluation |
| Cross-tenant or unauthorized disclosure | Source-side authorization, pervasive tenant context, isolated stores/caches, adversarial tests, fail-closed policy |
| Prompt/tool injection | Trust hierarchy, content isolation, typed allowlisted tools, least privilege, output validation, red-team corpus |
| Automation exceeds authority | Capability ceiling, command-layer enforcement, approval binding, state preconditions, action budgets, kill switch |
| Stale/conflicting source material | Validity metadata, freshness display, conflict detection, knowledge-owner workflow |
| Over-reliance and automation bias | Fact/inference labels, alternatives, uncertainty, meaningful human review, outcome monitoring |
| Employee surveillance or unlawful profiling | Purpose limitation, DPIA, aggregation/minimization, role controls, transparency, works-council/legal review where applicable |
| Model/provider change degrades behavior | Gateway abstraction, pinned versions, regression suite, canary, rollback, portability tests |
| Cost/latency becomes unviable | Use-case budgets, caching with safe partitioning, smaller models for bounded tasks, asynchronous analysis, routing evidence |
| Feedback poisons learning or leaks data | Governed triage, provenance, access controls, redaction, evaluation before release |
| Explanations create false certainty | Structured evidence trace and calibrated status rather than generated rationale alone |
| Vendor lock-in | Portable contracts, owned evaluation suite, exportable prompts/configuration, at least one tested fallback for critical workloads |

## 14. Decisions, open questions, and decision gates

### 14.1 Decisions made by this charter

1. Process Intelligence is an assistive orchestration component, not a system of record.
2. Production starts at L1 with a narrow use case; autonomy is earned per action and tenant.
3. Deterministic systems retain authority for policy, authorization, workflow invariants, and execution.
4. RAG and tools precede fine-tuning for tenant facts and changing process state.
5. Every material response is evidence-linked; every action uses typed commands and independent validation.
6. Tenant isolation and purpose-aware authorization apply before retrieval, during reasoning/tool use, and at output/action time.
7. Model/provider choice remains open and workload-specific until benchmark, privacy, latency, reliability, and cost evidence supports it.

### 14.2 Open decisions to resolve through discovery

- Which single process, role, language, and tenant archetype make the best first assistant?
- What are the authoritative APIs and permission semantics of the three input domains?
- What knowledge quality and versioning gaps must be fixed before Q&A is safe?
- What jurisdictions, residency constraints, special-category data, and worker-context obligations apply?
- What latency, availability, and unit-cost targets matter for the first workflow?
- Which actions are reversible and low risk enough for a future L5 pilot?
- What isolation tiers and deployment options will customers require?
- Who owns quality, privacy, model risk, incident response, approvals, and go/no-go decisions?

### 14.3 Evidence-based gates

| Gate | Decision | Required evidence |
|---|---|---|
| G0: Problem fit | Proceed with one assistant | Observed user pain, authoritative sources, measurable outcome, privacy feasibility |
| G1: Technical baseline | Choose initial retrieval/model configuration | Blind evaluation across candidate approaches; contract/privacy review; latency and cost measurements |
| G2: Internal pilot | Allow real authorized users | Isolation/security tests pass; citations and fallback meet thresholds; audit and deletion work end to end |
| G3: Tenant production L1 | Release read-only Q&A | Tenant acceptance, operational SLOs, support/incident readiness, DPIA/legal actions complete |
| G4: L2–L4 expansion | Add guidance/insights/recommendations | Task-specific datasets, calibrated thresholds, outcome evidence, UI and human-factors review |
| G5: L5 controlled action | Permit one write action | Command and approval controls, state-precondition tests, idempotency, compensation, staged rollout, explicit tenant opt-in |
| G6: L6 delegation | Permit bounded multi-step execution | Sustained L5 safety/outcome evidence, action budgets, runtime monitoring, stop/rollback drills, governance approval |

## 15. Staged route to production

### Stage 0 — Frame and govern

- Select one high-frequency, low-consequence question set for one process and role.
- Map data flows, ownership, lawful basis, permissions, retention, risks, and baseline user outcome.
- Inventory source quality and define the evaluation set before choosing a provider.
- Establish product, domain, security, privacy, and operations ownership.

**Exit:** signed use-case charter, threat/privacy assessment, source readiness, baseline metrics, and measurable success criteria.

### Stage 1 — Offline read-only prototype

- Implement authorized context contracts, deterministic queries, retrieval, model gateway, citations, and abstention.
- Compare hosted and private/local candidates on the same blinded workload; include a non-LLM/search baseline.
- Use synthetic or tightly controlled data; no production writes.

**Exit:** quality, privacy, latency, and cost thresholds met by at least one configuration; failure modes documented.

### Stage 2 — Internal/shadow pilot

- Run against production-like authorized data for approved users, read-only.
- Validate end-to-end audit, source deletion/ACL changes, tenant isolation, monitoring, and support playbooks.
- Shadow existing work so outputs do not affect process decisions.

**Exit:** no unresolved critical privacy/security findings, acceptable grounded accuracy and abstention, stable SLOs, positive user utility.

### Stage 3 — Narrow tenant production (L1)

- Release to an explicit cohort with tenant controls, in-product feedback, and clear limitations.
- Monitor segmented quality, privacy signals, cost, latency, and actual task outcomes.
- Maintain rapid provider/model/prompt rollback and a search/manual fallback.

**Exit:** sustained target performance and trust, supportability, and evidence that the assistant improves the chosen job.

### Stage 4 — Guided work and insight (L2–L3)

- Add deterministic conformance findings, checklists, and carefully validated insight models.
- Test recommendations in shadow mode and measure downstream outcomes and human factors.

**Exit:** high process-adherence rate, calibrated detection, useful explanations, and no material increase in unsafe reliance.

### Stage 5 — Recommendations (L4)

- Present ranked actions, alternatives, expected effects, uncertainty, and approval route.
- Keep execution manual while collecting accept/edit/reject and realized-outcome evidence.

**Exit:** recommendations outperform baseline decisions within risk tolerances and approval policy is validated.

### Stage 6 — One controlled action (L5)

- Choose one reversible, low-risk, high-volume command.
- Require preview and explicit approval; enforce fresh authorization, preconditions, idempotency, audit, and compensation.
- Canary by tenant/cohort with conservative limits and kill switch.

**Exit:** sustained safe execution, successful recovery drills, low exception rate, and explicit governance decision before adding another action.

### Stage 7 — Bounded delegation (L6, optional)

- Define an allowlisted plan, time/action/cost budgets, checkpoints, stop conditions, and escalation paths.
- Expand only action by action; never infer broad authority from prior approvals.

**Exit:** separately approved for each use case and tenant tier. Remaining at L1–L5 is a valid long-term product choice.

## 16. Initial narrow assistant proposal

Start with a **Process Navigator** for one governed, well-documented process. It answers:

- What is this task and why was it assigned to me?
- What verified step comes next?
- Which current procedure and policy sections apply?
- Which required inputs are missing?
- Who is the authorized owner or escalation role?
- What is blocking progress according to current process state?

It may read only the active user's authorized company context, approved knowledge, and selected process instance. It may not alter state, infer sensitive attributes, make legal conclusions, or answer without accessible evidence. Deterministic services compute ownership, missing fields, deadlines, and valid transitions; the model explains them with citations.

This assistant is narrow enough to evaluate rigorously, valuable without automation, and architecturally representative of later capabilities.

## 17. Minimum production readiness checklist for L1

- [ ] One named process, user cohort, purpose, owner, and outcome metric
- [ ] Authoritative source and version semantics documented
- [ ] Source-side tenant/row/field/edge authorization verified
- [ ] Context minimization, residency, retention, and deletion validated
- [ ] Threat model, privacy assessment/DPIA decision, and subprocessor review complete
- [ ] Golden, process, adversarial, privacy, and robustness datasets versioned
- [ ] Grounding, citation, fallback, latency, cost, and isolation gates passed
- [ ] Model/provider routing policy and contractual data-use controls approved
- [ ] Full structured audit trace and user-visible “as of” state available
- [ ] Monitoring, incident handling, rollback, and manual/search fallback tested
- [ ] User feedback and knowledge-owner correction workflows operating
- [ ] Tenant opt-in/configuration, documentation, and support readiness complete

---

### Charter review cadence

Review this charter at each capability gate and at least quarterly during active development. Revisit it after a material change in use case, jurisdiction, tenant tier, model/provider, data source, tool, autonomy level, or regulatory interpretation.
