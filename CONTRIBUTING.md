# Contributing

Thanks for helping improve this set of worked AI system design answers. Contributions are welcome. They need to clear the quality bar below, because readers use these answers to prepare for real interviews.

Maintained by [Landed](https://landed.jobs). No affiliation with any company named in this repo.

---

## What we accept

- **New designs.** A prompt not covered yet, written to the template below. Open an issue first so two people don't write the same one.
- **Corrections.** Wrong math, wrong mechanism, outdated hardware or pricing facts, broken links.
- **Sharper answers.** Better trade-off tables, failure modes you have seen in production, follow-up questions you were asked.
- **Sources.** Papers, official docs and engineering blog posts that explain a design decision better than what we link today.

## The non-negotiables

### 1. Every number shows its work

Estimates are labeled as estimates, with the assumptions written out step by step. Hardware specs come from vendor datasheets. Prices are given as ranges with a "check current pricing" note. A number with no derivation will be sent back.

### 2. Honest provenance

Interview prompts and follow-ups are **Representative** unless a public source shows a company asked them. If you add a reported question, link the public source. "Companies that ask this" lists companies whose products center on the system; it does not claim they ask the exact prompt.

### 3. Every link resolves

External links go to papers, official docs, engineering blogs or well-known repos. No link shorteners, no paywalled sources where a free one exists. Check each link before you open the PR.

### 4. License discipline

This repo is **MIT** (see [`LICENSE`](LICENSE)).

| Source license | What you may do |
|---|---|
| MIT, Apache-2.0, CC0 | Adapt text or code, with attribution |
| CC-BY-4.0 | Link and comment only. Do not copy verbatim |
| CC-BY-SA | Link and comment only. Share-alike would relicense this repo |
| Unknown or proprietary | Link only, if publicly readable |

### 5. Style

- Plain words, short sentences, active voice. Sentence-case headings.
- Tables for anything you compare.
- Take a position. "Use hybrid retrieval by default" beats "there are pros and cons".
- Mermaid diagrams must render on GitHub. Quote every label that has spaces or punctuation, for example `A["Rerank (cross-encoder)"]`, and do not name a node `end`.

## Design template

Every file in [`designs/`](designs/) uses these sections, in this order:

1. Title, nav line, difficulty, key trade-off, round length
2. Problem statement
3. Clarifying questions (table: question, why it matters, assumption)
4. Requirements (functional, non-functional with numbers)
5. Back-of-envelope estimates, step by step
6. Architecture (one mermaid diagram)
7. Component walkthrough
8. Data and eval plan
9. Trade-offs (table: decision, option A, option B, when to pick which)
10. Failure modes and mitigations
11. Scaling to 10x
12. What interviewers probe
13. Companies that ask this
14. Further reading (2 to 5 sources)

Copy [`designs/01-rag-over-100m-documents.md`](designs/01-rag-over-100m-documents.md) as your starting point.

## How to contribute

1. Open an issue with a [template](.github/ISSUE_TEMPLATE/), or go straight to a PR for small fixes.
2. Fork, branch, edit.
3. Keep internal links relative (`designs/06-llm-inference-serving.md`). CI runs [lychee](.github/workflows/links.yml) on local links for every PR.
4. Fill in the [PR checklist](.github/PULL_REQUEST_TEMPLATE.md).

## Recognition

Contributors are credited in release notes.
