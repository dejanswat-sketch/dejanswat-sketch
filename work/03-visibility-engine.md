# Case study 3 — AI Visibility Engine: measuring what an AI can actually read, and what it answers

**Type:** case study, own product · **Status:** working prototype, measured ·
**Disclosure:** built by Dejan Milošević; AI agents wrote part of the code under his direction and
review.

## The problem

"AI visibility" is usually sold as a promise. Three questions have checkable answers: what can a
crawler read from a domain, what will an assistant actually answer when a buyer asks, and did
anything change after the work was done. Almost nothing on the market separates those three.

## The solution

Four parts that share one rule — **evidence before claim**:

1. **Discovery (module 01):** reads robots.txt with a real parser (longest-match rules), sitemaps
   including indexes, and up to N pages; records for every fetch: status, time, bytes, sha256,
   redirect chain, content type.
2. **Capability graph (modules 11–13):** a professional profile whose capabilities are only
   "demonstrated" when an artifact answers and authorship is established; an audit classifies every
   repository and local program with an evidence level.
3. **AI answer lab (module 06):** asks real platforms the same buyer questions, and scores four
   separate things — found, identity accuracy, citation, recommendation.
4. **Deployment pipeline:** the site that publishes the results is verified on every deploy.

## Architecture

```
domain ──▶ discovery ──▶ evidence.jsonl (one line per observation)
profile + git history ──▶ audit ──▶ capability graph ──▶ matching
queries ──▶ platform adapters ──▶ answers ──▶ scoring ──▶ report + comparison
```

Storage is files: JSONL evidence, JSON runs, Markdown reports. No database, no server, no
dependencies beyond Node.

## Technologies

Node.js, HTTP measurement, Schema.org, headless Chrome (CDP), platform APIs (Perplexity sonar,
Gemini with search grounding, OpenAI and Anthropic chat as no-search baselines).

## The role

Product creator, system architect, and the person who defined the invariant that no capability may
be called demonstrated without proof. Agents wrote a large part of the implementation; every rule in
the test suite was specified by the author.

## Demonstration

```bash
node engine.mjs https://client-domain.com     # discovery
node recognition.mjs build                    # capability graph from evidence
node match.mjs demo                           # twelve scenarios, 12/12
node lab/run.mjs --platforms perplexity,gemini --langs en,sr
node lab/compare.mjs <baseline> <after>
```

## Verifiable results

- **44 automated tests**, four suites, zero failures, including the invariants: no demonstration
  without proof, no match from a single shared word, absence recorded as absence.
- **161 projects audited** (31 repositories + 130 local programs) with authorship and functionality
  levels; secrets checked and never printed.
- **16 real AI answers** across four platforms and two languages, stored with the exact prompt,
  timestamp, sources and score — and a control run proving the measurement is stable (0 changes).
- A production defect found and fixed on a live site because the measurement contradicted the
  assumption (pages served as `application/octet-stream`).

## Limits

- The sample is small: six questions, four platforms. It is a baseline, not a census.
- Platform answers depend on model version, locale and the day; every observation is dated for that
  reason.
- No opportunity discovery and no outcome tracking yet: the engine can prove visibility, not that a
  recommendation produced a conversation.
- One provider profile: matching is a demonstrated mechanism, not a marketplace.
