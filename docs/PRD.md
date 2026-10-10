# Product Requirements Document: WhatsApp AI Sales Agent SaaS

- **Status:** Draft for implementation planning
- **Product:** WhatsApp AI Sales Agent
- **Primary channel:** WhatsApp Business Platform / Cloud API
- **Delivery approach:** Audit the candidate implementation before deciding what to reuse.
- **Out of scope:** haircut salon management or unrelated vertical-specific software

## 1. Executive summary

Build a multi-tenant SaaS product that enables a business to configure an AI sales assistant on WhatsApp. The assistant answers common questions using approved business knowledge, explains products or services, qualifies leads, captures relevant details, and routes a conversation to a human when confidence is low or the customer requests a person.

This is not merely a chatbot wrapper. The product needs reliable messaging, tenant-aware business configuration, a trustworthy knowledge pipeline, a conversation lifecycle, operational controls, and measurable sales outcomes.

## 2. Problem statement

Businesses receive repetitive pre-sales questions over WhatsApp and often respond slowly or inconsistently. Customer information can be scattered across PDFs, FAQs, product catalogues, web pages and spreadsheets. Manual handling consumes staff time and causes missed leads outside business hours.

The platform should reduce repetitive work while preserving human control and avoiding ungrounded sales claims.

## 3. Goals and non-goals

### Goals
- Connect and operate a business's WhatsApp channel through an approved Meta integration.
- Provide onboarding for business profile, assistant instructions, hours, products/services and knowledge sources.
- Answer supported questions with tenant-specific, traceable knowledge.
- Capture and summarize leads, conversation intent and next steps.
- Support human takeover, pause/resume and escalation policies.
- Provide tenant isolation, auditability, usage visibility and operational reliability.
- Make AI costs measurable and configurable per tenant.

### Non-goals for the initial MVP
- Replacing a CRM, ERP, helpdesk or full e-commerce platform.
- Supporting every messaging channel at launch.
- Fully autonomous negotiation, payment collection or order fulfilment.
- Making unsupported guarantees about availability, delivery, pricing or policy.
- Training a custom foundation model.
- Complex visual flow building before the core conversation path is proven.
- Treating generated answers as truth or allowing the model to change business configuration.

## 4. Target users

- **Business owner / tenant admin:** connects WhatsApp, configures the assistant, loads product information, sets limits, inspects leads and understands performance.
- **Sales representative:** uses a shared inbox, reads summaries, sees lead details and takes over conversations.
- **Platform operator:** supports tenants, inspects failures, manages limits and investigates incidents.
- **End customer:** expects quick, relevant answers and a seamless route to a person.

## 5. Core user journeys

### 5.1 Tenant setup
1. Create a workspace and verify admin identity.
2. Enter business profile, locale, time zone, hours and sales policies.
3. Connect a WhatsApp Business account through the approved Meta flow, or use a controlled developer/test setup.
4. Configure assistant tone, permitted actions, escalation rules and response policies.
5. Add product/service catalogue and approved knowledge sources.
6. Run a test conversation and inspect the selected source evidence.
7. Enable the assistant after setup checks pass.

### 5.2 Customer conversation
1. Meta sends an inbound webhook.
2. Gateway validates the request and verifies tenant/channel association.
3. Gateway stores or durably enqueues a normalized event and returns a timely acknowledgement.
4. Worker deduplicates the event and appends it to the correct conversation.
5. Orchestrator checks conversation status, policy, rate limits and escalation state.
6. For an AI response, retrieve tenant-scoped relevant knowledge/catalogue records.
7. Generate a response under platform policy; validate output and channel constraints.
8. Record decision metadata, usage, latency and outcome; send through the WhatsApp adapter.
9. Retry transient failures safely and surface exhausted failures for operator review.
10. Escalate to a human when required.

### 5.3 Human handoff
A customer can ask for a person; the assistant can also escalate based on low retrieval confidence, unsupported requests, sensitive topics, repeated misunderstanding, policy triggers or tenant configuration. Once a human takes over, automated responses remain paused until an authorized user resumes them or a defined policy permits a transition.

### 5.4 Knowledge update
1. Validate source type, size, ownership and permissions.
2. Extract text and structure; retain source/version references.
3. Chunk and index with tenant and source identifiers.
4. Mark the new version ready only after indexing succeeds.
5. Replace or retire prior chunks safely; support retries and re-indexing.
6. Report failures and source freshness in the dashboard.

## 6. MVP scope

### Must have (P0)
- Tenant/workspace creation and authentication.
- Server-side role-based authorization.
- WhatsApp webhook verification, inbound handling and outbound text messaging through an approved integration.
- Durable event/message processing with deduplication and bounded retries.
- Conversation persistence and a basic shared inbox.
- Business profile, assistant policy and business-hours settings.
- Tenant-scoped knowledge ingestion and retrieval over a defined set of supported file types and URLs.
- Grounded response generation with safe fallback when evidence is insufficient.
- Human handoff and bot pause/resume.
- Basic lead capture: intent, need, summary, status and follow-up owner, plus contact details where volunteered/needed.
- Operator visibility into failed jobs and message status.
- Usage accounting for messages, model tokens/cost estimates, errors and latency.
- Audit events for security-sensitive actions.
- Deployment instructions, migrations, backup/restore notes and runbook.

### Should have (P1)
- Structured product/service catalogue.
- Lead filters, assignment and export.
- Basic analytics by tenant and period.
- Tenant-configurable working hours and escalation SLA.
- Knowledge source health, versioning and re-indexing.
- Supported message/template types in accordance with current Meta rules.
- Tenant quotas, alerts and configurable hard caps.

### Could have later (P2)
- Self-service Meta Embedded Signup if not available in the MVP integration.
- Subscription billing, invoices and payment-provider integration.
- CRM and webhook integrations.
- Multi-channel support.
- Visual flow builder, controlled tool calls, advanced campaigns, multilingual evaluation.
- Advanced attribution and experimentation.

## 7. Functional requirements

### FR-1 Identity and tenancy
- Users belong to one or more tenants with explicit membership.
- Roles at minimum: Owner/Admin, Sales Agent, Read-only Analyst, Platform Operator.
- Every tenant-scoped operation performs server-side authorization.
- Tenant IDs supplied by a client are never trusted without verifying membership and resource ownership.
- Administrative cross-tenant access is exceptional, audited and least-privileged.

**Acceptance:** Automated tests demonstrate that one tenant cannot read or mutate another tenant's conversations, leads, sources, settings or usage records.

### FR-2 WhatsApp integration
- Support webhook challenge verification and request signature validation according to Meta requirements.
- Normalize provider payloads into an internal event model.
- Handle provider retries and duplicate webhooks idempotently.
- Acknowledge valid webhook delivery quickly after durable acceptance; do not wait for model generation.
- Track inbound, queued, processing, sent, delivered/read where available, failed and handoff states.
- Respect channel messaging windows, template requirements, rate limits and API-version behavior as verified during implementation.
- Keep credentials in a managed secret store or environment-injection path, never source control.
- Isolate provider-specific code behind an adapter.

**Acceptance:** Replayed webhooks do not create duplicate customer messages or duplicate outbound replies; transient provider errors retry safely.

### FR-3 Conversation lifecycle
- Maintain a stable conversation identity by tenant and channel/customer identity.
- Preserve message direction, provider IDs, timestamps, status, safe metadata and source references.
- Serialize or safely coordinate state transitions for a conversation.
- Support open, bot-active, human-owned, paused, resolved and archived states (final enum may be refined).
- Record who changed ownership/status and when.
- Support search and pagination without cross-tenant exposure.

**Acceptance:** Concurrent inbound events do not corrupt state; human takeover prevents bot responses until resumed.

### FR-4 AI orchestration and policy
- Build responses from tenant configuration, channel rules, conversation context and approved evidence.
- Separate platform policy from tenant-editable prompt text.
- Treat retrieved documents and user content as untrusted data, not instructions.
- Never reveal prompts, secrets, internal tenant data or another customer's information.
- Do not invent stock, price, guarantees, discounts or policies.
- When evidence is missing, ambiguous or stale, ask a clarifying question, state the limitation, or escalate.
- Limit history/context, output tokens, tool calls and execution time.
- Use structured output where useful: response, evidence IDs, intent, lead fields and handoff recommendation.
- Use allowlisted tools with validated arguments; no unrestricted shell, browser or arbitrary network access in the MVP.
- Apply tenant-configured response and escalation policies.

**Acceptance:** Evaluations cover unsupported price/availability questions, prompt injection in uploaded documents, malicious customer messages, empty retrieval, stale documents and handoff requests.

### FR-5 Knowledge management / RAG
- Support a defined initial set of formats such as PDF, TXT, Markdown and CSV; confirm extraction quality and limits before launch.
- Track tenant, source ID/version, title, type, ingestion time, checksum and access state.
- Chunk documents using an appropriate, testable strategy.
- Tie each retrieval result to source version and tenant.
- Filter tenant/source scope at retrieval time, not merely after generation.
- Deduplicate by checksum where safe.
- Provide source states: pending, processing, ready, failed and retired.
- Define deletion and re-indexing behavior that removes or invalidates stale chunks.
- Keep evidence IDs internally and show useful attribution in the dashboard.
- Track retrieval quality and test tenant-specific questions.
- Limit file size, page count, extraction time and index growth.

**Acceptance:** Cross-tenant retrieval tests pass; deleting a source makes it unavailable to new responses; insufficient evidence produces a fallback rather than fabricated claims.

### FR-6 Lead management
- Capture lead fields only when available from the conversation or approved sources.
- Store intent, qualification answers, summary, interest, status, owner and follow-up date if applicable.
- Let authorized sales users edit, assign and change lead status.
- Link leads to their originating conversation and tenant.
- Provide export with access controls and auditability.

### FR-7 Dashboard
- Workspace setup and configuration.
- Conversation list/thread with state and assignment.
- Human takeover and bot resume controls.
- Knowledge upload, processing status and removal.
- Basic lead list/detail.
- Usage and error summary.
- Clear distinction between sandbox/test and live channel connections.
- Pagination, loading/empty/error states and confirmation for destructive actions.

### FR-8 Usage limits and commercial controls
- Track per-tenant message volume, model provider/model, input/output tokens where available, estimated model cost, retries and storage/index usage.
- Configure warning thresholds, rate limits and hard caps.
- On reaching a cap, pause automation, use fallback or notify an operator according to policy; do not silently incur unlimited cost.
- Make unit economics observable by tenant and feature.
- Add billing after metering and packaging are validated.

### FR-9 Audit and support
- Record tenant configuration changes, role changes, integration changes, exports/deletions, handoff/resume and platform operator access.
- Redact access tokens, signing secrets and sensitive customer data from logs.
- Provide correlation IDs across webhook, queue, worker, model request and send result.
- Document investigations for delivery, queue backlog, provider outage, failed ingestion and incorrect answers.

## 8. Non-functional requirements

These are initial engineering goals, not guarantees; validate them using representative load tests.

- **Reliability:** webhook acceptance depends on durable persistence/queueing, not model availability.
- **Latency:** measure end-to-end response and each stage separately; define p50/p95 targets after baselining.
- **Scalability:** scale APIs and workers independently, with bounded concurrency and backpressure.
- **Consistency:** idempotency and safe per-conversation state transitions.
- **Security:** least privilege, encryption in transit, managed secrets, tenant isolation and audited privileged operations.
- **Privacy:** documented retention, deletion/export, minimal PII collection and controlled access to content.
- **Operability:** health checks, metrics, traces, logs, alerting, backups and tested restore procedures.
- **Maintainability:** versioned event/API contracts, migrations, contract tests and clear service ownership.
- **Accessibility/localization:** usable dashboard, tenant locale/time zone and future support for local languages.
- **Portability:** avoid needless coupling to a model or vector vendor until evidence supports it.

## 9. Proposed architecture

This is an initial logical design. Validate service boundaries and technology choices before splitting the system into many independently deployed services.

```text
Customer (WhatsApp)
        |
        v
Meta WhatsApp Cloud API
        |
        v
Webhook Gateway ---> signature / tenant-channel validation
        |
        v
Durable event queue / stream
        |
        v
Conversation Worker / Orchestrator <----> Relational Database
        |                                      |
        |                                      +-- tenants, users, settings
        |                                      +-- conversations, messages
        |                                      +-- leads, jobs, audit, usage
        |
        +--> Tenant policy / business rules
        |
        +--> Retrieval service <------------ Knowledge ingestion workers
        |           |                              |
        |           +--> vector + keyword search  +--> object storage
        |           +--> tenant/source filters   +--> relational metadata
        |
        +--> LLM provider adapter
        |
        +--> outbound WhatsApp adapter
                    |
                    v
             Meta WhatsApp API

Dashboard --> Tenant API --> authentication/authorization + domain services
Observability --> logs, metrics, traces, alerts and audit tooling
```

### Logical component responsibilities
- **Dashboard:** UI; all authorization remains server-side.
- **Tenant API:** identity/membership, tenant config, integrations, source/lead/conversation APIs.
- **Webhook gateway:** provider verification, signature checks, normalized events, idempotency and fast acknowledgement.
- **Conversation engine:** state serialization, routing, prompt assembly, policy, handoff and outbound orchestration.
- **Ingestion workers:** extraction, normalization, chunking, index updates, version handling and retries.
- **Relational database:** source of truth for business state.
- **Queue/stream:** asynchronous decoupling, backpressure, retry scheduling and dead-letter visibility.
- **Object storage:** uploaded sources and controlled ingestion artifacts.
- **Retrieval:** vector plus keyword search and metadata filtering; evaluate relevance and operations before committing to a specific engine.
- **Provider adapters:** WhatsApp and LLM providers behind explicit interfaces.
- **Observability:** central signals with tenant-aware access controls and minimal sensitive content.

### Technology-selection guidance
A reasonable initial stack could use a React dashboard, a backend language the team operates well, PostgreSQL as transactional source of truth, a durable queue/stream, object storage and a vector/full-text retrieval layer. If reusing an existing repository, retain its choices until an audit identifies a concrete correctness, security, operational or cost problem.

Compare retrieval relevance, tenant filtering, migrations, backups, query performance, operational complexity and total cost before selecting a vector engine. Prefer the fewest deployable services that preserve required reliability boundaries.

## 10. Logical data model

- `Tenant(id, name, status, timezone, locale, created_at)`
- `User(id, identity_provider_id, status)`
- `Membership(tenant_id, user_id, role, created_at)`
- `ChannelConnection(id, tenant_id, provider, external_account_id, status, secret_reference, created_at)`
- `TenantSettings(tenant_id, assistant_policy, business_hours, escalation_policy, limits, version)`
- `Conversation(id, tenant_id, channel_connection_id, external_customer_ref, status, owner_user_id, last_message_at)`
- `Message(id, tenant_id, conversation_id, provider_message_id, direction, message_type, body_or_object_ref, status, created_at, metadata)`
- `WebhookEvent(id, tenant_id, provider_event_id, payload_hash, received_at, processing_status)`
- `KnowledgeSource(id, tenant_id, source_type, title, object_ref, checksum, version, status, created_at)`
- `KnowledgeChunk(id, tenant_id, source_id, source_version, chunk_index, text_or_object_ref, metadata, embedding_ref)`
- `Lead(id, tenant_id, conversation_id, status, intent, summary, qualification_data, assigned_to, created_at, updated_at)`
- `Job(id, tenant_id, job_type, idempotency_key, status, attempts, scheduled_at, completed_at, error_code)`
- `UsageEvent(id, tenant_id, conversation_id, provider, model, input_tokens, output_tokens, estimated_cost, created_at)`
- `AuditEvent(id, tenant_id, actor_id, action, resource_type, resource_id, metadata_redacted, created_at)`

Finalize normalization, indexes, retention and migrations during detailed design. Do not store raw provider secrets in ordinary application tables; use a managed secret store or controlled encryption scheme with access separation.

## 11. API and event contract direction

Illustrative normalized event envelope; finalize with schema validation and versioning:

```json
{
  "event_id": "provider-or-internal-unique-id",
  "event_type": "whatsapp.message.received",
  "schema_version": 1,
  "tenant_id": "tenant-id",
  "channel_connection_id": "connection-id",
  "occurred_at": "ISO-8601 timestamp",
  "correlation_id": "request-trace-id",
  "data": {
    "provider_message_id": "provider-id",
    "conversation_key": "opaque-customer-channel-key",
    "message_type": "text"
  }
}
```

Requirements:
- Validate every event against a versioned schema.
- Make IDs/idempotency keys unique in the correct scope.
- Avoid placing large files or unnecessary PII in queue messages.
- Design consumers for at-least-once delivery.
- Define retryable and terminal error categories.
- Do not claim exactly-once end-to-end delivery; use idempotency and reconciliation to approximate desired user-visible behavior.

## 12. Security, privacy and abuse prevention

### Security controls
- Use a maintained identity mechanism and a secure session/CSRF strategy suitable for the frontend.
- Apply tenant scoping to every API, search, export, background job and support tool.
- Verify webhook signatures and validate payload sizes/content.
- Rate-limit by tenant, customer/conversation and IP where appropriate.
- Store provider/LLM credentials in a secret manager with rotation procedures.
- Encrypt in transit and use managed encryption at rest.
- Validate files, apply archive/decompression limits and parser timeouts, and scan for malware where appropriate.
- Add dependency and secret scanning, reproducible builds where feasible and patch management.
- Redact credentials and sensitive content in logs/traces.
- Audit privilege changes, connection changes, exports and deletions.
- Back up data with documented recovery objectives and restricted access.

### AI-specific controls
- Treat customer messages and retrieved documents as untrusted.
- Separate business knowledge from executable policy and platform instructions.
- Prevent cross-tenant retrieval using defense-in-depth in authorization, query construction, metadata filters and tests.
- Use allowlisted tools and schema validation; avoid unconstrained side-effecting actions.
- Require confirmation or human action before consequential external operations.
- Define fallback/refusal paths for prohibited, unknown or high-risk requests.
- Run prompt-injection and data-exfiltration evaluations.
- Do not train on tenant/customer data by default; define explicit consent and retention if this changes.

### Privacy and compliance planning
Before commercial launch, assess applicable privacy, consumer-protection, messaging and data-transfer obligations for target markets. Provide terms, privacy notice, retention/deletion behavior, data-processing arrangements, incident response and a support contact. Do not claim certification or legal compliance without a documented assessment.

## 13. Reliability and operations

- Use durable acceptance before acknowledging webhook payloads where provider retry semantics permit.
- Make workers idempotent and bound retries with exponential backoff and jitter.
- Quarantine poison messages; expose redacted failure context and replay controls.
- Serialize one conversation's state transitions or protect them with compare-and-set semantics.
- Apply timeouts, concurrency limits, circuit breakers and bounded queues to provider/model calls.
- Respect provider limits and apply tenant-aware fairness.
- Track send attempts and reconcile uncertain outcomes before retrying potentially duplicate sends.
- Monitor queue depth/age, processing failures, send failures, webhook validation failures, model errors, latency, retrieval empty-rate and cost.
- Maintain runbooks for provider outages, expired credentials, queue backlogs, database issues, model releases, ingestion failures and restore.
- Test restore procedures; an untested backup is not a verified recovery plan.
- Use safe test tenants/numbers and explicit guards around live-send actions.

## 14. Observability and success metrics

### Reliability and quality
- Webhook acknowledgement latency and rejection count.
- Duplicate-event count.
- Queue depth and oldest-event age.
- Time from inbound message to send attempt and confirmation where available.
- Outbound failure/retry rate by error category.
- Worker crash/restart count.
- Ingestion success/failure rate.
- Retrieval empty rate and relevance on labelled evaluations.
- Grounding and policy adherence on human-reviewed tests.
- Handoff rate, time to human takeover and repeated misunderstanding rate.
- Per-tenant model usage and estimated cost per conversation.

### Business outcomes
- Leads captured and qualified.
- Qualified lead to human follow-up rate.
- Response time outside business hours.
- Conversion attribution only where reliable data exists.
- Tenant activation and retention.
- Operator time saved against a defensible baseline.

Do not optimize for deflection alone; avoiding human handoff while giving poor answers is not success.

## 15. Testing strategy

- **Unit:** policy decisions, tenant authorization, event normalization, idempotency, state transitions, retry classification and cost calculations.
- **Integration:** relational DB, queue/stream, object store, retrieval store, WhatsApp adapter fixtures and model provider adapters.
- **Contract:** webhook versions, internal event schemas, outbound requests and API shapes.
- **Security:** tenant-boundary tests, IDOR attempts, signature rejection, upload abuse, prompt injection and secret/log redaction.
- **Reliability:** duplicate/out-of-order events, worker crash, provider timeout after send, dead-letter replay, queue saturation and restore.
- **AI evaluation:** FAQs/product questions, unsupported facts, ambiguity, stale source, empty retrieval, multilingual messages, escalation and prompt injection.
- **End-to-end:** tenant setup in sandbox, message receipt, grounded answer, lead capture, handoff and resume.
- **Load:** representative concurrent tenants/conversations, webhook bursts, indexing bursts and provider throttling.

Maintain a versioned golden evaluation dataset. Material prompt, retrieval, model or orchestration changes should run against it before release.

## 16. Delivery milestones

### Milestone 0 — repository and architecture audit
- Inspect services, migrations, deployment manifests, environment examples, CI, tests, security configuration and runbooks.
- Map the current features to this PRD as implemented, partial, missing, unsafe or unverified.
- Verify the licence and dependency licences before reuse.
- Run locally and record reproducible results.
- Identify secrets, insecure defaults, unauthenticated endpoints and missing tenant enforcement.
- Record architecture decisions and prioritized gaps.

**Exit criteria:** reproducible startup guide, current architecture diagram, feature matrix, licence review, security findings and prioritized implementation plan.

### Milestone 1 — safe vertical slice
- Provision a tenant, configure a test channel, receive and normalize a webhook, durably process a message, return a deterministic response and persist history.
- Add idempotency, delivery state and tracing before complex AI behavior.

**Exit criteria:** duplicate events do not duplicate visible messages; failures can be retried/reconciled; tenant-boundary tests pass.

### Milestone 2 — knowledge-grounded assistant
- Implement ingestion/retrieval, answer generation, evidence metadata, fallback rules and golden-set evaluation.
- Begin with bounded source types and measurable retrieval quality.

**Exit criteria:** thresholds are defined, source deletion works, insufficient evidence falls back safely and prompt-injection tests pass.

### Milestone 3 — sales operations
- Add lead capture, inbox, assignment, takeover/pause/resume, business hours and escalation audit trail.

**Exit criteria:** a salesperson takes over without bot interference, inspects a summary and follows up on a lead.

### Milestone 4 — hardening and pilot
- Add quotas, usage metering, dashboards, alerts, backup/restore test, rate limits, runbooks and a controlled pilot.
- Validate latency, cost, reliability and answer quality with consented pilot use.

**Exit criteria:** agreed pilot SLOs and metrics are met; support procedures exist; deletion/retention is tested; launch blockers are documented.

### Milestone 5 — commercial readiness
- Add self-service onboarding and billing after core activation and unit economics are proven.
- Complete legal/privacy/security review for intended markets.
- Add support workflows, plan entitlements and fair-use enforcement.

**Exit criteria:** onboarding recovery, billing edge cases, data portability/deletion, operational ownership and support are validated.

## 17. Risks and mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| Cross-tenant data exposure | Critical security incident | Server-side scoping, row-level controls where useful, negative tests and audit |
| Hallucinated sales claims | Lost trust or customer harm | Grounding, evidence thresholds, fallback, evaluations and human escalation |
| Duplicate messages after retries | Confusing experience/cost | Idempotency and send-state reconciliation |
| WhatsApp policy/API changes | Failed messaging/onboarding | Provider adapter, version monitoring and conformance tests |
| Model/provider outage | Delayed responses | Durable queue, bounded retries, fallback and operator visibility |
| Runaway AI costs | Poor margins | Per-tenant metering, caps, budgets and token/context limits |
| Poor extraction | Inaccurate answers | Previews, ingestion diagnostics and labelled evaluation |
| Premature microservices | Operational complexity | Start with clear minimal boundaries; split when required |
| Overreliance on candidate repository | Hidden defects/licence risk | Audit code, dependencies, security, tests, licence and maturity |

## 18. Open decisions

1. Modular monolith versus separately deployed gateway/API/worker/ingestion services.
2. Redis Streams versus managed queue or another broker based on operations and delivery semantics.
3. PostgreSQL/pgvector plus full text versus dedicated vector store; benchmark real documents.
4. Reuse/repair the candidate dashboard versus rebuilding a scaffold.
5. Developer-managed Meta connection for pilot versus Embedded Signup for self-service.
6. Single default model provider with adapter interface versus multi-provider routing.
7. Retention and deletion policy by target market/customer needs.
8. Commercial packaging after unit economics validation.
9. Narrow initial customer segment, chosen after interviews rather than hard-coded prematurely.

Document each decision with context, options, trade-offs and revisit triggers.

## 19. Definition of done for MVP

The MVP is ready for a controlled pilot only when:
- A tenant can be onboarded to a permitted test/live WhatsApp setup through documented steps.
- Inbound messages are persisted or durably queued before acknowledgement where applicable.
- Duplicate webhooks are handled safely.
- Tenant isolation is proven by negative tests across APIs, inbox, leads, retrieval, jobs and exports.
- AI answers use approved knowledge and have a measurable fallback for insufficient evidence.
- Human takeover and bot pause/resume work reliably.
- Leads link to conversations and are only visible to authorized roles.
- Per-tenant usage and estimated costs are visible with configurable limits.
- Secrets and sensitive fields are excluded from logs and source control.
- CI, migration, deployment, backup and restore instructions exist.
- Alerts and runbooks cover high-impact failures.
- Privacy, retention/deletion, support ownership and target-market obligations are reviewed before real customer launch.
- Pilot metrics and a quality/incident review process are defined.

## 20. Product success criteria

A tenant can activate with minimal assistance, the assistant gives accurate grounded answers, qualified leads are reliably captured, human handoff is effortless, delivery failures are recoverable and per-tenant economics are measurable. Message volume or low handoff rate alone is not evidence of success.
