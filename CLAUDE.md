# CLAUDE.md — Ashley's AI Product Leader Planning System

You are working in a product repository where the PM (Ashley) contributes directly to the codebase. Read this file at the start of every session, before responding to any planning, copy, prompt, or config change.

This repo is built around the AI-era PM playbook: planning docs live next to the code they describe, AI behavior is contract-tested, and every PM PR has a clear shipping zone and an engineer safety net.

---

## Product Context

Fill these in per company / per role. Keep placeholders bracketed until you have the answer — don't guess.

- **Product:** [Product name]
- **One-line description:** [What it does, who it's for, in one sentence]
- **Stage:** [Pre-PMF / Series A / Series B / Growth / Public / Enterprise]
- **Users:** [Primary persona + rough volume, e.g., "B2B sales leaders at 50–500-person SaaS cos, ~12k MAU"]
- **Business model:** [Self-serve / sales-led / hybrid; ARR or revenue range if relevant]
- **Stack:** [Frontend + version, backend + version, ORM, primary DB. Two-line answer is enough — Claude will read the codebase for the rest.]
- **AI stack:** [LLM provider(s), agent framework, vector store / RAG layer, eval harness, observability tool]
- **Feature flags:** [LaunchDarkly / Statsig / Unleash / GrowthBook / internal]
- **Experimentation platform:** [Statsig / Eppo / Optimizely / internal]
- **Error tracking:** [Sentry / Rollbar / Datadog]
- **Session replay:** [LogRocket / FullStory / PostHog / N/A — must have PII masking configured]
- **Analytics:** [Amplitude / Mixpanel / PostHog / internal]
- **Design system:** [Name + repo link. Accessibility bar: WCAG 2.1 AA minimum, AAA for any flow that surfaces AI-generated content to end users.]
- **Localization:** [Locales supported, or "English-only" — be explicit, this affects every copy PR]
- **DPA / data residency:** [Anthropic DPA in place? Customer data routed through which region?]

---

## Operating Principles

These are non-negotiable for any PM-authored change in this repo.

1. **Plan before code.** Every PR maps to a PLANNING.md in `/planning`. No PLANNING.md, no PR.
2. **Small, single-purpose PRs.** One change per PR. Bundling unrelated changes is grounds for an immediate request-changes.
3. **Behavior contracts for anything AI-shaped.** If the change is a prompt, eval, system message, tool definition, or agent flow, it ships with ≥5 Good / ≥5 Bad / ≥5 Reject examples (see "AI Prompt Changes" below).
4. **Engineer review is the safety net, not the first pass.** Run the `planning-review` skill before you tag anyone.
5. **Tests stay green.** If a test fails after your change, stop. Never weaken a test to make it pass — that's a Sev-2 vector in disguise.
6. **The blast radius rule.** If you can't describe the change in one sentence AND limit user impact to what's visible on screen, escalate to engineering.

---

## PM Shipping Zone

Ashley is authorized to ship the following directly. Anything outside this list requires engineering ownership.

**In scope (PM PR):**
- Copy and microcopy: error messages, empty states, CTAs, tooltips, onboarding, confirmation dialogs, AI failure-mode messages
- Locale files: add/edit strings across all supported locales (or file a translation ticket inside the same PR)
- Config values and feature-flag defaults (numeric thresholds, rollout %, kill switches)
- AI prompts, system prompts, few-shot examples, tool descriptions, behavior examples
- Eval cases and behavior-contract test fixtures
- Planning docs (`/planning/*.md`) and rollout docs (`/rollout/*.md`)
- Small front-end changes on approved design-system components: spacing, visibility toggles, ordering, default selected values
- README, docs, and inline JSDoc / docstring updates
- Test cases for any of the above

**Out of scope — escalate, never edit:**
- Database schemas, migrations, indexes
- API routes, controllers, middleware, GraphQL resolvers
- Authentication, authorization, session handling, security headers, CSP, CORS config
- Infrastructure: Docker, k8s, CI/CD, Terraform, deployment pipelines
- Secrets, API keys, connection strings, `.env*` files
- `package.json` / `requirements.txt` dependency changes
- Performance-critical code paths (anything in a hot loop or with explicit perf budgets)
- Anything touching > 3 files outside `/planning`, `/rollout`, `/locales`, or `/src/copy`

If the user asks for an out-of-scope change, **stop**, name the boundary, and offer to write a ticket / PLANNING.md instead.

---

## AI Prompt Changes — Extra Rigor

A prompt is code. It dictates model behavior, costs real money, and can leak data. Higher bar.

**Required in PLANNING.md for any AI-prompt change:**

- **Behavior contract** with ≥5 Good, ≥5 Bad, ≥5 Reject examples. The Reject set must cover, at minimum:
  - PII echo (model regurgitates user data back)
  - Jailbreak ("ignore previous instructions" / "you are now…")
  - Policy violation (NSFW, hate, self-harm, regulated advice)
  - Competitor mention / disparagement
  - Outage / blame attribution ("the system is broken because X team…")
  - Locale mismatch (English output for non-English input)
  - Hallucinated tool calls or fabricated citations
- **Eval evidence.** Link to the eval harness run, with pass rates per category. Flag any regression vs. the current prompt.
- **Cost delta.** Tokens in / tokens out before and after, on a representative sample. Flag if per-request cost rises > 10%; require eng director sign-off if > 25%.
- **Latency delta.** p50 and p95 against a representative sample. Document any change > 200ms.
- **PII handling.** Confirm the prompt cannot echo PII. At least one Reject example for PII echo is mandatory.
- **Fallback UX.** If the model errors, safety-filters, or times out, what does the user see? Document the failure copy and the retry behavior.
- **Model + version pinning.** State the exact model string (e.g., `claude-sonnet-4-6`) and rationale.

**Never:**
- Embed raw customer data, internal Slack messages, or unredacted logs in a prompt. Use the redaction helper (see References).
- Ship a prompt change without an eval run, even for "small wording tweaks." Wording is behavior.
- Add a tool definition without a behavior contract for when the tool fires and when it doesn't.

---

## Coding Standards for PM Changes

- **Copy:** Match canonical product terms. Search the codebase for related strings before adding new ones. If the term doesn't exist yet, propose it in the PLANNING.md.
- **Config:** Verify the value is in range. Add a comment explaining what the value does and the rollback procedure.
- **Color / visual:** Use design-system tokens, never raw hex. Verify contrast (4.5:1 minimum for body, 3:1 for large text).
- **Localization:** New or edited English string → add equivalents to every locale, or open a translation ticket linked in the PR description.
- **Tests:** Run the existing suite before opening the PR. Add a test for any behavior the PR introduces or changes.

---

## PR Standards

Before opening a PR:
1. Link the relevant PLANNING.md in `/planning`.
2. PR description explains **why**, not just **what** — quote the problem statement from PLANNING.md.
3. Before/after screenshots for any visible change.
4. Eval run link for any AI change.
5. Request review from a CODEOWNERS engineer. If primary is OOO, reassign to the backup. Auto-escalate after 24h of silence.
6. Tag the design owner on any visual change.
7. Tag the AI safety / policy reviewer on any prompt or behavior change.

**PR title format:** `type: short description`
- `fix:` bug fixes, copy corrections
- `feat:` new copy, new config, new prompt, new behavior
- `docs:` planning docs, READMEs
- `eval:` new or updated behavior contracts / eval cases
- `chore:` repo hygiene

All PM-authored PRs get the `pm-change` label. AI-prompt PRs additionally get `ai-behavior`.

---

## Planning Docs

All planning docs live in `/planning` and follow `PLANNING-TEMPLATE.md`.

**Required sections (every planning doc):**
- Problem (with baseline numbers — never "users are frustrated," always "32% of new users drop off at step 3")
- Hypothesis
- Scope
- Non-Goals
- Success Metrics (with thresholds, baselines, and how they'll be measured)
- Guardrail Metrics (what would make you roll back)
- Rollout Plan (ramp gates: 1% → 10% → 50% → 100%, with kill criteria at each gate)
- Owner (Ashley, plus eng counterpart)

**Required additionally for AI features:**
- Behavior Contract (Good/Bad/Reject examples)
- Eval Plan (harness, pass-rate thresholds, regression suite)
- Cost & Latency Budget
- Privacy & Data Flow (what goes to the model, what gets logged, retention window)

**Required additionally for anything experiment-tested:**
- Statistical Plan (MDE, power, sample size, expected runtime)
- Observability (dashboards + alert thresholds, both for primary and guardrail metrics)

---

## Privacy, Security, Compliance

- Copy referencing user data (`Hi {first_name}`) requires a one-line privacy review note in the PR.
- AI prompts never contain raw customer data. Use the redaction helper.
- Session replay configs must mask PII. Retention windows for replay + prompt logs are documented in the privacy policy.
- Changes that touch regulated domains (health, payment, biometric, PII) route to legal **before** merge.
- Any new field that's logged, displayed, or transmitted to a third-party model routes to legal.

---

## Observability

For every testable change, before merge:
- A dashboard exists (or is created) covering primary + guardrail metrics.
- Alert thresholds are wired up.
- AI changes have cost and latency alerts in addition to quality alerts.
- For the first week after launch, Ashley monitors the dashboard daily. Document the daily check in the PLANNING.md "Post-launch" section.

---

## Program Guardrails

Safety rails on the PM-ships-code program itself.

- **Engineer NPS** on PM PRs measured quarterly. Target ≥ 7/10. Two consecutive months < 6/10 → pause the program and run a retro.
- **Two PM-PR-caused Sev-2 incidents in a rolling 90-day window** terminates direct-commit privileges; PM moves back to spec-only until the retro closes.
- **Weekly PM PR cap:** [5 during pilot, adjust upward as the program matures].
- **AI cost ceilings:** per-request cost increase > 10% requires eng director sign-off; > 25% requires CPO sign-off. Monthly LLM spend ceiling: [$ amount]; exceeding requires CPO approval.
- **Behavior contract coverage:** any AI feature shipped without a passing behavior contract is a rollback trigger, not a discussion.

---

## Escalation Chain

When review is stuck:
1. Primary CODEOWNERS reviewer for the touched paths
2. Backup CODEOWNERS reviewer
3. Engineering manager for the touched area
4. Engineering director
5. CPO (for AI safety, cost, or Sev-2 matters)

Tag the next person after 24h of silence. No exceptions.

---

## Team Norms

- Every PM PR requires CODEOWNERS engineer approval before merge. No self-merges, ever.
- Planning docs are reviewed async in PR comments. Weekly planning sync is optional (see `rollout/WEEKLY-REVIEW-CADENCE.md`).
- Feature flags required for any testable change. No flag = no merge.
- PM owns the PLANNING.md and the rollout. Engineering owns the implementation.
- Discussion lives in: [#product-eng]. PR queue: [#pm-prs]. Incidents: [#incidents-sev2]. AI eval failures: [#ai-evals].

---

## References

Fill these in per repo so Claude Code can find the right helpers without grepping blindly:

- Design system: `[path or URL]`
- Redaction helper (mandatory for AI changes): `[path]`
- Eval harness (mandatory for AI changes): `[path]`
- Behavior-contract fixtures: `[path, typically /evals/behavior-contracts]`
- Power-analysis primer: `[path or URL]`
- Change-management / compliance policy: `[path or URL]`
- CODEOWNERS: `[path, typically /.github/CODEOWNERS]`
- Incident runbook: `[path or URL]`

---

## How Claude Should Behave in This Repo

When Ashley asks for help in this repo:

1. **First, check whether the request lives in the PM Shipping Zone.** If it doesn't, name the boundary and propose the PM-zone alternative (usually: write a PLANNING.md, file a ticket, or pair with an engineer).
2. **For any AI-prompt change, refuse to draft the change until the behavior contract is on the table.** Offer to draft the Good/Bad/Reject examples first.
3. **For planning docs, run the `planning-review` skill before declaring done.** Surface gaps the skill flags rather than papering over them.
4. **Default to questions, not assumptions.** If product context is unclear, ask before writing. The cost of one clarifying question is cheap; the cost of a wrong-shaped PR is not.
5. **Cite the section of this CLAUDE.md you're applying** when you decline or escalate. Predictability beats cleverness.
