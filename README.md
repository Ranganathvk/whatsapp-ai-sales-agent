# WhatsApp AI Sales Agent SaaS

A multi-tenant SaaS platform that helps businesses answer WhatsApp customer inquiries, qualify leads, recommend products or services from approved business knowledge, and hand conversations to a human when needed.

> **Status:** Product specification / planning. This repository describes the intended product; it does not claim a production-ready implementation.

## Product direction

Build a dependable WhatsApp-first AI sales assistant for small and medium businesses. Businesses should be able to connect their WhatsApp Business account, configure their assistant, add approved knowledge and product information, test conversations, and review leads and outcomes from a dashboard.

Prioritize reliable messaging, tenant isolation, clear human handoff, observable AI behavior, and predictable operating cost over unconstrained autonomous behavior.

## Start here

- [Product Requirements Document](docs/PRD.md) — goals, user journeys, functional requirements, architecture, security, operations, and delivery milestones.
- [Candidate implementation foundation](https://github.com/Javier407/whatsapp-ai-saas) — an open-source repository identified for evaluation. It is a candidate to audit and extend, not assumed to be production-ready or part of this repository.

## Initial architecture direction

- **Web dashboard:** tenant setup, knowledge and catalog management, conversation inbox, lead review, settings, analytics.
- **Tenant API:** authentication, tenant/workspace configuration, authorization, business data.
- **WhatsApp gateway:** Meta webhook verification, inbound event validation, message normalization, outbound message adapter.
- **Conversation/flow engine:** durable state transitions, routing, response orchestration, fallback and human handoff.
- **Knowledge retrieval:** ingestion pipeline and retrieval over tenant-scoped documents and catalog data.
- **Async infrastructure:** durable queue/stream for processing, retries, dead-letter handling, and webhook decoupling.
- **Persistence:** relational store for tenants, configuration, messages, leads and audit events; object storage for uploaded source files.
- **Observability:** structured logs, metrics, traces, cost/latency tracking, alerts and operator runbooks.

Implementation choices should be validated with proof of concept and explicit trade-off records before becoming hard constraints.

## Product principles

1. **Tenant isolation by design:** every tenant-owned resource and retrieval must be scoped and authorized server-side.
2. **Grounded responses:** sales claims, product facts, pricing, availability and policies should come from approved tenant data or a clearly defined trusted integration.
3. **Human control:** provide clear escalation, pause, and takeover paths.
4. **Durability:** acknowledge webhooks safely, deduplicate retries, preserve message ordering where needed, and make failures recoverable.
5. **Transparent limits:** disclose AI uncertainty, channel limitations, usage limits, and costs to tenant operators.
6. **Privacy and security:** minimize collected data, protect secrets, redact sensitive logs, and define retention/deletion workflows.
7. **Measure outcomes:** monitor response quality and latency, lead qualification, handoff rate, conversion attribution, and per-tenant costs.

## Planning status

See the PRD for the MVP boundary and acceptance criteria. Meta onboarding, billing, production compliance, and implementation of every PRD feature are not represented as complete.
