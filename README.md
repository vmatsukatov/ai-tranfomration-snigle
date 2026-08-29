# AI Process & Knowledge Platform

## Project vision

This project explores how a digital agency can connect a customer company's structure, knowledge, processes, rules, routines, and business systems with AI in a secure and GDPR-conscious way.

The agency already delivers web-based business tools that work alongside Microsoft 365, as well as AI chat and document-processing assistants. The next step is a reusable platform that organizes each customer's information and makes it safely available to AI models and AI-assisted workflows.

The agency itself will be used as the first test organization before the platform is adapted for customers.

## The problem

Company knowledge is usually distributed across people, folders, documents, Microsoft 365, line-of-business applications, hosting environments, and cloud services. A general-purpose AI assistant does not automatically understand:

- how a particular company is organized;
- which people and roles are responsible for specific work;
- how customers, suppliers, systems, and resources are related;
- which rules and procedures apply in a particular situation;
- which information a user or AI agent is permitted to access;
- where information came from and whether it is current;
- when an AI recommendation requires human approval.

This project aims to provide that missing organizational and process layer.

## Core principles

- **Structure first:** Company structure and relationships provide the shared context for data, processes, permissions, and AI behavior.
- **Multi-tenant by design:** Every customer's identities, data, indexes, policies, and audit records must remain isolated.
- **GDPR and privacy by design:** Data minimization, purpose limitation, retention, deletion, provenance, access control, and auditability must be designed into every component.
- **Evidence before model choice:** RAG, MCP tools, local models, hosted models, conventional ML, and fine-tuning are options to evaluate—not predetermined solutions.
- **Authorization at retrieval and action time:** A model must not receive or act on information merely because it exists in an index.
- **Human control:** High-impact recommendations and actions require explicit approval, traceability, and safe fallback behavior.
- **Incremental delivery:** Begin with one organization, one narrow process, and measurable outcomes before increasing scope or autonomy.

## Component architecture

### 1. Company Structure & Ontology

The foundational model of the organization and its relationships.

Example entities include:

- CEO, CTO, project managers, HR, QA, technical leads, and developers;
- departments, teams, roles, responsibilities, and ownership;
- customers and customer technical contacts;
- hosting companies and other suppliers;
- Microsoft 365 tenants and accounts;
- Azure subscriptions, resources, applications, and environments;
- systems, services, assets, and their relationships.

This component provides stable identifiers and relationships that all other components can reference. It must be extensible because each customer will have a different organizational shape.

### 2. Knowledge Data & Connectors

The governed knowledge layer used to acquire, classify, normalize, store, index, synchronize, and retrieve customer information.

Potential sources include:

- uploaded files and folder structures;
- SharePoint, OneDrive, Teams, and other Microsoft 365 sources;
- databases and customer applications;
- APIs and MCP-accessible services;
- process documents, policies, contracts, manuals, and operational records.

Every knowledge object should retain tenant, ownership, provenance, classification, permission, version, retention, and company-structure metadata.

### 3. Process Graph

The representation of how work is designed and how it actually happens.

The graph can connect:

- processes, cases, tasks, events, and states;
- actors, roles, teams, customers, and suppliers;
- rules, decisions, approvals, and exceptions;
- systems, documents, inputs, outputs, and evidence;
- designed workflows and observed process instances.

A graph database is one possible implementation, but it should only be selected after the access patterns and pilot use case show that it is justified.

### 4. Process Intelligence

The assistance and insight layer that uses company structure, authorized knowledge, and current process state.

Its capabilities may grow in controlled stages:

1. Read-only questions and cited answers.
2. Process guidance and next-step suggestions.
3. Detection of missing information, risks, exceptions, or bottlenecks.
4. Recommendations requiring human approval.
5. Limited execution of explicitly authorized actions.

Deterministic workflow logic should handle known rules and state transitions. LLM reasoning should be used where interpretation is required, while conventional ML may be appropriate for classification, prediction, or anomaly detection when sufficient validated data exists.

### 5. Guardrails, Governance & Compliance

The control layer defining what each human or AI agent may see, infer, recommend, change, or execute.

It includes:

- tenant isolation and identity-based authorization;
- role-, purpose-, process-, and data-class-based policies;
- retrieval, prompt, output, tool-use, and action controls;
- approval gates and separation of duties;
- protection against prompt injection, data exfiltration, and cross-tenant leakage;
- logging, provenance, audit evidence, incident response, and emergency stop;
- retention, deletion, data-subject rights, and subprocessor governance.

Technical measures can support GDPR obligations but do not by themselves establish legal compliance. Qualified legal, privacy, and security review will be required.

## How the components connect

```text
Company Structure & Ontology
        |
        +--> Knowledge Data & Connectors
        |          |
        +----------+--> Process Graph
        |                    |
        +--------------------+--> Process Intelligence
        |
        +--> Guardrails, Governance & Compliance
                 |
                 +--> controls data access, AI context, tools, and actions
```

Guardrail requirements influence every component from the beginning, even though the full control layer will be implemented later.

## AI strategy: RAG, MCP, fine-tuning, and local models

These approaches solve different problems and can be combined:

- **RAG** supplies current, source-backed customer knowledge at request time. It is usually the first option for private and changing information.
- **MCP and tool integrations** allow controlled access to live systems and approved actions without copying all source data into a model or index.
- **Fine-tuning** can improve repeatable behavior, terminology, classification, or output format. It is generally not the right storage mechanism for changing private knowledge and creates additional training-data governance requirements.
- **Local or self-hosted models** can provide stronger deployment and data-residency control, but still require authorization, evaluation, monitoring, infrastructure, and lifecycle management.
- **Conventional ML** may be more suitable than an LLM for stable classification, prediction, ranking, or anomaly-detection tasks with sufficient quality data.

The initial hypothesis is a hybrid architecture: governed retrieval for knowledge, tools for live data and actions, deterministic logic for process enforcement, and carefully evaluated models for interpretation and assistance.

## Initial pilot

The agency will be the first tenant. The pilot should start with:

1. A minimal organizational structure and relationship model.
2. A small, representative set of governed documents and folders.
3. One clearly bounded internal process.
4. Read-only, cited AI assistance for that process.
5. Measurable evaluations for correctness, access control, privacy, usefulness, latency, and cost.

The first pilot process should be selected using explicit criteria: business value, manageable data sensitivity, clear ownership, observable outcomes, and low risk if the AI is wrong.

## Development sequence

1. Define the company ontology and tenant boundaries.
2. Define knowledge metadata, provenance, permissions, and connector contracts.
3. Model one process and its relationship to structure and knowledge.
4. Build a narrow read-only assistant with citations and evaluations.
5. Implement policy enforcement, auditing, and adversarial tests.
6. Introduce recommendations and approval-controlled actions only after the earlier stages are reliable.
7. Evaluate fine-tuning, local models, and conventional ML against specific measured requirements.

## Current status

The project is in the discovery and architecture phase. Each core component has a separate design workspace so it can progress from business definition to technical implementation through small, reviewable decisions.

No production architecture, model provider, vector database, graph database, or hosting platform has been selected yet.

