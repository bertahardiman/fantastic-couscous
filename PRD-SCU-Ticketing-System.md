# PRD — Seller Care Unit (SCU) Ticketing System

| Field | Value |
|---|---|
| Created at | 14 Apr 2026 |
| Last updated | 14 Apr 2026 |
| Document Status | `DRAFT` |
| Initiative Status | `NOT STARTED` |
| Document Owner | [Alberta Hardiman](mailto:alberta.hardiman@astronauts.id) |
| Tech Owner | ❓ TBD |
| Design Owner | ❓ TBD |
| QA | ❓ TBD |
| Stakeholders | SCU Team, Category Managers, Seller & Ads Platform Engineering |
| Epic | [PF-984 — Ticketing System for SCU](https://astronauts-id.atlassian.net/browse/PF-984) `PRD` |
| Relevant Docs | Figma: TBD · TRD: TBD |

---

## Table of Contents
1. [🎯 Context](#context)
2. [❗ Problem Statement](#problem-statement)
3. [📊 Success Metrics](#success-metrics)
4. [⚙️ Proposed Solution](#proposed-solution)
5. [📋 Requirements](#requirements)
6. [✈️ Rollout Strategy](#rollout-strategy)
7. [❓ Open Questions](#open-questions)
8. [📎 Appendix](#appendix)

---

## 🎯 Context

### Who Are We Solving For?

| Persona | Current Pain Points | Current Landscape |
|---|---|---|
| **Seller / Vendor** | No dedicated channel to report issues. Inquiries go through category managers with no acknowledgment, no ticket ID, no visibility on resolution status. | Submits issues informally via WhatsApp to their assigned category manager. Has no idea if the issue is being worked on or who owns it. |
| **SCU Agent** (future hire) | No tooling exists yet. Will be managing seller issues with no structured inbox, no assignment system, no reporting. | Not yet operational — this tooling is a prerequisite for the team to function. |
| **Category Manager** | Currently acting as an informal CX layer for sellers. Receives raw inquiries, triages them manually, then routes to PMs. Not their job. | Serves as the bottleneck between sellers and resolution. Volume is 10–50/week, and growing as seller count grows. |
| **PM (Berta et al.)** | Being pulled into individual seller issues that should be handled by an operations team. | Acts as final escalation point when category managers can't resolve issues, consuming product team bandwidth. |

### Current Landscape

Astro's Seller Portal (`astro-vendor-web`) is a web-based platform used by vendors to manage their relationship with Astro. It has four main sections:

| Portal Menu | What It Covers |
|---|---|
| **Profile** | Vendor profile, address, preferred shipping date, MOQ/MOV, tax/PKP status, contact/PIC |
| **Insight Report** | Fulfillment insight, brand insight, product insight |
| **Invoice** (Pengajuan & Tagihan) | Invoice submission and billing |
| **Daftar PO** | Purchase order list |

The portal is actively being developed. There is no dedicated support channel within it today.

For reference, the **Hub Care Unit (HCU)** is an analogous model already in use at Astro for last-mile mitra issues: the CX team operates from a dedicated admin dashboard module. The SCU ticketing system should follow this same internal-tool pattern, and not require a third-party tool like Zendesk.

### Issue Category Taxonomy

Based on analysis of real seller inquiries surfaced through category manager Google Chat channels (Apr 2026), the following categories have been identified and validated. The taxonomy is aligned to portal menu structure so vendors can self-identify the issue area.

| Category (Bahasa) | Maps to Portal Menu | Observed Volume | Example Issues |
|---|---|---|---|
| **Login & Akses** | Pre-portal | ~15% | OTP not received, portal invite email never arrived, account can't be accessed |
| **Profil & Konfigurasi** | Profile | ~10% | Tax/PKP status not reflected in PO, wrong PIC/contact in PO, address not updating |
| **Daftar PO** | Daftar PO | ~20% | PO not showing in portal, can't download PO, PO amount discrepancy |
| **Invoice & Pembayaran** | Invoice (Pengajuan & Tagihan) | ~40% | Can't submit invoice, invoice rejected + can't resubmit, bank account validation error, tukar faktur process questions |
| **Insight Report** | Insight Report | ~5% | Operational/fulfillment data showing "no data", brand data incorrect |
| **Bug / Error Portal** | Any menu | ~10% | App crash (client-side exception), page loading error, network connectivity error |

> **Note for product team:** Invoice & Pembayaran + Daftar PO together account for ~60% of current inquiry volume. This has direct implications for SCU agent onboarding and triage SOP design — agents should be deeply trained on the invoice submission and PO visibility flows from day one.

---

## ❗ Problem Statement

Currently, 10–50 seller inquiries reach Astro per week, routed entirely through category managers and PMs via informal channels (WhatsApp, verbal handoffs). There is no ticket ID system, no SLA, no category tagging, and no resolution tracking. This creates three compounding problems:

**1. Zero visibility into issue volume and type.** Astro cannot quantify how many issues sellers are experiencing, what categories they fall into, or how long they take to resolve. Decisions about what to prioritize on the Seller Portal have no support data to draw on.

**2. Unsustainable load on category managers and PMs.** Support routing is being absorbed by people whose primary jobs are commercial and product work. As seller count grows, this load will scale proportionally, making it a structural risk to team efficiency.

**3. Poor seller experience.** Sellers who report an issue receive no confirmation, no estimated resolution time, and no feedback on outcome. This erodes trust in the platform at exactly the stage where Astro needs vendors to be invested in growing on the platform.

> *How might we give sellers a direct, trackable channel to report issues from within the Seller Portal — while giving the SCU team the tools to manage, respond to, and resolve those issues without routing them through category managers or PMs?*

How do we think this can be solved?
1. Build a **ticket submission form** inside the Seller Portal so sellers can report issues directly, with a unique ticket ID upon submission.
2. Build a **SCU admin dashboard module** (similar to HCU) where seller care agents can view, assign, respond to, and close tickets.
3. Design the ticket data model to support **category tagging and status tracking** from day one, so Astro gains a reporting baseline immediately after launch.

---

## 📊 Success Metrics

**Opportunities:** There are ~100–200 active vendors on the Seller Portal today. At 10–50 issues/week, that is a meaningful issue-per-vendor ratio for a young platform. The metric targets below are conservative, reflecting that the SCU team is also being stood up in parallel.

| Goal | Metric | Target |
|---|---|---|
| Remove PM & catman from support routing | # of seller inquiries still routed through PM/catman post-launch | 0 within 60 days of launch |
| Establish a baseline of issue data | % of weeks with complete ticket category + resolution data | 100% from Week 1 post-launch |
| Deliver timely seller support | Average first-response time per ticket | < 24 hours within 60 days |
| Demonstrate seller trust in the channel | Ticket resubmission rate (same issue) | < 15% |

---

## ⚙️ Proposed Solution

### What We're Building

This is a **two-sided ticketing system** with a seller-facing submission interface and an agent-facing management interface. No third-party tool (e.g. Zendesk) — the agent side lives in Astro's existing admin dashboard, mirroring the HCU module pattern.

| What | Why (Problem Solved) | How |
|---|---|---|
| Ticket submission form in Seller Portal | Sellers have no structured channel today | New section/page in `astro-vendor-web` with form fields: issue category, subject, description, attachment |
| Unique ticket ID on submission | No accountability or tracking exists | Auto-generated ID assigned on submission; displayed to seller with confirmation |
| Seller-side ticket history & status view | Sellers can't follow up on their own issues | "My Tickets" view in Seller Portal showing ID, category, status, last updated |
| SCU admin module in admin dashboard | SCU agents have no tooling | New module (MVP) with ticket inbox, status management, reply thread, agent assignment |
| Notifications to seller on status change | Sellers are in the dark after submitting | Email + in-portal notification triggered on status change or agent reply |

### Product Vision

| | Current State | Phase 1 (This PRD) | Beyond (Phase 2+) |
|---|---|---|---|
| Seller experience | No channel; ad-hoc WhatsApp | Structured form in portal; ticket ID; status tracking | CSAT survey post-resolution; SLA-committed response times visible to seller |
| Agent experience | No tooling | Ticket inbox, status management, reply, assignment | SLA auto-escalation; internal notes; bulk actions |
| PM/catman involvement | Full routing + resolution | Zero involvement | Zero involvement |
| Data & reporting | No data | Raw ticket volume + category available | Reporting dashboard: volume by category, avg resolution time, open vs. resolved |

---

## 📋 Requirements

### User Stories

| Page / Feature | Requirement | User Story | Acceptance Criteria | Priority | Notes |
|---|---|---|---|---|---|
| **Seller Portal — Submit Ticket** | Seller can create a new ticket | As a seller, I want to submit a support ticket from the Seller Portal so that I can report an issue without contacting my category manager. GIVEN I am logged into the Seller Portal / WHEN I navigate to the support section and fill in the ticket form / THEN my ticket is submitted and I receive a unique ticket ID with a confirmation message. | Happy Path: Form accepts category (required dropdown with 6 options: Login & Akses, Profil & Konfigurasi, Daftar PO, Invoice & Pembayaran, Insight Report, Bug / Error Portal), subject (required, max 200 chars), description (required, max 2000 chars), attachment (optional, max 5MB). Non-Happy Path: Submission fails if category or subject is empty — inline validation shown. Edge Case: Attachment type restricted to jpg, png, pdf. Error State: If submission fails (network/server error), show retry prompt; do not clear form. | P0 | Categories finalized based on real inquiry analysis — see Context section |
| **Seller Portal — Ticket Confirmation** | Seller receives a ticket ID immediately after submission | As a seller, I want to see a confirmation with my ticket ID after submitting so that I have a reference for follow-up. GIVEN I have successfully submitted a ticket / WHEN the form is submitted / THEN I see a confirmation screen showing the ticket ID, issue category, and estimated first response note. | Happy Path: Confirmation displays ticket ID (e.g. SCU-00001), category, submission timestamp. Edge Case: Confirmation is still shown if the seller refreshes — ticket is not duplicated. | P0 | |
| **Seller Portal — My Tickets** | Seller can view their ticket history and statuses | As a seller, I want to see all my submitted tickets and their current statuses so that I know if my issues are being worked on. GIVEN I am on the My Tickets page / WHEN the page loads / THEN I see a list of my tickets with ticket ID, category, subject, status, and last updated date. | Happy Path: List shows all tickets sorted by most recently updated. Status labels: Open, In Progress, Resolved, Closed. Non-Happy Path: Empty state if no tickets submitted — show prompt to submit first ticket. Edge Case: Pagination or infinite scroll for sellers with many tickets (>20). | P0 | |
| **Seller Portal — Ticket Detail** | Seller can view the full thread of a ticket including agent replies | As a seller, I want to see the full conversation thread on my ticket so that I can understand what the agent has responded. GIVEN I click on a ticket in My Tickets / WHEN the detail page loads / THEN I see the original submission, all agent replies (chronological), current status, and ticket metadata. | Happy Path: Thread view shows submissions and replies in chronological order, timestamps on all entries. Non-Happy Path: If ticket has no reply yet, show "Awaiting response" state. Edge Case: Seller cannot edit the original submission after it is submitted. | P0 | |
| **SCU Admin — Ticket Inbox** | SCU agent can see all incoming tickets | As an SCU agent, I want to see all open tickets in one place so that I can triage and respond without losing context. GIVEN I am logged into the admin dashboard / WHEN I open the SCU module / THEN I see a paginated list of all tickets with columns: Ticket ID, Seller Name, Category, Subject, Status, Assigned Agent, Last Updated. | Happy Path: Default sort by oldest unresponded ticket first. Filter options: by status, by category. Search by ticket ID or seller name. Non-Happy Path: Empty state if no tickets. Edge Case: Tickets from deleted seller accounts should remain visible. | P0 | RBAC: SCU agent role required |
| **SCU Admin — Update Status** | SCU agent can update ticket status | As an SCU agent, I want to update a ticket's status so that sellers and other agents know where their issue stands. GIVEN I am on a ticket detail page / WHEN I change the status / THEN the ticket status updates immediately and the seller is notified. | Happy Path: Status transitions allowed: Open → In Progress → Resolved → Closed. Non-Happy Path: Agent cannot move to Closed without adding at least one reply. Edge Case: Closed tickets cannot be reopened by agents — flagged in Open Questions. | P0 | |
| **SCU Admin — Reply to Ticket** | SCU agent can reply on a ticket (seller-visible) | As an SCU agent, I want to add a reply to a ticket so that the seller knows what action is being taken. GIVEN I am on a ticket detail page / WHEN I add a reply and submit / THEN the reply appears in the thread and the seller receives a notification. | Happy Path: Reply field accepts plain text, max 2000 chars. Submission triggers notification to seller. Non-Happy Path: Empty reply cannot be submitted. Error State: If submission fails, show retry with reply text preserved. | P0 | |
| **SCU Admin — Assign Ticket** | SCU agent can assign a ticket to a team member | As an SCU agent, I want to assign a ticket to a specific teammate so that ownership is clear. GIVEN I am on a ticket detail page / WHEN I select an agent from the assignment dropdown / THEN the ticket is assigned and the assigned agent is notified. | Happy Path: Dropdown lists all active SCU agents. Assignee is displayed in ticket header and inbox list. Non-Happy Path: If the selected agent is inactive, show error. Edge Case: Unassigned tickets should be visually distinct in inbox. | P1 | |
| **SCU Admin — Internal Notes** | SCU agent can add notes not visible to seller | As an SCU agent, I want to add internal notes on a ticket so that I can leave context for teammates without exposing it to the seller. GIVEN I am on a ticket detail page / WHEN I add an internal note / THEN it appears in the thread visually distinct from seller-visible replies and is not sent to the seller. | Happy Path: Internal notes displayed with a distinct background (e.g. yellow). Clearly labeled "Internal note — not visible to seller". Error State: Internal note toggle must be explicit; default to visible reply to prevent accidental exposure. | P1 | |
| **Seller Portal + Admin — Notifications** | Seller is notified on status change or new reply | As a seller, I want to receive a notification when my ticket is updated so that I know when to check back. GIVEN my ticket status changes or an agent adds a reply / WHEN the change is saved / THEN I receive an in-portal notification and an email notification. | Happy Path: In-portal notification badge updates; email sent to seller's registered email. Non-Happy Path: If email delivery fails, in-portal notification still fires. Edge Case: Seller can view notification even if they are not logged in (email CTA links back to ticket). | P0 | Depends on existing notification infrastructure in Seller Portal — flagged in Open Questions |
| **SCU Admin — Ticket Priority** | SCU agent can tag a ticket with priority | As an SCU agent, I want to mark a ticket as High, Medium, or Low priority so that critical issues are addressed first. GIVEN I am on a ticket detail page / WHEN I set a priority / THEN the priority label is visible in the ticket detail and inbox list. | Happy Path: Default priority is Medium on all new tickets. Priority label visible in inbox, sortable. Non-Happy Path: Priority cannot be blank once set (but can be changed). | P1 | |

---

## ✈️ Rollout Strategy

| Phase | Description | Graduation Criteria |
|---|---|---|
| Phase 0 — Internal Prep | Agree on issue category taxonomy with SCU lead and category managers. Provision SCU agent accounts. Migrate 2–3 historically known open issues manually to seed the system. | Taxonomy finalized; ≥2 SCU agent accounts set up and tested; QA sign-off on core submission + agent reply flow |
| Phase 1 — Soft Launch | Enable ticket submission for all sellers. SCU agents begin using admin module as primary tool. Category managers and PMs directed to forward any incoming seller inquiries to SCU. | Zero new issues routed through PM or catman for 2 consecutive weeks; ≥20 tickets processed through the system |
| Phase 2 — Reporting + Priority | Ship P1 features: priority tagging, internal notes, assignment, basic volume reporting. | P1 feature QA passed; reporting data aligns with raw ticket count from Phase 1 |

---

## ❓ Open Questions

| Question | Answer | Date Answered | Owner |
|---|---|---|---|
| ~~What are the issue category options?~~ Issue category taxonomy is finalized — see Context section. | ✅ Resolved 14 Apr 2026 | 14 Apr 2026 | Berta |
| Does the Seller Portal currently have email notification infrastructure? Or do we need to build this from scratch? | TBD | — | Engineering (SAP team) |
| Who owns the SCU admin module — SAP engineering or a separate backend? Does HCU have a pattern we can reuse? | TBD | — | Engineering |
| Can a closed ticket be reopened by a seller if their issue recurs? What's the expected behavior? | TBD | — | Berta + SCU Lead |
| Is there an RBAC role for SCU agents already, or does one need to be created? | TBD | — | Engineering |
| What is the expected team size of the SCU at launch — 1 agent, 3, more? This affects whether assignment and queue management are P0 vs. P1. | TBD | — | Berta + SCU hiring |
| Should sellers be able to attach files to replies (follow-up messages), or only on initial submission? | TBD | — | Berta + Design |

---

## 📎 Appendix

### MVP Scope (Phase 1)

- **Seller Portal**
  - Ticket submission form (category, subject, description, optional attachment)
  - Ticket confirmation with unique ticket ID
  - My Tickets list view (ID, category, status, last updated)
  - Ticket detail view with conversation thread
  - In-portal + email notification on status change / new reply
- **SCU Admin Module**
  - Ticket inbox with filter (by status, by category) and search
  - Ticket detail page with full thread view
  - Status management (Open → In Progress → Resolved → Closed)
  - Seller-visible reply
  - Agent assignment

### Out of Scope (Phase 1)

- **Not in this release**
  - WhatsApp or email submission channels (web portal only for v1)
  - Zendesk or any third-party support tool integration
  - Help page / FAQ self-service (separate initiative)
  - SLA auto-escalation / breach alerts
  - CSAT survey post-resolution
  - Bulk ticket actions
  - Jira auto-creation for bug-type tickets
  - Mobile interface for SCU agents
  - AI-powered categorization or routing
- **Future consideration**
  - SLA commitments visible to sellers (Phase 2+)
  - CSAT feedback loop feeding back into Seller Portal product prioritization
  - Reporting dashboard with volume trends, category breakdown, avg resolution time

### Roadmap

| Phase | Timeline | Scope | Goal | Success Criteria |
|---|---|---|---|---|
| Phase 0 | W1–W2 May 2026 | Category taxonomy, RBAC provisioning, QA | Pre-launch prep | Taxonomy agreed, 2+ agents onboarded, QA sign-off |
| Phase 1 — MVP | W3 May – W2 Jun 2026 | Submission form, My Tickets, SCU inbox, reply, status, notifications | Live ticketing operational | 0 issues via PM/catman for 2 weeks |
| Phase 2 — P1 Features | W3–W4 Jun 2026 | Priority, internal notes, assignment, basic reporting | Operational maturity | P1 QA passed; reporting live |
