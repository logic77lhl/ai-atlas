[中文](README.md) | **English**

# AI Atlas · a map of the AI stack

One line: **no lessons, just a map.** Every concept answers four questions — where it came from, how it's used now, whether it's still in use, and what replaced it.

**Live**: https://logic77lhl.github.io/ai-atlas/ — 59 concepts, 9 domains, 116 deep links, two views (map / chronicle), a single dependency-free file that works offline. Progress lives in your browser; private Gist sync is optional.

## Two views of the same data

- **Map** — layered by domain; answers "which layer is this on, what does it depend on".
- **Chronicle** — ordered by year (1986–2024); answers "when did this arrive, what did it look like then".

The year is when a concept took shape, not its status today. A `~` prefix means an estimate — engineering practices have no single paper to cite, so the year is the best you can do.

## Five things you can do

1. **Read the dependency graph** — 6 layers, top to bottom answers "who depends on whom" (collapsible).
2. **Switch views** — map for structure, chronicle for time.
3. **Filter by status** — standard / mainstream / transitional / declining / historical / contested / not learned.
4. **Open a concept** — predecessor, present use, limits, deep links.
5. **Mark it** — want to learn / understood / learned.

Search matches Chinese name, English name, the three body lines, and the year.

## What this is not

- **Not a tutorial.** No derivations, no re-typed lecture notes.
- **Not a link dump.** Every link has to answer a specific question.
- **Not an extension of hf-quest.** Separate project, separate progress. hf-quest is referenced only as the "tooling track".

## What this is

- 9 domains, 59 concepts (hard cap: 60)
- 6-layer dependency graph: foundations → models → training → inference/retrieval → agents/evaluation → engineering/limits
- Chronicle view: 7 periods, each stating what that period was trying to solve
- 4 lines per concept: predecessor / now / status / limits, each with a "last checked" month
- Deep links to papers, GitHub implementations, official docs
- Progress is only "want / understood / learned" — no lesson counters
- Status filter, domain jump, full-text search, export / import / clear
- Private Gist sync: last-write-wins per concept, conflicts never overwrite silently

## Gist sync

1. GitHub → Settings → Developer settings → Personal access tokens → **Tokens (classic)** → check only `gist`
2. Click "sync" in the footer → paste the token → "create new archive" → you get a Gist ID
3. On another machine, open the same URL, paste the same token + Gist ID → "sync now"

The token stays in that machine's browser storage; it is never written into the page or the export file. Marks auto-push 2.5 s after you change them, and pull on load. "Clear" in the same dialog removes it.

## Status labels

| Label | Meaning | What a new project should do | Count |
|---|---|---|---|
| **standard** | the de-facto default | just use it | 28 |
| **mainstream** | widely used, has clear alternatives | evaluate the alternatives first | 16 |
| **transitional** | being replaced | be careful in new work | 1 |
| **declining** | maintenance mode | avoid unless legacy | 0 |
| **historical** | for understanding the lineage | know it, don't build on it | 3 |
| **contested** | no consensus in practice | depends on context; distrust single conclusions | 11 |

## Data model

The data lives in the `DATA` array inside `index.html`. One entry:

```js
{ id:"kvcache", zh:"KV 缓存", en:"KV cache", d:"inference", s:"standard", v:"2026-10",
  b:"every generated token required recomputing the whole history",  // predecessor
  n:"cache K/V, trade memory for speed",                            // now
  e:"long contexts blow up memory — that's why GQA / quantized KV", // limits
  l:[["PagedAttention","https://arxiv.org/abs/2309.06180"]] }
```

Years live in a separate `YEAR` table (`id -> "2017-06"`, `~` prefix = estimate) and are joined at load time, so changing a year doesn't touch the body copy.

`CONCEPTS.md` is generated from this data, so the two never diverge. **Edit only `index.html`**, then re-run the generator.

## Files

| File | Contents |
|---|---|
| `index.html` | the app and the single source of data |
| `BRIEF.md` | requirements, research, decisions, risks, milestones |
| `CONCEPTS.md` | all 59 concepts in full (generated, for reading and review) |
| `README.md` / `README.en.md` | what this is and what's next |

## Known limits

- This machine can't reach `huggingface.co`, `lmarena.ai`, `platform.openai.com`, `copyright.gov`, `docs.anthropic.com`, so deep links avoid those domains and use papers / GitHub / reachable official docs instead.
- Status is a judgement, not a fact. Every entry carries a date; "grey out when stale" isn't built yet.
- Years are judgements too. Paper-backed concepts are accurate to the month; engineering practices are accurate to the year (13 entries carry `~`).
- The timeline stops at 2024. Nothing for 2025–2026 doesn't mean nothing changed — it means the changes haven't settled into stable concepts.
- Gist sync overwrites the whole archive and keeps no history. If two machines edit the same concept at once, the newer timestamp wins and the other edit is lost.
- Speech, recommender systems, RL theory, and world models are deliberately excluded to keep it to one screen.

## Relationship to hf-quest

Separate repo, separate progress, separate deployment. They don't merge.

The 42 hf-quest lessons are positioned here as the "tooling track: using and deploying models", covering the `inference` and `engineering` domains.

## License

[MIT](LICENSE)
