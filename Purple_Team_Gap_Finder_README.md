# Purple Team Gap Finder

A self-hosted **Breach and Attack Simulation (BAS)** platform that lets security teams run MITRE ATT&CK-mapped attack campaigns, measure detection coverage against their SIEM, and generate AI-assisted gap analysis and remediation reports — without exposing the underlying Caldera/Splunk/Wazuh infrastructure to end users.

Conceptually comparable to commercial BAS platforms like **AttackIQ, SafeBreach, and Cymulate**, but open-source, self-hosted, and free of recurring licensing fees.

---

## Overview

A client/org admin signs up, creates a campaign (choosing OS target, threat categories, and execution mode), runs it, and receives a MITRE ATT&CK heatmap showing exactly which techniques their security stack detected — along with AI-generated remediation guidance and exportable PDF/CSV reports (Brief / Detailed / Executive).

---

## Key Features

- **Campaign Wizard** — step-by-step creation: name → assessment type (Agent-based / Agentless / Hybrid) → target OS (Windows/Linux/Both) → threat categories (MITRE tactics) → report level
- **Dual Execution Modes**
  - **Agent-based**: lightweight Go/Python agents deployed on target hosts, communicate over secure TLS, auto-update, heartbeat monitoring
  - **Agentless**: SSH (Linux) / WinRM (Windows) remote execution with encrypted credential storage — no software installation required
- **MITRE ATT&CK Heatmap** — interactive matrix (14 tactics × 700+ techniques), color-coded by detected (green) / partial (yellow) / gap (red) / not tested (white)
- **Gap Analysis Engine** — severity-scored (1–5) detection gaps with specific, technique-tailored remediation steps
- **SIEM Integration** — Wazuh (live) and Splunk detection pulling, correlates technique execution with actual alerts fired
- **AI Chatbot Assistant** — natural-language campaign creation, gap explanation, remediation guidance, and compliance mapping (NIST/CIS/ISO 27001), with multi-turn conversation memory scoped to a selected campaign
- **Reporting** — Brief / Detailed / Executive PDF reports + raw CSV exports, dynamically generated per campaign (not static/hardcoded)
- **Scheduled Campaigns** — cron-based recurring campaigns with email notifications on completion/failure
- **Multi-tenant Organization Management** — RBAC with Owner/Admin/Member/Viewer roles, invite-by-email
- **Subscription & Billing** — Stripe-integrated tiered plans (Free/Pro/Enterprise) with usage quota enforcement
- **Real-time Updates** — WebSocket-driven live campaign progress

---

## Tech Stack

| Layer | Technology |
|---|---|
| **Frontend** | React, TypeScript, Vite |
| **Backend** | Node.js, Express.js |
| **Database** | PostgreSQL (multi-tenant schema) |
| **Execution Engine** | MITRE Caldera (red team automation), Atomic Red Team |
| **Detection / SIEM** | Wazuh, Splunk |
| **Agents** | Go (cross-platform), Python (Linux/macOS), PowerShell (Windows) |
| **Agentless Remoting** | SSH (Linux targets), WinRM (Windows targets) |
| **Credential Storage** | HashiCorp Vault |
| **AI** | OpenAI / Anthropic / Ollama (provider-agnostic) |
| **Payments** | Stripe |
| **Real-time** | WebSocket |
| **Reporting** | PDFKit (PDF), CSV export |
| **Containerization** | Docker, Docker Compose |
| **Mixed runtime** | Python scripts invoked via `child_process.spawn()` for Caldera/Splunk/heatmap generation alongside the Node.js core |

---

## Architecture

```
User → NGINX → Backend API (Node.js) → Orchestrator Layer
                                              │
                       ┌──────────────────────┼──────────────────────┐
                       ▼                      ▼                      ▼
                  Caldera (private)     Splunk/Wazuh (private)   PostgreSQL
                  Agent-based exec       Detection pulling         Multi-tenant data
                       │                      │
                       ▼                      ▼
              Agentless (SSH/WinRM)   Gap Analysis Engine
                                              │
                                              ▼
                                   Heatmap + AI Remediation + Reports
```

**Design principle:** Users never interact with Caldera, Splunk, or Wazuh directly — the Orchestrator layer abstracts all of it behind a clean campaign API.

### Core Workflow
```
Signup/Login → Empty Dashboard (no campaigns yet)
     ↓
Create Campaign (OS + Threat Categories + Report Level + Execution Mode)
     ↓
Run Campaign → Caldera/Agentless executes techniques
     ↓
Pull Detections (Wazuh/Splunk) → Gap Analysis (severity scoring)
     ↓
Heatmap Generation → AI Remediation Suggestions
     ↓
Download Report (Brief/Detailed/Executive PDF/CSV)
```

---

## Database Schema (Key Tables)

| Table | Purpose |
|---|---|
| `organizations` / `organization_members` | Multi-tenancy root, RBAC membership |
| `users` | Auth, roles, OAuth identities |
| `campaigns` | Campaign config (OS, threat categories, report level, status) |
| `agents` / `agent_tasks` / `agent_heartbeats` | Agent-based execution tracking |
| `agentless_targets` / `agentless_executions` | SSH/WinRM target configs and run history |
| `executions` | Per-technique execution records |
| `detections` | SIEM-confirmed detection events linked to executions |
| `gaps` | Detected vs. undetected techniques with severity and remediation |
| `techniques` | Full MITRE ATT&CK knowledge base (700+ techniques) |
| `compliance_mappings` | Technique → NIST/CIS/ISO 27001 mapping |
| `subscription_billing` | Stripe customers, tiers, usage quotas |
| `scheduled_tasks` | Cron-based recurring campaign configs |
| `ai_conversations` / `ai_messages` | Chatbot context and history |
| `webhooks` | Event-driven third-party integration triggers |

---

## Project Status Snapshot

| Phase | Completion |
|---|---|
| Phase 1 — Core Platform (auth, RBAC, campaigns, billing) | ~100% |
| Phase 2 — Agent-based execution | ~90% (OS cross-testing pending) |
| Phase 2 — Agentless execution | ~85% (proxy/hybrid testing pending) |
| Phase 3 — AI Chatbot & advanced features | ~91% (safety testing pending) |
| Phase 4 — Cloud deployment, monitoring, launch | Not started |

### Known Critical Gaps (as of last audit)
- No test coverage (no Jest/Mocha/Vitest)
- Secrets present in `.env`, not gitignored
- No security headers (Helmet), unrestricted CORS
- No rate limiting on auth/AI endpoints
- No input validation library on backend routes
- No frontend auth route guards
- Mixed JS + Python services increase maintenance overhead

---

## Roadmap to Production

1. **Critical fixes** — executive report crash, campaign Run button ignoring filters, org name bug in reports
2. **High priority** — Google OAuth wiring, org-scoped campaign filtering, Stripe env/key fixes
3. **Medium priority** — user profile/password management, subscription enforcement middleware, empty-state UX for new orgs
4. **Low priority** — agent binary distribution, agentless fallback automation, Docker Compose production stack, post-deploy update scripting
5. **Hardening** — test suite, secrets management, Helmet + rate limiting + input validation, audit logging, refresh token rotation

---

## Deployment

```bash
# Docker Compose (Postgres + Backend + Frontend + Caldera + Splunk/Wazuh)
docker-compose up -d
```

Environment variables required: `DATABASE_URL`, `JWT_SECRET`, `CALDERA_API_KEY`, `OPENAI_API_KEY` (or equivalent), `STRIPE_SECRET_KEY`, SMTP credentials for notifications.

---

## Comparison to Commercial BAS Platforms (AttackIQ, etc.)

| | This Project | AttackIQ |
|---|---|---|
| Execution methods | Agent-based + Agentless | Agent-based + Agentless |
| MITRE ATT&CK coverage | Full | Full |
| SIEM verification | Trusts DB detection record | Queries SIEM API directly to confirm alert fired |
| Scenario library | Manually seeded | 1,000+ pre-built |
| Cost | Self-hosted, free | $50K–200K+/year enterprise licensing |
| Customization | Full source access | Limited |
| AI remediation guidance | Built-in (OpenAI/Claude) | Limited |

---

## Author

Built as a hands-on exploration of full-stack security tooling — covering red-team automation (Caldera/Atomic Red Team), SIEM integration (Wazuh/Splunk), multi-tenant SaaS architecture, AI-assisted security analysis, and BAS platform design patterns used by commercial tools like AttackIQ.
