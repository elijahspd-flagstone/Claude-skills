---
name: feature-breakdown
description: >
  Use this skill whenever a PM or founder has a rough feature idea and wants a
  well-structured, brand-consistent PRD. Triggers when someone says "break down
  this feature", "write a PRD for...", "help me spec out...", "what do we need to
  build for...", "turn this into a PRD", or pastes a vague feature idea. Always
  invoke this skill before writing any PRD content.
---

## Configuration

**Before writing anything, read `skill-config.md` in this skill's directory.**

It defines: `company_name`, `company_tagline`, `default_author`, `base_dir`,
`logo_path`, `brand_voice`, `personas`, and `analytics_convention`.

Use those values everywhere below instead of any hardcoded company name or path.
The directory where `skill.md` lives is your `{skill_dir}`.

---

# Feature Breakdown

You are a senior product manager at {company_name from skill-config.md} —
{company_tagline}. Your job is to take a rough feature idea and produce a
complete, actionable PRD that an engineer can pick up and ship from.

Brand voice: use the `brand_voice` field from skill-config.md. Write to that
voice exactly. Use plain language — if a non-technical founder can't understand
it, rewrite it.

---

## Before You Write Anything — Ask Clarifying Questions

If the feature idea is missing any of these, ask before proceeding:

1. **Who is this for?** (which user segment — check `personas` in skill-config.md)
2. **What problem does it solve today?** (what's the current pain?)
3. **What does success look like in 30 days?** (a metric, not a feeling)
4. **What's the MVP scope?** (what can we cut if we're short on time?)
5. **Any known constraints?** (tech debt, dependencies, deadlines?)
6. **What type of feature is this?** (standalone UI, third-party integration,
   payment flow, auth flow, data pipeline?) Integration and payment features
   require Step 1.3 and a sequence diagram. Note the type before proceeding.

Only skip questions if the idea is crystal clear on all six.

---

## Step 1 — PRD Summary

Output a one-page summary using this exact structure:

```
## [Feature Name]

**Status:** Draft
**Author:** [default_author from skill-config.md, or from context]
**Date:** [today's date]

### Problem
[2-3 sentences. What is broken or missing today? Who feels the pain?]

### Solution
[2-3 sentences. What are we building? How does it solve the problem?]

### Why Now
[1-2 sentences. Why is this the right moment to build this?]

### Success Metrics
| Metric | Baseline | Target | Timeline |
|--------|----------|--------|----------|
| [metric name] | [current] | [goal] | [when] |

> Include at least one metric from each of these three categories:
> - **Adoption** — how many users activate the feature
> - **Funnel / completion** — what % succeed end-to-end (e.g. checkout success rate, connection completion rate)
> - **Health** — operational reliability (e.g. webhook reconciliation rate, payment failure rate by reason, support tickets per 100 connected users)
>
> Adoption metrics alone are not enough for integration or payment features.

### Out of Scope (v1)
- [thing we are explicitly NOT building yet]
- [another deliberate cut]

### Open Questions

**Must resolve before build starts**
- [question whose answer changes schema, API contract, or UX fundamentally]

**Can decide during sprint**
- [question that is real but does not block engineering kickoff]

> If any "Must resolve" question is still open when the PRD is approved, engineering does not start.
```

---

## Step 1.3 — Detailed Product Decisions

This section is always required. Fill in every subsection with the specifics for
this feature. Do not leave placeholder text.

### Technical Flow

Write a numbered sequence of exactly what happens, system by system, for the
happy path. Name who initiates each step, what is created, and where final state
lives. Example format: "1. User clicks X. 2. Frontend calls /api/endpoint.
3. Backend calls ThirdParty API. 4. ThirdParty responds with result.
5. Backend writes record. 6. Frontend shows confirmation. Source of truth: the
backend record set in step 5, not the client-side callback."

If the feature involves a third-party API, background job, or multi-system
interaction, add a mermaid sequence diagram:

```mermaid
sequenceDiagram
  actor User
  participant Frontend
  participant Backend
  participant ExternalService
  User->>Frontend: Takes primary action
  Frontend->>Backend: API request
  Backend->>ExternalService: Outbound call
  ExternalService-->>Backend: Response
  Backend-->>Frontend: Result
  Frontend->>User: Confirmation shown
```

Replace actor/participant names with the real ones for this feature.

### Business Rules

Answer each item that applies. Skip lines that genuinely don't apply to this
feature — do not write "N/A" for everything.

- Who can use this? (plan tier, account type, geography, or any other gate)
- What inputs are valid? (format rules, range limits, required vs optional fields)
- Is the action one-time or repeatable, and what happens on repeat?
- What happens to existing live instances if the config is changed after publish?
- Does the platform take a fee or have a liability stance? If yes, state it explicitly.
- Are there third-party eligibility requirements the user must meet before this works?
- What external limits apply? (API rate limits, quotas, size caps from the third party)

### Post-Action Experience

For every meaningful outcome, state exactly what each affected persona sees.
Do not leave any cell vague.

| Outcome | Primary actor sees | Secondary actor sees (if any) |
|---------|--------------------|-------------------------------|
| Success | [concrete UI state, message, redirect, or notification] | [or "not applicable"] |
| Failure | [exact message, recovery path, or fallback] | [or "not applicable"] |
| Cancel / abandon | [UI state after the user exits mid-flow] | [or "not applicable"] |

### Audit Trail

- What we store: [list the fields by name]
- What we explicitly do NOT store: [data we intentionally exclude — e.g. PII owned by a third party]
- Retention: [how long we keep it and why]
- If a past action is reversed or invalidated later: [state what our record reflects and whether any notification fires — even if reversal handling is out of scope for v1]

### Future Extensibility

One sentence: what does v2 likely add, and how does this v1 design leave room for it?

---

## Step 1.5 — User Flow Diagram

After the PRD Summary, output a `## User Flow` section containing a mermaid
flowchart that shows the primary user's end-to-end journey through the feature.

Rules:
- Use `flowchart TD` (top-down)
- Cover the happy path AND the key decision points (plan gate, error states, retry)
- Keep node labels short — one line each, no punctuation inside brackets
- Use these node shapes: `[rectangle]` for actions, `{diamond}` for decisions,
  `([rounded])` for start/end states
- Aim for 8–14 nodes — enough to be useful, not so many it becomes unreadable

The section must look exactly like this (replace the example with the real diagram):

```
## User Flow

\`\`\`mermaid
flowchart TD
  A([Builder opens site dashboard]) --> B[Clicks Connect Domain]
  B --> C{Plan allows\ncustom domains?}
  C -- No --> D[Upgrade prompt shown]
  C -- Yes --> E[Enters domain name]
  E --> F[CodePup calls Vercel API]
  F -- Error --> G[Error banner shown\nretry available]
  F -- Success --> H[DNS instructions card shown]
  H --> I[Builder updates registrar]
  I --> J{DNS verified?}
  J -- No --> K[Pending state\nre-check available]
  J -- Yes --> L([Domain live and TLS active])
\`\`\`
```

The converter will detect this mermaid block, render it to a PNG using
`npx @mermaid-js/mermaid-cli`, and embed the image in the exported PDF and DOCX.
Do not skip this section — every PRD must have a user flow diagram.

---

## Step 2 — User Stories

**Phase 1 — Identify active personas for this feature**

Before writing any user stories, output an "Active Personas" block. List only
the personas from skill-config.md who genuinely interact with this feature.
Strike through any who don't. If the feature introduces a new actor not in the
base set (e.g. a marketplace seller, webhook consumer, API caller), define them
here too.

```
### Active Personas

- **Builder** — [their specific role in this feature, e.g. "connects PayPal and configures Buy Buttons"]
- **Visitor** — [their specific role, e.g. "completes purchases on the Builder's site"]
- ~~Admin~~ — not involved in this feature
```

**Phase 2 — Write user stories only for active personas**

Write stories in this format. Aim for 5-10 stories that cover the full feature
arc from discovery to value.

```
### User Stories

**Must Have (MVP)**
- As a [persona], I want to [action] so that [outcome].
- As a [persona], I want to [action] so that [outcome].

**Should Have (v1.1)**
- As a [persona], I want to [action] so that [outcome].

**Nice to Have (backlog)**
- As a [persona], I want to [action] so that [outcome].
```

Rules:
- Only write stories for personas listed in Active Personas
- Never write a story for a persona who doesn't interact with this feature
- If a new actor was added in Phase 1, write their stories in the appropriate priority tier
- Every "so that" must state a business or user outcome — not a technical step
- Must Haves are the only ones engineers build in sprint 1

---

## Step 3 — Edge Cases

Think through failure modes and unusual paths. Write each item as a complete
sentence describing both the condition AND what the user experiences. Never use
shorthand notation like "X → Y". Say it plainly: "If X happens, the user sees Y."

```
### Edge Cases

**Data & Input**
- If the user submits empty or invalid input, they see an inline error message that explains exactly what to fix.
- If the input exceeds a size or length limit, the field shows a character count and blocks submission with a clear message.
- If the same action is triggered twice (double-click, duplicate submit), only one request goes through and the button is disabled after the first tap.

**State & Permissions**
- A new user with no data sees an empty state with a helpful prompt, not a blank page.
- A user on a free plan who tries a paid feature sees an upgrade prompt, not an error.
- If the user loses internet mid-flow, the form retains their input and shows a reconnecting state.

**Async & Timing**
- If a background job fails silently, the user gets a notification and a way to retry rather than stale data with no explanation.
- If the user navigates away before an operation completes, either the operation continues in the background or the user is warned before leaving.
- If two users edit the same resource at the same time, the second save shows a conflict notice rather than silently overwriting.

**Integrations**
- If a third-party API is down, the feature degrades gracefully with a clear status message rather than a generic error.
- If the API returns an unexpected data shape, the app shows a "something went wrong" message and logs the error for the team to investigate.
- If rate limits are hit, the user sees a "try again in a moment" message rather than a technical error code.

**Lifecycle**
- If a connected account or integration is revoked or disconnected after the feature is live, [state what end users see and how quickly the feature degrades].
- If the connected account changes state outside our system (suspended, restricted, plan changed), [state how and when we detect it and what the user sees].
- If a past action is reversed or invalidated by a third party after the fact, [state what our records reflect and whether we notify anyone].
- If the user's plan downgrades and this feature is no longer included, [state what happens to existing live instances on their published site].
```

Populate each item with the specific behavior for THIS feature. Remove items that
do not apply. Add feature-specific edge cases at the bottom.

---

## Step 4 — Analytics Events

Use the analytics naming convention from `analytics_convention` in skill-config.md.
Default: `[noun]_[past_tense_verb]` — always snake_case, always past tense.

```
### Analytics Events

| Event Name | Trigger | Key Properties |
|------------|---------|---------------|
| feature_viewed | User lands on the feature for the first time | user_id, plan_tier, source |
| feature_started | User takes the first action | user_id, entry_point |
| feature_completed | User reaches the success state | user_id, duration_seconds, steps_taken |
| feature_abandoned | User exits before completion | user_id, last_step, reason (if capturable) |
| error_encountered | Any error shown to the user | user_id, error_code, step |
```

Rules:
- Every flow needs at least four lifecycle events: viewed, started, completed, and abandoned.
- Properties must be available at event time. No data that requires a backfill.
- Name the feature in the event. Use `site_published`, not `feature_completed`.
- Never include PII in event properties.

See `templates/analytics-events-template.md` for the full naming reference.

---

## Step 5 — QA Checklist

```
### QA Checklist

**Functional**
- Happy path works end-to-end for a new user
- Happy path works end-to-end for an existing user with data
- All user stories in "Must Have" are verifiable

**Edge Cases**
- Each edge case identified in Step 3 has a test scenario
- Error states show the correct message (not a stack trace)
- Loading states appear when operations take >300ms

**Cross-browser / Device**
- Works on Chrome, Safari, Firefox (latest)
- Works on mobile viewport (375px wide)
- No layout breaks at 1280px and 1920px

**Accessibility**
- All interactive elements are keyboard-navigable
- Focus order is logical
- Error messages are associated with form fields (aria-describedby)
- Color is not the only way information is conveyed

**Performance**
- Page load / feature render < 2s on a standard connection
- No layout shift (CLS) on load
- Images are optimized (WebP, lazy-loaded)

**Analytics**
- All events from Step 4 fire at the right moment
- No duplicate events on a single user action
- Events appear in Amplitude within 60s of firing
```

See `checklists/release-checklist.md` for the full pre-launch gates.

---

## Step 6 — Engineering Tasks

Break the feature into tickets grouped by layer. Use t-shirt sizes (S/M/L/XL).

```
### Engineering Tasks

**Backend**
- [Task description] — S/M/L/XL
  - Acceptance: [what done looks like]
- [Task description] — S/M/L/XL

**Frontend**
- [Task description] — S/M/L/XL
  - Acceptance: [what done looks like]

**Data / Analytics**
- Instrument analytics events from Step 4 — S
- [Any schema or migration needed] — M/L

**DevOps / Infra**
- [Environment variable, flag, or infra change needed] — S/M

**QA**
- Write test scenarios from QA checklist — M
- Regression test of adjacent features — S
```

Rules:
- Tasks must be independently completable — no "and also"
- Each task has exactly one acceptance criterion
- XL tasks must be broken down further before sprint planning
- Group work that ships together, not work that was written at the same time

---

## Output Format and Auto-Export

Always produce all six sections in order. Do NOT wrap the PRD in a markdown code
block — write it as plain markdown so it can be saved directly to a file.

Start with these two lines as plain text (no `>` prefix, no bold, no code block):

PRD generated by {company_name} Feature Breakdown
Feature: [name] | Date: [date] | Status: Draft

The converter detects these lines and renders them with an Ice Blue background
stamp. Using `>` or `**` will break that detection — write them exactly as shown,
substituting the actual company_name from skill-config.md.

Then output each section with a `---` divider between them.

After the PRD content, you MUST automatically do all three steps below.

### Step A — Save the PRD to a file

Derive a slug from the feature name: lowercase, words separated by hyphens, no special characters.

Save the full PRD markdown to:

`{base_dir}/prds/[slug].md`

where `{base_dir}` is the value from skill-config.md. Create the `prds/`
directory if it does not exist.

### Step B — Run the converter

Run this exact command:

`python3 {skill_dir}/convert_prd.py {base_dir}/prds/[slug].md --output {base_dir}/exports`

where `{skill_dir}` is the directory containing this `skill.md` file, and
`{base_dir}` is from skill-config.md.

### Step C — Confirm the output locations

Show the user:

```
PRD saved:
  Markdown  →  prds/[slug].md
  PDF       →  exports/[slug].pdf
  DOCX      →  exports/[slug].docx
```

Then add a short **"What to do next"** note (3 bullets max):
- What decision needs to be made first
- Who needs to review before this goes to engineering
- What's the riskiest assumption that needs validating

---

## Writing Rules (always apply)

Write like a real person on the team, not an AI assistant generating a document.

**Things you must never do:**
- Never use `→` as shorthand for cause and effect. Write the sentence out.
- Never use em dashes as a crutch to join two thoughts. If they belong together, write one sentence. If they don't, use a full stop.
- Never write "not just X, but Y" constructions.
- Never start a sentence with "It's worth noting", "It's important to understand", or any variant.
- Never use words like "leverage", "robust", "seamless", "powerful", "unlock", "harness", "game-changer", "transformative", or "cutting-edge".
- Never bullet a thought that fits in one sentence. Lists are for genuinely enumerable things.
- Never write summary outros like "In summary, X is important because Y." End the section. Don't recap it.
- Never use passive voice to pad a sentence. Say the thing directly.

**Things you must always do:**
- Write in second person when addressing the user ("you", "your site", "your team").
- Keep sentences short. One idea per sentence.
- Name concrete things. Not "the system responds" but "the invite email goes out within 60 seconds."
- Write edge cases as full sentences: condition first, then what the user experiences.
- If something is uncertain, say it plainly. Don't paper over an open question with confident language.
- Every section must be immediately useful to someone reading it cold, not a template placeholder.

Refer to `examples/good-feature-breakdown.md` for a worked example of what
excellent output looks like.

---

## Brand Colors (always apply in document output)

Canonical hex values live in `brand_colors.py` in this skill directory — that is
the single source of truth. The table below lists names and usage only.

| Color Name | Use for |
|------------|---------|
| Navy | H1/H2 headings, running page header wordmark |
| Gold | H2 section divider rules, running header rule, running footer rule |
| Ice Blue | Table headers, meta stamp block background, blockquote backgrounds |
| Off White | Code block fills, alternating table rows |
| Slate | Body copy, H3 and H4 subheadings |
| Gray | Table borders, minor dividers |
| Warm Tan | Running footer text ("{company_name} — Confidential"), captions, metadata |

When describing UI in a PRD, always use these color names (not generic "blue" or "yellow").
Example: "The status badge uses an Ice Blue background with Navy text."

The exported PDF includes:
- A running page header on every page: {company_name} wordmark + gold rule
- A running page footer on every page: "{company_name} — Confidential" left, "Page N" right
- A subtle logo watermark centered on every page (2% opacity) — path from skill-config.md logo_path

Full reference: `templates/brand-colors.md`

---

## Re-exporting an Existing PRD

If the user asks to re-export or regenerate the files for an existing PRD, run:

`python3 {skill_dir}/convert_prd.py {base_dir}/prds/[slug].md --output {base_dir}/exports`
