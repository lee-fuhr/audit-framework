---
name: qq-audit-master
description: Meta-orchestrator that selects and sequences the right combination of audit domains based on project context, conversation state, and user intent. Routes to 13 domain-specific audit skills containing 255 expert-persona frameworks.
version: 1.0.0
author: Lee Fuhr
triggers:
  - "full audit"
  - "audit everything"
  - "quality audit"
  - "what should I audit"
  - "run audits"
  - "comprehensive audit"
---

# Master audit orchestrator

Routes to 13 domain-specific audit skills (255 total expert-persona frameworks) based on what the project needs. Knows which domains matter for which kinds of work and sequences them intelligently.

---

## Modes

### Mode 1: Smart routing (no args or general request)

Triggers: `/qq-audit`, "audit this", "what should I audit", "full audit", etc.

**Analyze the project context and recommend a tailored audit plan:**

1. **Identify the project type** from conversation context, files, and CLAUDE.md:
   - Marketing/brochure site → Visual, Copy, SEO, Performance, Accessibility (via UX #13)
   - SaaS/web app → UX, Product, Visual, Frontend, Performance, Security
   - API/backend service → Backend, Security, Testing, DevOps
   - Internal tool → UX, Product, Frontend, Testing
   - E-commerce → UX, Product, Visual, Performance, SEO, Security, Compliance
   - AI/chatbot → UX, Copy (#16 conversational UI), Product, Frontend

2. **Check what's already been audited** — look for `data/audit-*` reports in the project. Don't re-run domains that were audited recently unless explicitly asked.

3. **Present the recommendation:**

   > "Based on [project type], I'd recommend these audits in this order:
   >
   > 1. **Product** (20 frameworks) — verify the product is right before polishing
   > 2. **UX** (20 frameworks) — then make sure humans can use it
   > 3. **Visual** (22 frameworks) — then make it look professional
   > 4. **Copy** (22 frameworks) — then audit the writing
   > 5. **Frontend** (22 frameworks) — then verify code quality
   >
   > Skip: Backend, DevOps, Compliance (not applicable here)
   >
   > Run all 5? Or pick specific ones?"

4. **Use AskUserQuestion** with the recommended domains as options (multiSelect).

### Mode 2: Specific domains (domain names as args)

Triggers: `/qq-audit ux + copy`, "audit product and visual", "run ux then frontend", etc.

Parse the domain names from args (fuzzy match), sequence them in the recommended order, and execute each by invoking its `/qq-audit-[domain]` skill.

### Mode 3: Preset combos (named profiles)

Triggers: `/qq-audit pre-launch`, `/qq-audit quick`, etc.

| Profile | Domains | When to use |
|---------|---------|-------------|
| **quick** | UX, Visual, Copy | Fast quality check — appearance and usability |
| **pre-launch** | Product, UX, Visual, Copy, Frontend, Performance, Security, SEO | Before going live |
| **post-build** | UX, Visual, Copy, Frontend | After completing a feature build |
| **deep** | All 13 domains | Comprehensive quality audit |
| **code** | Frontend, Backend, Testing, Security, DevOps | Engineering quality |
| **user-facing** | UX, Product, Visual, Copy, Performance | Everything the user touches |
| **marketing** | Copy, Visual, SEO, Performance | Marketing site quality |

### Mode 4: List all domains

Triggers: "list", "what domains", "show me everything", "help", etc.

Show the domain inventory table below.

---

## Domain inventory

| # | Domain | Command | Frameworks | Expert lens |
|---|--------|---------|-----------|-------------|
| 1 | UX | `/qq-audit-ux` | 20 | Does the brain work well with this? |
| 2 | Product | `/qq-audit-product` | 20 | Is this the right product? |
| 3 | Visual UI | `/qq-audit-visual` | 22 | Does this look professional? |
| 4 | Content/Copy | `/qq-audit-copy` | 22 | Is the writing good? |
| 5 | Frontend | `/qq-audit-frontend` | 22 | Is the code well-built? |
| 6 | Performance | `/qq-audit-performance` | 22 | Is it fast? |
| 7 | Security | `/qq-audit-security` | 23 | Is it safe? |
| 8 | QA/Testing | `/qq-audit-testing` | 22 | Is it tested? |
| 9 | Backend/API | `/qq-audit-backend` | 22 | Is the server-side solid? |
| 10 | SEO | `/qq-audit-seo` | 15 | Can people find it? |
| 11 | DevOps | `/qq-audit-devops` | 15 | Is it deployable and resilient? |
| 12 | Data Quality | `/qq-audit-data` | 15 | Is the data trustworthy? |
| 13 | Compliance | `/qq-audit-compliance` | 15 | Is it legally sound? |
| | **Total** | | **255** | |

---

## Recommended domain order

When running multiple domains, this sequence prevents wasted work:

1. **Product** — Is this even the right thing? Fix product problems before polishing.
2. **UX** — Can humans use it? Fix usability before visual design.
3. **Visual** — Does it look professional? Fix aesthetics before copy audit.
4. **Copy** — Is the writing good? Fix content after visual context is settled.
5. **Frontend** — Is the code well-built? Fix code quality after UI is stable.
6. **Backend** — Is the server-side solid?
7. **Performance** — Is it fast? Measure after code is clean.
8. **Security** — Is it safe?
9. **Testing** — Is it tested?
10. **SEO** — Can people find it?
11. **DevOps** — Is it deployable?
12. **Data** — Is the data trustworthy?
13. **Compliance** — Is it legally sound?

The logic: fix the WHAT (product) before the HOW (UX), the HOW before the LOOK (visual), the LOOK before the WORDS (copy), the user-facing before the engineering, the engineering before the operations.

---

## Execution flow

For each selected domain (in order):

1. **Invoke the domain's slash command** — e.g., run `/qq-audit-ux` with any scope constraints
2. **The domain orchestrator handles everything** — smart interview, serial framework execution, fixes, report
3. **Collect the domain's output** — SUS score (for UX), overall assessment, critical findings
4. **Brief Lee on results** — "UX scored 82. 3 critical findings fixed, 5 remaining. Moving to Visual."
5. **Proceed to next domain**

Between domains, ask: "Ready for [next domain]? Or adjust the plan?"

---

## Smart interview (master level)

Before selecting domains, gather context the same way domain orchestrators do — but at the project level:

1. What kind of project is this? (infer from files/conversation)
2. What stage is it at? (prototype, feature-complete, pre-launch, live)
3. What's the quality bar? (internal tool, client deliverable, public product)
4. Any specific concerns? (from conversation — "the forms feel broken", "worried about mobile")
5. What's already been audited? (check for existing audit reports)

Pre-fill everything possible. Only ask about genuine gaps.

---

## Key principles

- **Product before polish** — don't optimize the wrong thing. Validate product decisions first.
- **User-facing before engineering** — fix what users see before fixing what only engineers see.
- **Fix between domains** — each domain's critical issues get fixed before the next domain starts.
- **No redundant work** — if UX #13 (WCAG) already ran, skip accessibility checks in other domains.
- **Cumulative context** — each domain receives findings from previous domains to avoid re-reporting.
- **Lee stays in control** — always present the plan, always pause between domains, always let Lee adjust.
