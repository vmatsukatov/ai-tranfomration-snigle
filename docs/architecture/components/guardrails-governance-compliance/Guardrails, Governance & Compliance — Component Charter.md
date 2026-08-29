# Guardrails, Governance & Compliance — Component Charter

**Status:** Architecture charter / “empty box”  
**Version:** 0.1  
**Date:** 2026-08-29  
**Owners:** Executive Risk Owner, DPO/Privacy, CISO/Security, AI Governance Lead, Platform Engineering  
**Applies to:** Every human, service, model, workflow, connector, and AI agent in the multi-tenant AI process platform

## 1. Purpose and compliance boundary

This component is the platform's decision and assurance boundary. It defines and, in later implementation, enforces what each human or machine principal may **see, infer, recommend, change, approve, or execute**, under which purpose, against which customer/process/data context, and with what evidence.

This charter starts with policy architecture and threat modeling. It specifies requirements and interfaces, not a control implementation.

> **No compliance claim:** Implementing this charter, or any set of technical controls, does not by itself make the platform or a tenant GDPR-compliant. Compliance depends on facts, purposes, lawful bases, contracts, notices, organizational practice, jurisdiction, and ongoing operation. Qualified privacy counsel, the relevant DPO, and security specialists must validate the decisions identified in this charter. The EU AI Act and sector, employment, consumer, discrimination, records, and cybersecurity law may apply in addition to GDPR.

### Outcomes

The component shall:

1. turn governance decisions into explicit, versioned, testable policy;
2. apply least privilege, purpose limitation, data minimization, tenant isolation, and bounded agency consistently;
3. keep untrusted content from becoming authority;
4. require proportionate human control for consequential or irreversible actions;
5. generate trustworthy evidence of inputs, decisions, approvals, actions, and outcomes;
6. support privacy rights, investigations, regulatory duties, and customer assurance;
7. fail safely when identity, context, policy, classification, or evidence is missing or stale.

### Non-goals

This component does not:

- choose a tenant's lawful basis, determine controller/processor status, or provide legal advice;
- replace enterprise identity, key management, data loss prevention, security monitoring, case management, or records systems;
- decide whether an AI use case is lawful, fair, or socially acceptable without accountable human review;
- treat model confidence, natural-language instructions, or possession of a link as authorization;
- silently resolve conflicts between legal, contractual, tenant, and platform requirements.

## 2. Governing principles and invariants

The words **MUST**, **SHOULD**, and **MAY** are normative.

1. **Deny by default.** Missing, ambiguous, unverifiable, expired, conflicting, or stale context MUST deny the operation or route it to an approved safe fallback.
2. **Tenant isolation is non-bypassable.** Tenant scope MUST be bound at authentication, storage, retrieval, cache, model context, tool invocation, logging, export, and support access. Tenant-supplied policy may only narrow platform isolation guarantees.
3. **Policy before data.** Authorization MUST occur before discovery, retrieval, inference, transformation, disclosure, or side effects—not only before displaying results.
4. **Untrusted data is never authority.** Prompts, documents, web pages, tool output, retrieved text, metadata, and model output MUST NOT create permissions, change policy, self-approve, or expand tool scope.
5. **Authorization is operation-specific.** Permission to see a source does not imply permission to infer a sensitive attribute, recommend an action, modify a record, or execute a process.
6. **Purpose is enforceable context.** Every personal-data operation MUST carry an approved purpose and use-case identifier; “general assistance,” model improvement, or future reuse is not a sufficient catch-all.
7. **Minimum necessary context.** Only the fields and passages necessary for the authorized step enter retrieval or model context. The Process Graph is a scoping input, not a route around source authorization.
8. **No self-escalation.** Agents cannot grant themselves roles, widen objectives, create approval evidence, or change their own budget, policy, or stop conditions.
9. **Human approval must be meaningful.** Approvers need authority, competence, relevant evidence, time, and an intelligible preview of effects. Approval cannot be buried in a general click-through.
10. **Irreversibility increases control.** External, high-impact, bulk, privileged, destructive, financial, legal, employment, safety, or rights-affecting operations require stronger assurance and usually explicit approval.
11. **Evidence is part of the transaction.** An action that requires an audit record MUST fail closed if the required evidence cannot be durably recorded, except under a documented break-glass rule.
12. **Privacy-preserving observability.** Logs MUST contain enough evidence to reconstruct decisions without becoming an unrestricted shadow copy of prompts, documents, secrets, or personal data.
13. **Policy changes are controlled changes.** Versioning, review, testing, rollout, rollback, and provenance apply to policy, prompts, models, tools, classifications, and approval matrices.
14. **Continuous assurance.** Pre-release testing, runtime monitoring, incident learning, and periodic review are required; a one-time assessment is insufficient.

## 3. Context dependencies and contracts

Earlier platform components MUST expose the following authoritative, immutable-at-decision-time context. The guardrails component consumes it but does not invent it.

| Source component | Required context | Required guarantees |
|---|---|---|
| Company Structure | tenant, legal entity, business unit, team, reporting/delegation relationships, customer/account ownership, geographic and employment context | stable IDs; effective dates; source provenance; no authorization from display names; change events; historical reconstruction |
| Identity | person/service/agent ID, authentication strength, tenant memberships, assigned and activated roles, service ownership, session/risk state, delegation | tenant-bound identities; short-lived credentials; workload identity for agents; MFA/step-up claims; revocation; no shared principals |
| Knowledge & classifications | asset/field/chunk classification, personal/special-category/child/criminal/secret markers, residency, retention class, permitted purposes, source owner, legal hold | classifications inherited through derivation; field/chunk granularity where needed; “unknown” treated as restricted; lineage and confidence; review workflow |
| Process Graph | process and instance ID, step, state, owner, participants, customer, input/output objects, allowed transitions, effect/risk tier, approval nodes | versioned definitions; immutable instance history; transition preconditions; idempotency; compensation/rollback semantics; no graph edge grants data access by itself |
| Tools/connectors | declared capabilities, input/output schemas, effect class, tenant/account binding, scopes, reversibility, rate/budget limits, data destinations | allowlisted versions; isolated credentials; typed calls; dry run where possible; receipts; bounded network/data access |
| Model/runtime | model and prompt versions, capabilities, region, provider, training/data-use terms, context window, safety configuration | approved inventory; change notification; tenant-safe caching; no provider training on tenant data unless explicitly approved and legally validated |

Every protected object and event MUST use opaque, stable IDs and carry `tenant_id`. Human-readable names and model-produced identifiers are display-only until resolved and re-authorized.

### Mandatory integration sequence

Every earlier component MUST design for the following sequence:

`authenticate principal → resolve authoritative context → classify requested operation → authorize → minimize/redact → perform guarded operation → validate result → approve if required → commit/execute → record evidence → monitor outcome`

No component may call a model, retrieval service, or effectful tool first and “check permission later.”

## 4. Policy architecture

### 4.1 Layers

Policy is evaluated as intersecting constraints:

1. **Law and regulatory obligations** — encoded only after qualified interpretation for the relevant jurisdiction and use case.
2. **Platform baseline** — non-waivable tenant isolation, security, abuse, and evidence controls.
3. **Contract and deployment** — data-processing terms, region, subprocessor, feature, and model/provider restrictions.
4. **Tenant governance** — tenant risk appetite, roles, classifications, purposes, approval matrices, retention, and prohibited uses.
5. **Process policy** — allowed actors, data, transitions, tools, budgets, and required approvals for a Process Graph version.
6. **Object/data policy** — classification, owner, customer, residency, consent/objection restrictions, legal hold, and permitted purposes.
7. **Runtime risk policy** — authentication strength, session risk, anomaly state, incident containment, and temporary restrictions.

The effective decision is the most restrictive applicable result. An explicit non-waivable deny wins. Conflicts MUST be surfaced with policy IDs; they MUST NOT be resolved by a model.

### 4.2 Decision tuple

Every decision MUST evaluate at least:

```text
Decision(
  tenant,
  principal = person | service | agent,
  principal_roles,
  customer/account,
  process_definition + version,
  process_instance + step + state,
  resource + field/chunk classification + provenance,
  declared purpose + lawful-basis record reference,
  operation = discover | see | retrieve | infer | recommend |
              create | change | approve | execute | export | delete,
  tool + capability + destination,
  effect/risk tier,
  time + region + channel + authentication/risk context,
  policy bundle version
) -> allow | deny | transform | require_approval | require_step_up | defer
```

The decision response MUST include a reason code, matched policy versions, obligations (for example redaction, row limit, approval, watermark, retention), decision ID, and expiry. It MUST NOT expose sensitive policy internals to an unauthorized caller.

### 4.3 Authorization model

Use a hybrid model:

- **RBAC** for understandable baseline job permissions;
- **ABAC** for tenant, customer, classification, purpose, geography, time, session, and effect conditions;
- **relationship-based controls** for Company Structure and customer/process ownership;
- **capability grants** for narrowly scoped, time-bound agent and tool authority;
- **process/state authorization** for transitions in the Process Graph.

A role alone MUST NOT authorize sensitive data or action. Agents MUST receive task-specific capability grants constrained by tenant, process instance, purpose, resources, tools, effect class, budget, expiry, and approval status. Delegation MUST be explicit, bounded, attributable, revocable, and no broader than the delegator's active authority.

### 4.4 Control and evidence planes

The future implementation should separate:

- **Policy administration:** authoring, legal/control mapping, validation, review, signing, publication, rollback.
- **Policy decision:** deterministic evaluation of trusted context and applicable versions.
- **Policy enforcement:** gateways at query, retrieval, prompt assembly, output, tool, workflow transition, export, and support-access boundaries.
- **Evidence:** append-only decision/action receipts, approval chains, provenance, policy snapshots, and integrity verification.
- **Assurance:** test corpora, simulations, red-team harnesses, metrics, drift detection, and release gates.

Policy enforcement MUST NOT rely solely on system prompts or model behavior. Deterministic enforcement must sit outside the model at every consequential boundary.

## 5. Guardrail lifecycle

| Stage | Mandatory guardrails | Default failure behavior |
|---|---|---|
| Input | authenticate and tenant-bind; malware/content scan; schema/size/type validation; detect secrets and sensitive data; classify; establish purpose and provenance; treat instructions inside content as untrusted | reject, quarantine, redact, or request authorized clarification |
| Retrieval | authorize before search and again before fetch; tenant/customer/process filters at the data layer; field/chunk controls; purpose and residency filters; top-k/context limits; source allowlist; no unauthorized existence leakage | return no data and a safe reason code |
| Context assembly | preserve instruction/data boundaries; label source trust; minimize; redact/tokenize; exclude hidden metadata and unrelated conversation memory; attach provenance and policy obligations | do not invoke model |
| Model generation/inference | approved model/prompt; constrained task and output schema; no authority from retrieved content; inference policy for sensitive attributes; timeout/token/cost limits; uncertainty and citation requirements where relevant | abstain or provide non-sensitive safe response |
| Output | validate schema, classification, destination, factual support, secrets/PII, harmful or prohibited content, and authorization to disclose; prevent reconstruction through aggregation; label AI-generated recommendations | redact, block, downgrade, or require review |
| Tool planning | allowlisted typed capability; arguments generated as a proposal only; bind tenant/account; calculate effect/risk tier; simulate/dry-run; enforce rate, value, record-count, and time limits | deny or require approval |
| Approval | show intended action, target, data, diffs, uncertainty, downstream effects, reversibility, policy reasons, and conflicts; bind approval to exact action hash and expiry | no execution |
| Execution | re-authorize immediately before commit; verify unchanged action hash/state; use least-privilege credential; idempotency key; transaction/compensation where possible; no chained expansion of scope | stop safely; roll back/compensate if possible |
| Post-action | verify outcome; record receipt; alert on deviation; update Process Graph state only after confirmed effect; support correction/reversal and incident linkage | mark uncertain, stop dependent steps, escalate |

### 5.1 Inference controls

Inference is a distinct operation. The system MUST separately govern:

- deriving personal, sensitive, special-category, behavioral, performance, health, political, union, biometric, criminal, or vulnerability attributes;
- joining sources that create a more sensitive view than either source alone;
- cohorting, ranking, scoring, profiling, predicting, or recommending about a person;
- using proxies or correlations to recreate restricted attributes;
- persisting inferred attributes or feeding them back into retrieval, training, evaluation, or Company Structure.

Absent an approved use case, necessary legal review, and explicit policy, restricted inference MUST be prohibited. A disclaimer that an inference “may be wrong” is not a safeguard.

### 5.2 Effect tiers

Each use case and tool capability MUST have a reviewed tier:

| Tier | Typical effect | Default control |
|---|---|---|
| E0 | read-only on public/non-sensitive data; no external effect | policy check and evidence |
| E1 | internal draft/recommendation; reversible; low sensitivity | user confirmation where context could be misunderstood |
| E2 | internal record change, external communication, moderate personal/confidential data, or bounded workflow transition | explicit approval and immediate re-authorization |
| E3 | bulk, privileged, financial, contractual, employment, legal, safety, deletion, high-sensitivity, rights-affecting, or materially irreversible action | maker-checker separation, step-up auth, simulation, narrow limits, rollback plan, enhanced evidence |
| E4 | prohibited or outside approved risk appetite | block; only policy change through formal governance can reclassify |

Tiering is contextual: an individually harmless action may become E3 through scale, accumulation, timing, destination, or vulnerable population.

## 6. Human control, separation of duties, and emergency stop

### Approval requirements

- The requester/agent, approver, and policy administrator MUST be distinct for E3 operations.
- No agent or model may approve its own proposal or interpret casual conversation as approval.
- Approval MUST be specific, informed, authenticated, time-bound, non-replayable, and bound to the exact action, targets, policy version, and material preview.
- Material changes after approval—including target set, content, price/value, data class, destination, tool version, or Process Graph state—invalidate approval.
- Batch approval requires bounded membership and a reviewable manifest; “approve all future similar actions” is not valid runtime approval.
- High-risk overrides and break-glass access require a reason, ticket/incident link, narrow duration/scope, enhanced logging, prompt notification, and retrospective independent review.
- Approvers MUST be protected from automation bias through calibrated uncertainty, source evidence, dissenting signals, and an obvious reject/edit/stop path.

### Escalation

Policy must map reason codes to accountable queues: data owner, process owner, security, privacy/DPO, legal, tenant administrator, or executive risk owner. Escalation MUST not disclose the contested data to people who lack access. Service-level objectives, ownership, outcome, and expiry MUST be recorded.

### Emergency stop

The platform MUST support independently authorized stop controls at agent, tool, process instance, tenant, connector, model/provider, region, and platform levels. Stop controls must:

- revoke queued and future capabilities and credentials;
- prevent new tool commits and pause dependent Process Graph transitions;
- preserve evidence and in-flight state;
- avoid corrupting transactions; use a defined safe state or compensation;
- propagate within a tested maximum time;
- be operable without the affected model or normal policy deployment path;
- require controlled, documented recovery rather than automatic restart.

## 7. Threat model

### Assets and trust boundaries

Protected assets include tenant/customer data, personal and inferred data, prompts and policies, credentials, connector scopes, models and indexes, Process Graph state, decisions/approvals, audit evidence, and availability of safety controls.

Trust boundaries exist between tenants; tenant and platform operations; human and agent; model and deterministic controls; retrieval stores and prompt context; platform and model/tool providers; control and evidence planes; production and analytics/testing; regions; and current versus historical Company Structure.

### Required abuse cases and mitigations

| Threat | Abuse case | Architectural requirements | Assurance evidence |
|---|---|---|---|
| Direct/indirect prompt injection | user, document, web page, email, image, or tool output tells the agent to ignore policy or leak/act | instruction/data separation; source trust labels; deterministic authorization; constrained tools; no secret in model context unless necessary; output and destination controls | adversarial corpus across modalities/languages; injection canaries; zero unauthorized effects |
| Exfiltration | attacker asks for secrets directly, encodes them, reconstructs them over turns, or sends them through a tool/URL | egress allowlist; DLP; aggregation/query budgets; tenant-bound memory/cache; destination authorization; redact secrets; anomaly detection | direct, encoded, fragmented, and multi-turn exfiltration tests |
| Cross-tenant leakage | bad filter, shared cache/index, confused identity, support access, log/analytics join, model memory | tenant partitioning and cryptographic/context binding; data-layer filters; per-tenant cache keys/index namespaces/keys where warranted; negative authorization tests | isolation tests at storage, retrieval, cache, logs, exports, backups, analytics, support |
| Excessive agency | agent expands goal, spawns unbounded work, chains tools, changes policy, spends/transacts excessively | scoped capability grants; allowlisted plans; call/depth/time/cost/value/record limits; no self-modifying policy; approvals; stop controls | long-horizon simulation; budget and recursion tests; forbidden-chain coverage |
| Unsafe automation | plausible but wrong output triggers consequential workflow; stale state or duplicate execution | separate propose/approve/execute; E-tiering; state preconditions; idempotency; dry-run; independent validation; outcome verification | duplicate/stale/race/failure-injection tests; rollback exercises |
| Confused deputy | authorized service is induced to use its privilege for an unauthorized user/tenant/purpose | end-to-end principal/tenant/purpose propagation; on-behalf-of token; re-authorization at each hop; no ambient credential | delegation and confused-deputy tests |
| Poisoned knowledge/provenance | malicious or stale content changes recommendations or instructions | ingestion trust policy; provenance; integrity/version checks; quarantine; freshness; citations; source-owner correction | poisoning, stale-source, provenance-loss tests |
| Sensitive inference | model infers restricted traits or scores people from permitted data | inference treated as authorization action; prohibited attribute/proxy rules; output controls; fairness and rights assessment | inference/proxy red-team and disparate-impact evaluation where lawful/appropriate |
| Privilege/policy drift | org change, role accumulation, stale grants, policy regression | effective-dated structure; revocation propagation; access review; policy diff and simulation; short-lived grants | joiner/mover/leaver tests; stale-context chaos tests |
| Evidence tampering/overcollection | attacker edits logs, or logs become a new privacy breach | append-only integrity; access separation; minimization; encryption; retention/disposal; clock synchronization; export controls | integrity verification; deletion/hold tests; audit reconstruction drill |
| Availability/safety bypass | policy service outage prompts permissive fallback; attacker disables stop/logging | fail closed; local signed safe baseline where justified; isolated stop path; backpressure; no silent bypass | dependency outage, partition, latency, and kill-switch exercises |
| Supply-chain/provider risk | model/tool/subprocessor changes behavior, region, terms, or data use | approved inventory; version pinning; contractual restrictions; provider evaluation; change gates; exit plan | provider attestations plus independent validation; migration/revocation exercise |

Threat modeling MUST be repeated per use case and Process Graph version, including malicious insiders, compromised accounts/tools, honest mistakes, vulnerable data subjects, and failures without an attacker.

## 8. Governance, roles, and accountability

| Role | Accountable duties | Must not be solely responsible for |
|---|---|---|
| Board/executive risk owner | risk appetite; prohibited uses; residual E3 risk acceptance; resources; oversight | day-to-day approval of own sponsored use case |
| Tenant executive/accountable owner | tenant purposes, use-case sponsorship, customer obligations, operational accountability | overriding platform isolation or law |
| DPO/privacy function | independent advice and monitoring; DPIA advice; privacy rights and regulator interface | business purpose ownership; automatic approval of all processing |
| Qualified legal counsel | legal roles/bases; notices; Article 22; special categories; transfers; contracts; AI Act/sector law interpretation | technical security acceptance |
| CISO/security owner | threat model, security baseline, incident readiness, residual security risk advice | self-attestation by implementation team |
| AI governance/risk committee | use-case inventory and tier; model/tool approval; fairness/harm review; assurance criteria; exceptions | replacing DPO, counsel, data owner, or executive accountability |
| Data owner/steward | classification, quality, permitted purposes, retention/hold input, access review | authorizing beyond tenant/legal constraints |
| Process owner | intended purpose, Process Graph controls, effect tier, outcomes, approvers, rollback, monitoring | approving own E3 action as sole approver |
| Platform/control owner | control design, policy service, evidence, testing, operation | unilateral risk acceptance or legal interpretation |
| Model/tool owner | inventory, limitations, change notices, evaluations, provider management | production release approval alone |
| Independent assurance/internal audit | design and operating-effectiveness review; evidence sampling; issue tracking | building controls being audited |
| Incident commander | containment, coordination, decision log, recovery | deleting or editing evidence |

Named people, deputies, competence requirements, conflicts of interest, and approval limits MUST be maintained per tenant and platform. Material exceptions require an owner, justification, compensating measures, expiry, evidence, and independent review; no permanent silent exceptions.

## 9. Audit evidence, provenance, and records

### Decision/action receipt

For each material operation, record at least:

- event/trace/decision IDs; timestamp and trusted clock source;
- tenant, principal, workload identity, on-behalf-of person, session/authentication level;
- customer, process definition/version, instance/step/state, purpose/use-case ID;
- operation, resource IDs and classifications (content only when necessary), tool/capability/destination;
- policy bundle and rule IDs, context versions, decision, obligations, reason code;
- model/provider/version, prompt-template hash, retrieval source IDs/versions, output hash and classification;
- proposed action and diff/manifest hash; approval identities, authority, timestamps, decision and expiry;
- execution credential/capability ID, idempotency key, tool receipt, result, verification, rollback/compensation;
- incident, exception, DPIA, legal assessment, contract, and retention-class references where applicable.

### Evidence properties

Evidence MUST be append-only or tamper-evident, encrypted, access-controlled separately from production administration, tenant-segregated, searchable by authorized investigators, exportable in a documented format, time-synchronized, and integrity-verifiable. Privileged access and evidence exports are themselves audited.

Do not log complete prompts, retrieved passages, model outputs, tokens, secrets, or special-category data by default. Prefer IDs, hashes, classifications, structured decision facts, and selectively protected payload escrow only where necessity and proportionality are documented. Debug mode MUST NOT weaken tenant or privacy boundaries.

### Retention and disposal

There is no single universal retention period. A reviewed schedule MUST reconcile purpose, minimization/storage limitation, limitation periods, security/incident needs, contracts, sector rules, AI Act duties where applicable, tenant deletion commitments, backups, legal holds, and data-subject rights. Retention rules must apply to logs, model traces, indexes, caches, evaluations, exports, backups, and derived/inferred data—not only source records.

Deletion MUST be verifiable and propagated to derived stores unless a documented lawful exception or legal hold applies. Holds must be scoped, approved, reviewable, and released. Counsel and the DPO MUST approve the schedule and conflicts; engineering implements it.

### Reporting

Provide role-appropriate reporting for tenants and platform governance: denied/approved/executed actions, break-glass use, policy exceptions, overdue reviews, access recertification, cross-tenant probes, injection/exfiltration attempts, data rights SLA, deletion completion, residency exceptions, model/tool changes, incidents, assurance coverage, and residual risk. Metrics must not create incentives for superficial approvals or suppress incident reporting.

## 10. Privacy, data protection, and regulatory interfaces

### Privacy by design and default

Each use case MUST maintain a processing record linked to: specific purpose; categories of data and people; source; necessity/proportionality; legal-role and lawful-basis decisions; recipients; regions/transfers; retention; rights handling; security measures; model/tool providers; and approved Process Graph version. Defaults MUST minimize collection, access, context, retention, audience, automation, and external effects.

The system MUST support consent/withdrawal where consent is the chosen valid basis, objections/restrictions where applicable, and policy changes without assuming consent is always required or always appropriate. Legal basis and purpose compatibility are qualified legal decisions, not model classifications.

### DPIA and related assessment triggers

A mandatory pre-production screening must route to the DPO/privacy function when processing may create high risk, including combinations of:

- systematic evaluation, scoring, profiling, ranking, or prediction about people;
- solely or materially automated decisions with legal or similarly significant effects;
- special-category or criminal-offence data, highly sensitive confidential data, or inferred equivalents;
- large-scale monitoring, joining datasets, novel AI, opaque inference, vulnerable people, children, employees, or power imbalance;
- location/behavior monitoring, biometrics, denial of service/opportunity, or inability to exercise a right;
- a new purpose, model/provider, tool capability, data category, recipient, region/transfer, scale, or materially changed Process Graph;
- residual high privacy risk after mitigation.

Where the screen or DPO determines a DPIA is required, production use MUST remain blocked until the DPIA is completed, measures are assigned, residual risk is accepted by authorized humans, and any required prior consultation is resolved. A fundamental-rights assessment, legitimate-interest assessment, transfer impact assessment, security threat model, or AI Act conformity/risk assessment may also be required; one does not automatically replace another.

### Automated decisions and human review

Use cases affecting employment, access to services, finance, legal rights, safety, or comparable significant interests require counsel/DPO review of GDPR Article 22 and other applicable law before design approval. “Human in the loop” is not automatically meaningful human involvement. Reviewers must have authority, competence, relevant information, independence, and the ability to change the outcome. The platform must support notice, contestability, correction, explanation appropriate to the context, and traceable reconsideration when required.

### Data-subject rights

The data inventory and provenance graph MUST enable authorized staff to locate, access/export, rectify, restrict, erase, or isolate personal data and derived/inferred data across operational stores, vector/search indexes, caches, logs, evaluations, prompts/traces, and backups as applicable. It must support identity verification, scope/jurisdiction assessment, third-party rights and exemptions, deadlines, holds, processor-to-controller routing, fulfillment evidence, and non-reintroduction after deletion. Qualified privacy/legal review decides the applicable right and exceptions.

### Subprocessors, residency, and transfers

Maintain a machine-readable inventory of every model, hosting, observability, support, security, and tool provider that may process tenant data, including role, purpose, data classes, regions, transfer route, onward subprocessors, retention/deletion, provider training/data-use setting, incident terms, audit evidence, and exit process.

Policy MUST block a provider, region, or transfer not approved for the tenant/data class/purpose. Procurement, counsel, privacy, and security review are required before onboarding or material change. Standard contractual clauses, adequacy, supplementary measures, localization, and transfer impact are legal/factual determinations; a region selector or encryption alone is not proof of lawful transfer.

## 11. Incident response and operational resilience

Security/privacy/AI incidents include suspected cross-tenant access, personal-data breach, injection with impact, unauthorized inference/disclosure/action, tool misuse, unsafe decision, evidence failure, policy bypass, provider incident, repeated near miss, and inability to honor a stop or right.

The response design MUST provide:

1. detection and severity criteria, including impact to people rather than only system uptime;
2. one-action containment using scoped emergency stops and credential revocation;
3. preservation of tamper-evident evidence and a decision timeline;
4. tenant, DPO, legal, security, provider, insurer, executive, and authority notification paths;
5. jurisdiction- and contract-aware clocks without hard-coding a universal reporting conclusion;
6. impact assessment, affected tenant/person/data/process identification, and safe remediation;
7. recovery criteria, independent authorization to resume, and heightened monitoring;
8. root-cause analysis across technical, organizational, provider, and human factors;
9. policy/test updates and tracked corrective actions.

Counsel and the DPO determine whether an event is a personal-data breach, whether notification is required, to whom, and when. Applicable AI Act/sector reporting must be separately assessed. At least annual tabletop exercises should include cross-tenant leakage, compromised connector, malicious insider, unsafe bulk action, provider outage, and failed evidence pipeline.

## 12. Policy-as-code

### Requirements

Policy artifacts MUST be:

- declarative, deterministic, typed, schema-validated, human-reviewable, and machine-enforceable;
- immutable and content-addressed once published, with semantic version, owner, rationale, control/legal mapping, effective/expiry dates, and signatures;
- scoped by platform/tenant/use case/process and protected from tenant widening of non-waivable baselines;
- developed with peer review, separation of author/approver/deployer for high-risk rules, CI tests, policy diff, impact simulation, staged rollout, monitoring, and rollback;
- evaluated against authoritative context, never facts invented by the model;
- able to return structured obligations and stable reason codes, not just boolean access;
- portable enough to test locally and reconstruct historical decisions with the original policy and context snapshots.

Emergency policy changes require the same integrity and evidence, even if review is retrospective under an approved incident procedure. Natural-language policy may explain intent but is not the enforcement artifact.

### Example decision intent (non-implementation)

```yaml
request:
  tenant: t-123
  principal: agent:invoice-assistant
  on_behalf_of: person:p-456
  customer: c-789
  process: invoice-resolution@7 / instance:i-42 / step:propose-credit
  purpose: resolve-disputed-invoice
  operation: recommend
  resources: [invoice:8842, contract:991]
  classifications: [personal, customer-confidential]
  tool: none
  effect_tier: E1
decision:
  result: allow
  obligations: [minimize-fields, cite-sources, label-as-draft, no-sensitive-inference]
  policy_bundle: sha256:...
  expires_at: ...
```

Changing `operation` to `execute`, changing customer, adding payroll data, or adding an email/payment tool requires a new decision; the prior allow cannot be reused.

## 13. Evaluation and assurance strategy

### Evaluation layers

1. **Policy unit/property tests:** allow/deny boundaries, precedence, missing context, non-interference, temporal rules, delegation, and obligation output.
2. **Golden authorization matrix:** representative tenant × person/agent × role × customer × process × data class × purpose × action combinations, including negative pairs.
3. **Isolation tests:** storage, search/vector retrieval, cache, memory, logs, analytics, backups, exports, support, and provider paths.
4. **Adversarial AI tests:** direct/indirect/multimodal/multilingual injection; encoding and fragmentation; data poisoning; sensitive inference; authority spoofing; approval manipulation; tool-output injection.
5. **Agent/action tests:** excessive planning depth, goal drift, tool chaining, stale state, duplicate requests, races, partial failures, budget exhaustion, approval replay, idempotency, rollback, and stop latency.
6. **Privacy/rights tests:** minimization, purpose change, withdrawal/objection/restriction flags, search completeness, rectification, deletion propagation, legal hold, export, reintroduction prevention.
7. **Operational tests:** policy/evidence/identity outage, partition, clock drift, revocation propagation, key rotation, provider change, backup restore, incident drills.
8. **Human-factors tests:** approver comprehension, automation bias, alert fatigue, accessibility, rejection/appeal usability, and escalation handling.
9. **Independent assurance:** design review, penetration test, tenant isolation assessment, code/config review, evidence sampling, DPIA/control validation, and remediation closure.

### Release metrics and gates

Each use case needs predeclared thresholds tied to harm and effect tier. At minimum measure:

- unauthorized disclosure/action rate (release target: zero in test corpus);
- cross-tenant isolation failures (zero tolerance);
- injection attack success resulting in protected disclosure/effect (zero tolerance);
- false allow and false deny by policy reason and affected group/context;
- approval bypass/replay and stale-state execution (zero tolerance);
- emergency-stop propagation and safe-state success;
- evidence completeness/integrity and reconstruction success;
- rights-search/deletion completeness and SLA;
- model/tool quality, unsupported claims, uncertainty calibration, and human override;
- drift after model, prompt, policy, data, connector, or Process Graph changes.

Passing a benchmark does not prove safety or compliance. Test coverage, residual risk, limitations, and known failures must accompany results. Any material change triggers risk-based re-evaluation and possibly renewed legal/privacy/security review.

## 14. Staged path to production assurance

### Stage 0 — Charter and risk framing

Deliver tenant/use-case inventory, prohibited-use baseline, roles, decision tuple, effect taxonomy, data classification contract, threat model, legal/privacy screening, and open decisions. No production data or effects.

**Exit:** executive risk owner, DPO/privacy, security, architecture, and counsel acknowledge scope, roles, and review triggers.

### Stage 1 — Policy simulation with synthetic data

Build schemas, policy repository, decision API contract, golden matrix, evidence schema, tool capability manifest, and offline adversarial harness. Use synthetic/non-personal data and read-only mock tools.

**Exit:** deterministic decisions, negative tests, policy history reconstruction, and fail-closed behavior demonstrated.

### Stage 2 — Shadow/read-only pilot

Run within one internal tenant and narrow process; compare policy decisions without external effects; validate tenant/data filters, provenance, minimization, monitoring, rights discovery, stop control, and support access.

**Exit:** DPIA/other required assessments approved; legal/contract/provider reviews completed; security testing remediated; evidence and incident drills pass.

### Stage 3 — Human-approved reversible actions

Permit a small allowlist of E1/E2 actions with explicit approval, strict limits, dry-run/diff, idempotency, rollback, outcome verification, and staffed escalation. No E3 automation.

**Exit:** operating-effectiveness evidence over an agreed period; human-factors results acceptable; tenant acceptance; rollback and breach exercises complete.

### Stage 4 — Limited production and tenant expansion

Add tenants/use cases separately; never infer approval by similarity. Maintain canary rollout, feature-level stop, provider/version pinning, continuous red team, access reviews, incident SLOs, and periodic governance reporting.

**Exit:** independent assurance, risk acceptance for each effect tier, capacity/resilience evidence, contractual readiness, and no unresolved critical findings.

### Stage 5 — Higher agency only by evidence

Consider narrowly bounded E3 capabilities only where benefits, reversibility, mature evidence, strong separation of duties, meaningful oversight, legal permissibility, and incident readiness are demonstrated. Keep E4 prohibited.

**Exit:** explicit executive, process/data owner, security, DPO/privacy, and qualified legal approval as applicable; independent pre-release review; continuous monitoring and rapid rollback.

## 15. Initial risk register

| Risk | Initial posture | Owner | Required treatment / decision |
|---|---|---|---|
| Cross-tenant data or action leakage | Critical / no tolerance | CISO + Platform | prove end-to-end isolation; independent test before any pilot |
| Incorrect legal role, basis, purpose, or Article 22 conclusion | Critical | Tenant controller + Counsel/DPO | use-case-specific documented determination; block unresolved use |
| Unsafe agent action from injection or hallucination | Critical | AI Governance + Process Owner | deterministic controls, bounded tools, approvals, adversarial and failure testing |
| Sensitive/proxy inference and discriminatory impact | High/Critical | Tenant Risk Owner + DPO/Legal | prohibit by default; impact/fairness assessment and contestability where considered |
| Overbroad connectors/provider data use | High | Procurement + Security/Privacy | capability inventory, contract/transfer review, data-use restrictions, exit plan |
| Audit logs become sensitive shadow dataset | High | Evidence Owner + DPO | minimize, segregate, schedule retention, test rights/deletion/hold behavior |
| Stale Company Structure or identity grants | High | Identity/Structure Owners | effective dating, revocation SLA, short-lived capabilities, recertification |
| Approval theater/automation bias | High | Process Owner + Human Factors | meaningful preview, competence, independence, sampled quality and override review |
| Policy/model/tool drift invalidates assurance | High | Change Advisory + AI Governance | material-change triggers, version pinning, regression/red-team gates |
| Emergency stop fails or leaves partial effects | High | Operations + Process Owner | independent stop plane, safe-state design, compensation, recurring exercises |
| Incomplete rights fulfillment in derived stores | High | Privacy + Data Owners | provenance inventory, end-to-end rights tests, processor routing |
| Residency/transfer mismatch | High | Legal/Privacy + Platform | machine-enforced region/provider policy backed by current legal assessment |

## 16. Open decisions

The following must be resolved before implementation commitments:

1. In each operating model, who is controller, joint controller, processor, or subprocessor for prompts, retrieval, logs, model evaluation, and agent actions?
2. Which jurisdictions, sectors, tenant types, vulnerable groups, and employee/customer use cases are in scope?
3. Which use cases are prohibited, E4, or require board/executive acceptance? Which may fall under EU AI Act high-risk or transparency rules?
4. What taxonomy defines personal, special-category, criminal, confidential, secret, inferred, and “unknown” data at field/chunk/derived-output level?
5. What is the canonical purpose vocabulary, and who approves purpose compatibility or changes?
6. How are customer/account relationships represented when one person or record belongs to multiple customers or legal entities?
7. What constitutes meaningful human review for each consequential Process Graph step?
8. Which tools and effect classes are permitted; what value, volume, recursion, time, and cost limits apply?
9. Which policy engine, enforcement topology, and tenant isolation model meet latency, availability, and historical reconstruction requirements?
10. What evidence payloads are necessary and proportionate, and what tenant/record-specific retention and legal-hold rules apply?
11. How will data-subject rights propagate through vector indexes, caches, logs, evaluations, derived features, model adaptation, and backups?
12. Which models/providers/regions/subprocessors are acceptable under contract, transfer, security, data-use, and exit requirements?
13. What assurance thresholds, review cadence, independent testing, and customer evidence are required by tier?
14. Who can activate/deactivate break-glass and emergency stops, and what maximum propagation and recovery times are acceptable?
15. How are policy conflicts, appeals, exceptions, and tenant requests to weaken defaults adjudicated?

## 17. Mandatory qualified review gates

### Qualified legal counsel and DPO/privacy review

Required before production for legal-role allocation; lawful basis and purpose compatibility; notices and transparency; consent where used; special-category/criminal data; automated decision-making/profiling and Article 22; children, employees, vulnerable people, and monitoring; DPIA necessity/content and residual high risk; data-subject rights and exceptions; retention/holds; processor/subprocessor terms; international transfers/residency; breach notification; EU AI Act role/classification/obligations; and applicable employment, discrimination, consumer, records, financial, health, safety, or sector law.

### Qualified security review

Required for tenant isolation, identity/delegation, policy bypass, retrieval and vector-store security, prompt injection, data exfiltration, tool/connector sandboxing, credential design, evidence integrity, cryptography/key management, provider/supply-chain risk, abuse monitoring, incident response, resilience, emergency stop, penetration testing, and independent production readiness.

### Joint multidisciplinary review

Required for effect tiering, sensitive inference, fairness and fundamental-rights impact, meaningful human oversight, user/approver experience, prohibited uses, residual risk acceptance, and material changes. Neither a legal sign-off nor a penetration test substitutes for this review.

## 18. Source framework and review baseline

This charter should be reviewed against current authoritative text at each legal/design gate:

- [General Data Protection Regulation, Regulation (EU) 2016/679](https://eur-lex.europa.eu/eli/reg/2016/679/oj) — especially principles, privacy by design/default, security, records, processors, rights, DPIAs, automated decisions, breaches, and transfers.
- [EDPB: Privacy by design and by default](https://www.edpb.europa.eu/topics/ai-and-technology/privacy-by-design-and-by-default_en) — data protection must be designed in from the start and reviewed continuously.
- [EDPB: Data protection impact assessment](https://www.edpb.europa.eu/topics/accountability-and-compliance-tools/data-protection-impact-assessment_en) — DPIAs precede processing likely to result in high risk to individuals' rights and freedoms.
- [EDPB Opinion 28/2024 on personal data in AI models](https://www.edpb.europa.eu/documents/opinion-of-the-board-art-64/opinion-282024-on-certain-data-protection-aspects-related-to_en) — anonymity, legitimate interests, and unlawfully processed training/development data require contextual assessment.
- [EU Artificial Intelligence Act, Regulation (EU) 2024/1689](https://eur-lex.europa.eu/eli/reg/2024/1689/oj) — applicability, role, risk classification, human oversight, logging, transparency, and other duties require separate review.
- [NIST AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework) and [Generative AI Profile](https://doi.org/10.6028/NIST.AI.600-1) — voluntary governance, mapping, measurement, management, content provenance, pre-deployment testing, and incident disclosure guidance.
- [OWASP Top 10 for LLM Applications 2025](https://owasp.org/www-project-top-10-for-large-language-model-applications/) — practical threat categories including prompt injection, sensitive information disclosure, excessive agency, and vector/embedding weaknesses.

These sources are a baseline, not an exhaustive legal or control catalog. Applicable law, regulatory guidance, standards, contracts, threat intelligence, and platform facts must be rechecked at each material change and scheduled review.

## 19. Definition of ready for implementation

The “empty box” is ready to become a designed component only when:

- all context contracts have owners and stable schemas;
- the authorization tuple, effect taxonomy, policy precedence, and fail-safe behavior are approved;
- initial use cases and prohibited uses are inventoried and tiered;
- tenant isolation and evidence architectures have passed independent design review;
- legal/privacy/security triggers are embedded in delivery gates;
- policy, evidence, approval, tool, emergency-stop, rights, and incident interfaces have acceptance tests;
- unresolved open decisions have accountable owners and due dates;
- no team treats this charter or future control test results as a legal compliance certificate.
