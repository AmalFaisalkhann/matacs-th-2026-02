# WRITEUP.md

**Reference:** MATACS-TH-2026-02

**Flow in one line:** upload → read text from PDF (code) → AI extracts facts with quotes → code validates → code runs mandatory checks → code scores and ranks → AI writes a one-line reason → a person reviews and decides. See `diagram/`.

## 1. AI vs. rules

**AI (two steps):**
- Extracting facts (price, weeks, warranty years, licence statement) from messy proposal text.
- Writing a one-line plain-English reason from numbers already calculated.

**Plain code (everything else):** PDF-to-text, format and quote checks, mandatory pass/fail, scoring maths, sorting, flags and logging.

**Why the line is here:** AI is good at reading varied wording, but it is not reliable for repeatable maths or strict rules. Pass/fail checks and weights must give the same answer every time and be easy to defend, so code decides. In short: AI reads, code decides.

## 2. Consistency and explainability

- The AI runs at temperature 0 with a pinned model, a fixed JSON format and versioned prompts.
- Each extraction is saved once. Scoring re-runs from the saved data, so the same input always gives the same ranking.
- If a proposal is extracted again, small AI differences are possible. The system compares the numbers, flags any change, and keeps the approved extraction unless a person accepts the change.
- Scoring is simple weighted maths (for example, price score = lowest eligible price ÷ this price × 100), so anyone can reproduce a score with a calculator.
- A reviewer sees a per-criterion score table for each proposal. Every value links to its source quote, the formula used and the one-line reason. Example: A scores 98.0 because it gets full marks on experience, timeline and team.

## 3. Human in the loop

Three checkpoints:
1. **After extraction:** the reviewer sees flagged items: missing values, unclear wording, quotes that don't match, instruction-like text.
2. **After ranking:** the reviewer confirms, re-admits, excludes or moves a proposal. Every override needs a written reason.
3. **Award:** always a person. The tool only recommends.

In the sample set, B is ineligible (the reviewer can see the failed check), C is held for a decision, and D is held until the bidder confirms a schedule.

**Audit log (append-only):** uploader, file fingerprint, extracted text, model and prompt versions, raw AI output, criteria version, scores, flags, each override (who, when, before, after, reason) and the final decision.

## 4. Changing criteria

- Criteria are versioned. An edit creates a new version and never overwrites the old one. The admin must confirm that weights total 100%.
- Every score stores the criteria version it used, so old results stay accurate.
- **Weight or threshold change:** re-score from saved extractions. No AI cost, and the result is instant and repeatable.
- **New or changed criterion that needs new facts:** re-extract only that criterion.
- The screen shows "scored under v1, v2 available" with before/after rank changes. A reviewer chooses whether to adopt v2. Awards already made keep their original version for the record.
- If a mandatory criterion changes, affected proposals are re-checked and any that now fail are clearly marked ineligible.

## 5. Risks and safeguards

| Risk | Safeguard |
|---|---|
| Wrong extraction | Every value needs a quote, and code checks the quote exists in the text. Invalid output is retried once, then sent to manual review. Unreadable or scanned PDFs are flagged. |
| Made-up facts | No quote means no value. Missing or vague details become "unclear", not guesses (D's schedule, B's subcontractor). |
| Manipulation (prompt injection) | The proposal is treated as data, never instructions. Instruction-like text (C) is ignored, reported and held for a person. The AI cannot set scores, so even a successful trick cannot change a rank. |
| Bias | Scoring uses only admin criteria. Company name, size and location are not part of scores. Same rules for everyone, and override patterns are reviewed regularly. Formats the AI reads poorly (scans, other languages) are flagged, not silently penalised. |
| Over-trust in AI | Output is labelled a recommendation. Humans own the award. |
| Cost | One extraction call per proposal, cached by file fingerprint. Re-scoring uses no AI. Limits on file size and pages. |
| Confidential data | Role-based access, only needed text sent to the AI, and only fictional data used in testing. |

## Assumptions

- "Last 5 years" is read as a start year of 2021 or later, because proposals state years, not dates. C ("since 2020") is flagged for this as well.
- Scoring scales (12 projects = full marks; warranty scales of 25 and 12 years) are my assumptions. In the real tool, the admin sets them.
- Price and timeline are scored against the best eligible bid. Ineligible bids do not set the baseline.
- I flagged C instead of ranking it, because a manipulation attempt is an integrity issue for a person to decide. Ranking it with a warning would also be defensible.
- PDFs are assumed to be text-based.
