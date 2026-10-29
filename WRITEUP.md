# Write-up: AI Proposal Triage Design

**Core idea:** the AI reads messy text. Code makes every decision that must be repeatable. The AI never sets a score or rank. See [diagram/flow.png](diagram/flow.png).

**Assumptions (stated instead of waiting for a reply):**
- "Last 5 years" means 2021 onward.
- Relative scores are anchored to every proposal that **passes the mandatory gates**, including flagged ones, so scores do not jump when a flagged proposal is fixed.

## 1. AI vs. rules

| Step | Who | Why |
|------|-----|-----|
| PDF to text, file hash, load criteria snapshot | Code | Mechanical |
| Pull facts into fixed JSON, with a quote for each | **AI** | The same fact is worded many ways (year ranges, "to be confirmed" promises). Rules break on this. |
| Schema, quote, injection and missing-fact checks | Code | Must be strict |
| Mandatory pass/fail gates | Rules | Compliance decisions must be exact and auditable |
| Scoring, ranking, tie-break | Code | Simple arithmetic. AI adds nothing and could vary. |
| One-line reason | **AI** (optional) | Good at plain wording. Sees only final numbers. |

The line: AI where the input is unstructured language, code where the output must be identical every time.

## 2. Consistency and explainability

- **Repeatable:** temperature 0, pinned model, versioned prompt, fixed schema. Scoring is code, so the same facts always give the same ranking.
- **The only variable step is extraction.** It is cached by file hash and criteria version, so a re-run gives the same result. A forced re-extraction is compared with the cache and differences are flagged.
- **Tie-break:** higher experience score, then lower price.
- **Explainable:** each row shows every criterion's 0 to 10 score, its weight, the extracted fact and the exact quote it came from.

## 3. Human in the loop

A person:
- confirms every mandatory gate result, especially fails;
- resolves every flag (missing fact, quote not found, injection detected);
- can override any rank or score, with a required written reason;
- makes the final award. The tool only recommends.

**Audit log (append-only):** who and when, file hashes, criteria version and hash, model and version, prompt version, raw AI output, facts and quotes, flags, gate results, per-criterion scores, reason text, every override (before, after, who, why) and the final decision.

## 4. Changing criteria

- Every edit creates a **new criteria version**. Old versions are never changed.
- Existing scores stay, labelled "scored under v1", and are shown as **stale**.
- Re-scoring reuses stored facts. Weight changes need **no new AI call**. Only a criterion needing a new fact triggers extraction of that fact.
- The reviewer sees a before/after rank comparison. A decision already made under v1 keeps that label.

## 5. Risks and safeguards

| Risk | Safeguard |
|------|-----------|
| Wrong extraction | Every fact needs a quote, checked word for word against the text and shown to the reviewer. |
| Made-up facts | Missing means null, never a guess, and flags the proposal. A reason containing any number not in its input is rejected. |
| Manipulation / prompt injection | Proposal text is untrusted data inside delimiters; the prompt says to ignore instructions in it. The AI fills fixed fields only and cannot score. A separate code scan catches phrases like "rank it first". Hits are flagged and never change the score. |
| Bias (polished writing, big firms) | The AI extracts facts and never judges quality; scores come from numbers. Bidder names can be hidden during first review. |
| Poor PDF / OCR | Low text confidence or empty pages flag the file for manual entry. |
| Cost | One cached extraction call per proposal on a smaller model. No AI for maths. Reason step optional. |
| Data privacy | Provider with no training on inputs and in-region storage. Only proposal text is sent. Access-controlled file storage. |

## Sample set result

- **B, ineligible:** the bidder holds no electrical licence and leaves the work to an unnamed subcontractor, so the mandatory licence gate fails.
- **D, flagged:** passes both gates but gives no completion time, so timeline cannot be scored. Not ranked; reviewer requests a committed schedule.
- **C, ranked with a review note:** its embedded instruction to automated reviewers is ignored and C is scored on real content only. I used `ranked`, not `flagged`, because here `flagged` means "cannot be scored" and C's facts are complete. The manipulation is stated in its reason for the human.

**Rubric (0 to 10 per criterion; total = sum of score x weight / 10):**

| Criterion (weight) | Rule |
|--------------------|------|
| Experience (30) | 10 x installs / most among gate-passers (14). Only 100 kW+ in last 5 years. |
| Price (25) | Linear: 10 at lowest gate-passing price ($231,000), 0 at $260,000 ceiling |
| Timeline (15) | Linear: 10 at fastest gate-passing bid (12 weeks), 0 at 16 weeks |
| Warranty (15) | 10 x (panel + workmanship years) / best total (37) |
| Team (15) | 2.5 each: named lead, crew, safety plan, performance monitoring |

| Proposal | Experience | Price | Timeline | Warranty | Team | **Total** | Rank |
|----------|-----------|-------|----------|----------|------|-----------|------|
| A | 10.0 | 7.6 | 10.0 | 9.5 | 10.0 | **93.2** | 1 |
| E | 5.7 | 6.0 | 7.5 | 10.0 | 10.0 | **73.5** | 2 |
| C | 6.4 | 5.5 | 5.0 | 9.5 | 5.0 | **62.3** | 3 |

- C's team score is 5: weekly progress reports are not system monitoring, and no safety plan is stated.
- C's install period starts in 2020, so some of its 9 installs may be outside the window. Scored as stated with a reviewer note; C stays third either way.
