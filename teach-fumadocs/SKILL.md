---
name: teach-fumadocs
description: Teach a topic via a structured Fumadocs site with real-world examples and human prose.
disable-model-invocation: true
argument-hint: "What would you like to learn?"
---

The user has asked you to teach them something. This is a stateful request — they intend to learn the topic over multiple sessions. You deliver the teaching as a **Fumadocs site** in `docs/` at the project root, not as standalone `*.html` files. Every lesson is a `.mdx` page with runnable code examples and real-world analogies. Every sentence passes the unslop filter.

## Teaching workspace

Treat the current directory as a teaching workspace. State lives in two places:

**Agent state (project root, not rendered):**

- `MISSION.md` — the *reason* the user is learning. Ground for every decision. Format in [MISSION-FORMAT.md](./MISSION-FORMAT.md).
- `RESOURCES.md` — curated high-trust sources. Format in [RESOURCES-FORMAT.md](./RESOURCES-FORMAT.md).
- `learning-records/*.md` — decision-grade insights, `0001-slug.md` incrementing. Format in [LEARNING-RECORD-FORMAT.md](./LEARNING-RECORD-FORMAT.md).
- `NOTES.md` — scratchpad for user preferences and working notes.
- `reference/glossary.md` or `docs/content/docs/reference/glossary.mdx` (or `docs/content/reference/glossary.mdx`) — canonical language. Format in [GLOSSARY-FORMAT.md](./GLOSSARY-FORMAT.md).

**Published content (rendered by Fumadocs, inside `docs/`):**

- `docs/content/docs/<topic>/0001-<dash-case>.mdx` — **lesson**. One self-contained MDX page per lesson, titled with incrementing number. The primary unit of teaching.
- `docs/content/docs/reference/*.mdx` — **reference**. Compressed cheat sheets, syntax, algorithms, glossaries for quick lookup. Lessons link here.
- `docs/components/*` — **reusable components** shared across lessons (callouts, quiz widgets, simulators, diagram helpers). See [Assets](#assets).

Detect content directory on each run: prefer `docs/content/docs/` (standard Fumadocs MDX convention), or `docs/content/` if the project uses that layout.

## Philosophy

To learn deeply the user needs three things:

- **Knowledge**, captured from high-quality, high-trust resources.
- **Skills**, acquired through relevant interactive lessons you devise from that knowledge.
- **Wisdom**, which comes from interacting with other learners and practitioners.

Before `RESOURCES.md` is well-populated, your focus is finding high-quality resources. Never trust parametric knowledge alone.

Some topics need more skills than knowledge. Theoretical physics leans knowledge; yoga leans skills.

### Fluency vs storage strength

Split two types of learning:

- **Fluency strength**: in-the-moment retrieval.
- **Storage strength**: long-term retention. This is the real goal.

Fluency gives an illusory sense of mastery. Design lessons for storage strength via desirable difficulty:

- Retrieval practice (recall from memory)
- Spacing (distribute practice over time)
- Interleaving (mix related topics in practice — for skills only)

## Workflow — do these in order

### 1. Ground the `mission`

Read `MISSION.md`. If missing, empty, or vague, interview the user on *why* they want to learn this before writing any lesson. Every lesson must trace back to the mission.

- Push for concrete outcomes: "Ship a Rust CLI to my team" beats "learn Rust".
- Record constraints (time, budget, preferences) and out-of-scope topics.

*Done when* `MISSION.md` exists, follows [MISSION-FORMAT.md](./MISSION-FORMAT.md), and user confirms it. If the mission shifts later, update the file and add a learning record cross-linked to it.

### 2. Validate `docs/` is a Fumadocs app — guard (do not create it)

This skill never creates or sets up Fumadocs itself. All teaching content goes in `docs/` **only if** a Fumadocs app already exists there.

Check on every run before writing any lesson or reference:

```
Test-Path docs/package.json
  AND Select-String "fumadocs" docs/package.json  (fumadocs-core, fumadocs-ui, or fumadocs-mdx)
  AND ( Test-Path docs/source.config.ts  OR  Test-Path docs/source.config.js  OR  Test-Path docs/source.config.mjs  OR  Test-Path docs/lib/source.ts  OR  Test-Path docs/src/lib/source.ts  OR  Test-Path docs/next.config.mjs  OR  Test-Path docs/next.config.ts  OR  Test-Path docs/next.config.js )
  AND ( Test-Path docs/content/docs  OR  Test-Path docs/content )
```

- If any check fails: **stop**. Tell the user:

  > No Fumadocs app found in `docs/`. Set up a Fumadocs project in `docs/` first (e.g. `npm create fumadocs-app` or follow https://fumadocs.dev/docs), then re-run this skill. I will not create or set it up for you.

  Do not create `docs/`, do not write `package.json`, `source.config.*`, or `next.config.*`.

*Done when* guard passes, or user has been told to set up and you have stopped.

Detect layout once guard passes: if `docs/content/docs/` exists use it; else if `docs/content/` exists use it; else default to `docs/content/docs/` and note it in the lesson. Adapt `meta.json` updates accordingly.

### 3. Inventory `resources`

Read `RESOURCES.md` and `reference/` or `docs/content/docs/reference/` (or `docs/content/reference/`). Knowledge for lessons must be drawn from resources listed there, not parametric guesses. Annotate every entry with what it covers and when to reach for it.

*Done when* `RESOURCES.md` has at least two high-trust Knowledge entries relevant to the next lesson, or an explicit `## Gaps` section noting what is missing. If shallow or marketing-dressed sources are present, prune them.

### 4. Find the zone of proximal development

Each lesson must be "just hard enough".

Figure it out by:

- Reading `learning-records/*.md` for what the user already demonstrated.
- Reading `NOTES.md` for preferences.
- Mapping the mission to the next most relevant, tightly scoped win.

If the user named an exact thing, teach that (still within mission). Otherwise pick the smallest next step that yields a tangible win completable in <12 minutes.

*Done when* you can state in one sentence what the next lesson will teach and why it is the correct next step.

### 5. Author the `lesson` as MDX in `docs/`

One lesson = one `docs/content/docs/<topic>/NNNN-<dash-case>.mdx` (or `docs/content/...`). Increment NNNN from the highest existing number across that topic.

Each lesson must:

- Be self-contained and completable quickly. Clean, readable typography via Fumadocs UI — think Tufte, not marketing.
- Open with a **real-world analogy** before formal terms. Example: token bucket as a day-pass pool, rate limiter as a nightclub bouncer.
- Teach **knowledge first**, then make it stick with a skill move. Knowledge = the minimum required to acquire the skill.
- Contain at least one **runnable code example** (or executable pseudo-code) with line-by-line explanation mapped to the real-world analogy. For non-code topics, use an equivalent concrete worked example grounded in a real scenario. Never present abstract theory without a concrete instance the user can run, copy, or act on.
- Cite sources inline — every non-obvious claim links to `RESOURCES.md` or the primary source.
- Include a **feedback loop**: a quiz, a small in-browser task, or a set of real-world steps with immediate feedback. For quizzes, each answer must be the same length (words and characters if possible) so formatting gives no clue.
- End with: link to other lessons (`meta.json` order), link to reference docs, a reminder to ask follow-ups, and a recommended primary source to read/watch.
- Link via MDX anchors to other lessons and reference docs.
- Update `meta.json` in the topic folder (e.g. `docs/content/docs/<topic>/meta.json`) to include the new page slug in the `pages` array for sidebar order.

Reuse is the default. Before authoring, read `docs/components/`. Build from what is already there or Fumadocs UI built-ins (Callout, Tabs, Steps, Accordion, Cards). When a new reusable piece is needed, write it as a component in `docs/components/` and import it in MDX; never inline code a future lesson would duplicate. A shared quiz or simulator component is the first thing every site earns.

*Done when* the `.mdx` file exists, has valid frontmatter (`title`, `description`), renders without MDX errors, is listed in `meta.json`, and contains at least one real-world analogy + one explained code/concrete example + one feedback loop.

### 6. Update `reference` as MDX

While creating lessons, also create compressed reference docs in `docs/content/docs/reference/*.mdx` (or `docs/content/reference/*.mdx`). These are the raw units of learning — cheat sheets, syntax, algorithms, glossaries.

- Lessons are rarely revisited; reference is. Keep it tight and scannable.
- Glossaries are essential. Once a term is defined, adhere to it in every lesson.

*Done when* any new concept introduced in the lesson that warrants quick lookup has a corresponding entry in reference, and lessons link to it.

### 7. Apply the `unslop` pass — every time you write prose

Removing patterns is half the job; voiceless writing is just as obvious. After drafting any MDX, run this pass before saving:

**Process:** 1) Scan for patterns below. 2) Rewrite, preserve meaning, match intended tone. 3) Add soul (next). 4) Self-audit: "What makes this obviously AI generated?" Fix remains.

**Adding soul:**

- Have opinions. React to facts instead of neutral pros/cons.
- Vary rhythm. Short sentences. Then longer ones that take their time.
- Acknowledge complexity. "Impressive but also kind of unsettling" beats "impressive".
- Use "I" when it fits.
- Let some mess in. Perfect structure looks machine-made.
- Be specific. Not "this is concerning" but "there is something unsettling about agents churning at 3am".

**Patterns to detect and fix (full list — check every line):**

Content: puffery ("pivotal moment", "testament to", "evolving landscape", "indelible mark"), name-dropping without context, superficial -ing phrases ("highlighting...", "ensuring..."), promotional language ("nestled", "breathtaking", "groundbreaking"), vague attributions ("Experts believe"), formulaic challenges ("Despite challenges... continues to thrive").

Language: AI vocabulary (Additionally, crucial, delve, enduring, enhance, fostering, garner, interplay, intricate, landscape-abstract, pivotal, showcase, tapestry-abstract, testament, underscore, vibrant), fancy "is" ("serves as", "boasts"), "Not just X, but Y", rule of three forcing, synonym cycling (pick one term, repeat it), false ranges ("from X to Y" without a real scale).

Style: em dash overuse — use periods or commas only, no em dashes, no parentheses-as-dash; colon overuse as mid-sentence connector; boldface overuse; inline-header lists ("**Performance:** Performance improved..." → prose); title case headings → sentence case; decorative emojis; curly quotes → straight quotes.

Communication: chatbot phrases ("I hope this helps!", "Let me know if..."), cutoff disclaimers, sycophantic tone ("Great question!").

Filler: "In order to" → "To", "Due to the fact that" → "Because", "It is important to note that" → delete; excessive hedging → "may"; generic conclusions → specific facts.

Jargon: abstract metaphor nouns — substrate, wedge, vector, locus, vantage, nexus, primitive (noun), harness (metaphor), surface (API surface), bedrock, scaffolding (metaphor), modality, paradigm, gold-plating, ratchet (metaphor), evacuate (for code), endgame, north star, flywheel → plain concrete word.

Plain speech: say what it does, not how it feels ("the database stays close at hand" → "`.toSQL()` returns the exact string sent to the database"); shorten or split dense sentences to one idea each; active voice (name the actor, "the compiler validates queries"); cut adverbs or use measured numbers; prefer the plain word (utilize→use, leverage→use, facilitate→help, numerous→many, in the event that→if).

*Done when* no pattern above remains, headings are sentence case, prose is active and specific, and a second read does not feel machine-made.

### 8. Record learning and preferences

- `NOTES.md`: when the user expresses how they want to be taught, record it.
- `learning-records/NNNN-slug.md`: write one when the user demonstrates genuine understanding, discloses prior knowledge, corrects a misconception, or the mission shifts. Not for mere coverage. Format in [LEARNING-RECORD-FORMAT.md](./LEARNING-RECORD-FORMAT.md). Scan for highest number and increment.

When a later record contradicts an earlier one, mark the old `Status: superseded by LR-NNNN` rather than deleting it.

*Done when* any new insight from this session is captured, or you note explicitly that no record was warranted.

## Lessons

A lesson is the main thing you produce. Each is one MDX file in `docs/content/docs/<topic>/` (or `docs/content/<topic>/`).

- Beautiful and scannable via Fumadocs UI. Short, <12 min, one tangible win tied to the mission, in the user's zone of proximal development.
- Links to other lessons and reference via MDX links and `meta.json`.
- Recommends a primary source.
- Reminds the user to ask follow-ups.

## Assets

Lessons are built from Fumadocs UI components and reusable custom **components** in `docs/components/`: callouts, quiz widgets, simulators, diagram helpers.

Reuse before inventing. Fumadocs provides built-in components (Callout, Card, Cards, Tabs, Steps, Accordion); lean on those first. A shared MDX component is the first custom asset every site earns so the course looks like one course, not a pile of one-offs.

## The mission

Every lesson traces to the mission. If `MISSION.md` is not populated, question the user on why they want to learn this before anything else. A bad mission is worse than no mission. One mission per workspace; revise when reality shifts and confirm with the user.

## Reference documents

Lessons reference `docs/content/docs/reference/*.mdx` (or `docs/content/reference/*.mdx`). Keep them compressed — syntax, algorithms, flowcharts, glossaries. Once `glossary.mdx` exists, every lesson must use its terms.

## Notes on this skill

- This skill combines `teach` (stateful, incremental, citation-backed) with `unslop` (human prose). It inverts only the output format: MDX in a Fumadocs `docs/` app instead of standalone `lessons/*.html` + `reference/*.html`.
- If you also need the original HTML mode, invoke the `teach` skill directly. Do not mix both outputs for one topic.
