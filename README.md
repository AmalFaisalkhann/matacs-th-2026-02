# MATACS-TH-2026-02: AI Proposal Triage Design

**Name:** Amal Khan
**Reference:** MATACS-TH-2026-02
**Time actually spent:** About 40 minutes

## What's in this repo

- [WRITEUP.md](WRITEUP.md): my answers to the five questions
- [PROMPTS.md](PROMPTS.md): the two AI prompts (extraction and one-line reason), written the way I'd send them
- [ranking.json](ranking.json): my expected ranking of proposals A to E
- [diagram/process-flow.png](diagram/process-flow.png): the flow from upload to ranked output, with AI steps, code steps and human checkpoints marked in different colours

There is no prototype. I focused on the design.

## My approach in short

The idea I kept coming back to is: **AI reads, code decides, a person awards.**

The AI only pulls facts out of the messy proposal text (and has to give a quote as proof for each one), then later turns the finished numbers into a one-line reason. Everything else, like the pass/fail checks, the scoring maths and the sorting, is plain code, so the same input always gives the same ranking. A person reviews the list and makes the final award.

How the sample set came out:
- **A** is first and **E** is second.
- **B** is ineligible. The electrical work is subcontracted and not confirmed, so the bidder doesn't actually hold the licence. Its low price never gets looked at.
- **C** is flagged. It has hidden text trying to tell the reviewer to rank it first. I treated that as data, not an instruction, and left the decision to a person.
- **D** is flagged. It gives no fixed completion time, and I didn't want the tool to guess one.

The assumptions I made (like how I read "last 5 years" and the scoring scales) are listed at the end of WRITEUP.md.

## How I used AI

I used Claude for most of the first drafts: the prompts, the scoring rules, the write-up and the diagram. I worked through it one step at a time and committed after each step. I'm still learning this kind of design work, so I used the AI to get a structure I could then read through, understand and explain.

**Where I corrected the AI's output:** Claude's first answer gave rough score estimates before there was any written scoring method (about 87 for E and about 72 for D). Once the actual rubric was written out, the calculated scores came out a little different (87.7 for E, and 73.75 out of 85 available points for D). I used the calculated numbers in `ranking.json` and not the early guesses, because a score should come from a method someone can re-check, not from an estimate.

**A judgment call I want to flag:** Claude noted that proposal C could be either ranked with a warning or held back as flagged. I went with flagged, because trying to manipulate the review looks like an integrity problem that a person should decide on. I know ranking it with a warning is also a fair choice.
