# JourneyHelp
### A context-aware, voice-first help layer for digital products

JourneyHelp is an open-source portfolio project exploring how any product can offer contextual help through **voice or text**, grounded in approved product knowledge.

The flagship demo is an automotive experience: a driver asks about a dashboard indicator or vehicle feature, and JourneyHelp uses the vehicle context plus the owner's manual to provide a concise, source-grounded explanation. The same answer engine should also work for banking and B2B SaaS demonstrations.

> **Project status:** Design and starter-repository stage. This README records the intended product and technical decisions; it does not claim that the proposed APIs or integrations are already implemented.

---

## 1. Product vision

**Make complex products easier to understand at the moment a person needs help.**

Most help centers make users search documentation, leave their current workflow, or translate product terminology into a support ticket. JourneyHelp aims to put a small, embeddable assistant inside the product journey itself.

A host product supplies relevant context—such as the page being viewed, product model, feature, locale, or selected warning—and JourneyHelp combines that context with approved knowledge to answer the user's question.

### The core promise

**Any product can provide context and knowledge to the same help engine; users can interact through voice or text.**
JourneyHelp is not intended to be a generic chatbot that confidently answers everything. It should answer from evidence, ask a clarifying question when needed, and say when the available documentation is insufficient.



---
## 2. Executive Summary & Problem Statement

**Problem**
Most help centers force users to leave their current workflow, search through dense documentation, or translate their issue into a support ticket. This creates friction in complex digital journeys.

**Solution**
JourneyHelp is a context-aware, voice-first help layer that embeds directly into a product's user journey. By combining host-provided context (e.g., current page, selected feature, product model) with approved product knowledge via Retrieval-Augmented Generation (RAG), it provides concise, source-grounded explanations via voice or text.

---
## 3. Product Vision & Goals

**Vision**
Make complex products easier to understand at the exact moment a person needs help, without leaving their workflow.
Core Promise: Any product can provide context and knowledge to the same help engine; users can interact through voice or text.

**MVP Success Criterion**
A user can ask a contextual question in the automotive demo, receive a grounded answer with source evidence, choose their preferred response mode (voice/text), and see the exact same core answer behavior replicated in the B2B SaaS demo.


---
## 4. Target Audience
Product Teams: Seeking to reduce friction in complex digital journeys.
CX & Support Teams: Aiming to resolve routine "how-to" questions in context.
OEMs & Automotive Digital Teams: Needing to explain vehicle features and dashboard indicators safely.
B2B SaaS Vendors: Whose users struggle with complex configurations and workflows.
Banks & Financial Providers: Requiring clear explanations of approved product terms and benefits.


---
## 5. Scope Definition
**5.1 In-Scope (MVP)**
Embeddable Widget: Text input and push-to-talk voice input.
Response Modes: First-run choice between spoken or text answers; session-level preference memory.
Accessibility & Controls: Visible transcript for spoken answers; stop, replay, and switch-mode controls.
Shared Answer Engine: Single RAG pipeline for both voice and text inputs.
Knowledge & Evidence: Source citations, metadata filtering, and explicit "insufficient evidence" / clarification behaviors.
Context Injection: Host-provided context (product, model/year, page, selected feature, locale).
Demo Environments: Simulated automotive dashboard + owner's manual knowledge set; one small B2B SaaS demo.
Evaluation: Basic evaluation cases, structured logs, and local Dockerized development setup.

**5.2 Out-of-Scope (MVP)**
Always-on microphone access or wake-word detection.
Real vehicle telemetry, OBD/CAN integration, or vehicle control.
Autonomous actions affecting accounts, vehicles, or finances.
Access to private banking/account data.
Fine-tuning foundation models or training custom STT/TTS models.
Enterprise SSO, complex RBAC, billing, and multi-region deployment.
Multi-agent orchestration.

---







