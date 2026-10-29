# CAISP
The Certified AI Security Practitioner (CAISP) is an advanced, hands-on certification designed for cybersecurity professionals, penetration testers, and DevSecOps engineers who want to master the art and science of securing artificial intelligence systems.

As enterprises rapidly adopt Large Language Models (LLMs), Retrieval-Augmented Generation (RAG) pipelines, and agentic workflows, traditional web and network security controls are no longer enough. CAISP bridges the gap between theoretical AI safety and real-world offensive security, training practitioners to audit, exploit, and harden modern AI architectures.

## Core Learning & Practical Domains

Candidates enrolled in the CAISP program gain practical expertise across critical domains aligned with the OWASP Top 10 for Large Language Models and Enterprise APIs:

LLM Prompt & Guardrail Evasion: Bypassing static regex Data Loss Prevention (DLP) filters and defeating rigid system prompt instructions.

Vector Store & RAG Security: Exploiting client-controlled clearance scope parameters, performing vector database oversharing enumeration, and executing indirect prompt injections via document ingestion.

Output Handling & Client-Side Execution: Defeating naive inbound Web Application Filters (WAFs) and engineering AI-driven Cross-Site Scripting (XSS) payloads.

Agentic Frameworks & Excessive Agency: Auditing restricted backend tool calls, extracting shadow model supply-chain manifests, and chaining vulnerabilities.

API Pivoting & Infrastructure Exploitation: Conducting Broken Authentication attacks against internal diagnostic daemons, exploiting Server-Side Request Forgery (SSRF) to access isolated loopback services, and executing agentic OS command injections.


# NexusAI Enterprise Security Lab

A containerized, black-box security research environment built around a production-style **Retrieval-Augmented Generation (RAG) and agentic AI assistant**. The lab simulates a hardened enterprise deployment — complete with an inbound Web Application Firewall (WAF), role-scoped vector retrieval, and an outbound Data Loss Prevention (DLP) engine — so practitioners can audit, exploit, and understand the real attack surface of modern LLM systems.

The environment ships as a guided, self-contained exercise with **10 progressive assessment objectives** spanning prompt security, vector store weaknesses, output handling, agentic tool abuse, and infrastructure pivoting. Objectives are mapped to the **OWASP Top 10 for Large Language Models** and enterprise API security categories.

---

## Overview

**NexusCorp** has deployed **NexusAI (v4.2.0-rc3)**, an enterprise RAG and agentic diagnostic assistant that:

- Answers employee policy inquiries through a chat interface
- Indexes uploaded compliance documents into a corporate knowledge base
- Exposes internal diagnostic tools for system administrators

Your role is that of an **AI security assessor** performing a full black-box assessment against the architecture — from the chat layer down to isolated internal services — and working through the full chain of vulnerabilities the deployment presents.

The lab is deliberately adversarial: every control that looks solid from the outside (WAF, DLP, clearance scoping) contains a realistic misconfiguration or design flaw commonly observed in real-world LLM deployments.

---

## Lab Architecture

The environment runs across three primary containers, plus an isolated loopback service inside the UI gateway:

| Component | Endpoint | Description |
|---|---|---|
| **NexusAI Portal & Agent Gateway** | `http://localhost:5000` | Flask web portal handling chat sessions, document ingestion, inbound WAF checks, outbound DLP filtering, and backend agent tool execution. |
| **Local LLM Inference Engine** | internal `:11434` (Docker bridge) | Ollama container serving the `qwen2.5:1.5b` chat completion model. |
| **Corporate Vector Store** | internal `:8000` (Docker bridge) | ChromaDB instance storing corporate policies, executive memos, DevOps notes, and user-uploaded `.txt` files with clearance metadata. |
| **Internal Loopback KMS Vault** | `127.0.0.1:5678` (loopback only) | Local Key Management Service bound strictly to the loopback interface inside the gateway container. Unreachable from the host network — *or so the deployment claims.* |

### Entry Points

| Route | Methods | Description |
|---|---|---|
| `http://localhost:5000/` | `GET` | Main NexusAI Enterprise Assistant web interface |
| `http://localhost:5000/chat` | `POST` | Submits prompts and clearance scope parameters to the RAG + LLM pipeline |
| `http://localhost:5000/upload` | `POST` | Indexes uploaded `.txt` policy or resume files into the employee vector collection |
| `http://localhost:5000/api/v1/audit/status` | `GET, POST` | Administrative audit daemon and internal service healthcheck endpoint |
| `http://localhost:5000/api/v1/...` | `GET` | Additional undocumented internal endpoints — discovery is part of the exercise |

---

## Quick Start

**Prerequisites:** Docker Engine or Docker Desktop with **≥ 4 GB RAM** allocated.

**1. Start the container stack**

```bash
docker-compose up -d
```

**2. Download the local language model**

```bash
docker exec -it chimera-ollama ollama pull qwen2.5:1.5b
```

**3. Verify readiness**

Open [http://localhost:5000](http://localhost:5000) in your browser and ask a baseline question, e.g.:

> *"What are the standard operating hours?"*

Once NexusAI replies, the environment is live and you can begin.

---

## Assessment Objectives

The lab contains **10 verification objectives**, grouped into three phases. Each objective maps to a concrete weakness class in the OWASP LLM Top 10 and enterprise API Top 10.

### Phase I — Prompt Guardrails & Vector Knowledge Base

| # | Objective | Mapped Weakness |
|---|---|---|
| 01 | System prompt leakage & outbound DLP evasion | LLM07: System Prompt Leakage |
| 02 | RAG metadata access control bypass | LLM08: Vector & Embedding Weaknesses |
| 03 | Vector store oversharing (DevOps escrow) | LLM08 / API2: Privileged Data Exposure |
| 04 | Indirect prompt injection via document ingestion | LLM01: Indirect Prompt Injection |

### Phase II — Output Handling, Excessive Agency & Supply Chain

| # | Objective | Mapped Weakness |
|---|---|---|
| 05 | Insecure output handling & WAF bypass (XSS) | LLM05: Improper Output Handling |
| 06 | Excessive agency (restricted syslog tool) | LLM06: Excessive Agency |
| 07 | Shadow model supply-chain manifest exposure | LLM03 / API9: Improper Inventory Management |

### Phase III — Shadow API Pivoting, SSRF & Agentic RCE

| # | Objective | Mapped Weakness |
|---|---|---|
| 08 | Broken administrative audit authentication | API2: Broken Authentication |
| 09 | Internal KMS vault loopback SSRF | API7: Server-Side Request Forgery |
| 10 | Agentic OS command injection & filter bypass | LLM06 / OS Command Injection |

Each captured token follows the format `FLAG{...}`. If you extract an inner value through an encoding or formatting bypass, wrap it back in `FLAG{...}`.

---

## Rules of Engagement

1. **Black-box only.** All testing must be performed against `http://localhost:5000` using a browser, DevTools, `curl`, or an intercepting proxy (Burp Suite, OWASP ZAP, etc.).
2. **No container tampering.** Do not use `docker exec` or filesystem inspection on the UI container to shortcut the environment. Application logic is compiled and obfuscated; every objective is solvable through application and AI security exploitation alone.
3. **Flag format.** All tokens use the format `FLAG{...}`. Re-wrap any extracted inner value before submission.

---

## Tooling Suggestions

- **Browser DevTools** — inspect form elements, clearance scope parameters, and rendered DOM behavior
- **Intercepting proxy** — Burp Suite or OWASP ZAP for modifying chat requests, headers, and audit daemon payloads
- **curl** — for direct probing of internal API routes once discovered

---

## Tech Stack

- **Application:** Python / Flask (NexusAI Portal & Agent Gateway)
- **LLM Runtime:** Ollama (`qwen2.5:1.5b`) — fully local, no external API calls
- **Vector Store:** ChromaDB
- **Containerization:** Docker Compose

---

## Educational Disclaimer

This environment is designed exclusively for **authorized security training and research**. Every vulnerability it contains is intentional and sandboxed. Do not apply the techniques demonstrated here against systems you do not own or have explicit permission to test.

---

*Built for hands-on AI security education. Learn by doing.*
