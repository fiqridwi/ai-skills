---
name: summarize
description: Summarizes large context (long files, folders, PDFs, slide decks, URLs, PR threads, pasted text, or the current conversation) into a layered summary that builds understanding — TL;DR, big picture, key points with source locators, glossary, open questions, and ASCII flow diagrams (Mermaid with --mermaid) for any process, status lifecycle, or system sequence found in the source. Uses direct reading for small inputs and map-reduce with parallel sub-agents for inputs too large to read at once. Use when the user runs /summarize or asks to summarize, recap, or explain a large body of material.
when_to_use: 'User says "rangkum", "ringkas", "summary", "jelaskan isi dokumen ini", "aku belum paham konteksnya", "TL;DR", or points at a big file/folder/PDF/URL and wants to understand it fast.'
argument-hint: <path | dir | url | "chat"> [--depth tldr|standard|deep] [--focus "<pertanyaan>"] [--lang id|en] [--mermaid]
---

# Summarize: understand a large context fast

Input: $ARGUMENTS
(If empty: summarize what the user's latest message points at; if nothing, summarize the current conversation.)

Goal is **understanding**, not shortening. The reader should finish knowing: what this is, why it exists, how its parts connect, what was decided, and what is still unclear.

## Defaults
- `--depth standard` (TL;DR + big picture + key points + glossary + open questions). `tldr` = 3–5 lines only. `deep` = add per-section breakdown and structure diagram.
- `--focus`: if given, every section answers through that lens; drop material irrelevant to it and say so in one line.
- `--lang`: match the user's language (default Bahasa Indonesia). Keep technical terms, identifiers, and quotes in their original language.
- `--mermaid`: off by default. Flow diagrams are ASCII only; with the flag, add a Mermaid version under each ASCII diagram.

## 1. Locate and size the source
| Source | How to read |
|---|---|
| Text/code file | `wc -l -c` first, then Read |
| Directory | `find` + `wc` to list files and sizes; skip vendored/generated dirs (node_modules, dist, .git, lockfiles) |
| PDF | `pdftotext -layout file.pdf -` (or Read with `pages`). If text looks garbled (dropped `fi`/`ff` ligatures, split table cells), say so and spot-check those pages with Read `pages` |
| .docx/.pptx/.xlsx | use the matching `anthropic-skills:*` skill to extract text |
| URL | WebFetch (ask for full content, not a summary); PR/issue → `gh` |
| `chat` | the conversation so far |

Estimate size: ~4 chars ≈ 1 token. Then pick the strategy.

## 2. Pick the strategy
| Size | Strategy |
|---|---|
| < ~40k tokens (~1,500 lines) | **Direct**: read it all yourself |
| ~40k–150k tokens, or 2–10 files | **Sequential map**: read in chunks yourself, write chunk notes to the scratchpad as you go |
| > ~150k tokens, or many files/sources | **Parallel map-reduce**: split into 3–6 chunks along natural seams (chapters, dirs, files), one `Explore`/`general-purpose` sub-agent per chunk, sent in one message, using [chunk-brief.md](chunk-brief.md) |

Split on **natural boundaries** (headings, sections, files, slides), never mid-section. Write notes to the scratchpad directory (`summarize-notes.md`) so they survive context compaction.

## 3. Map: extract notes per chunk
For each chunk capture, with a **locator** (`file:line`, page, slide, heading, or URL anchor) on every item:
- Snapshot date, if the source is dated ("kondisi per…", "pagi ini", "besok")
- Purpose / main claim of the chunk
- Key facts, numbers, dates, names, decisions
- Entities and terms (with a one-line definition)
- Relations to other parts ("depends on", "contradicts", "replaces")
- **Flows**: ordered steps, status transitions, approval chains, system-to-system calls, decision rules. Record them as `A → B (trigger/condition)` edges with locators.
- Open questions, TODOs, ambiguities, contradictions

Notes are extractive: paraphrase faithfully, quote when wording matters. No interpretation yet.

## 4. Reduce: merge into one picture
1. Merge chunk notes; dedupe entities; unify terms (pick one name per concept).
2. Find the spine: the 3–7 themes everything else hangs on.
3. Surface **conflicts across chunks** explicitly — never silently pick one side.
4. Order by importance to understanding, not by source order.
5. **Flows**: if any chunk recorded flow edges, stitch the edges of each flow together across chunks and diagram every flow. This runs at every depth except `tldr`. Read [flow-diagrams.md](flow-diagrams.md) for detection signals, diagram type, rendering, and fidelity rules.

## 5. Densify (Chain-of-Density pass)
Rewrite the TL;DR and key points 2 times: each pass, add the most important missing entity/fact while keeping length the same, removing filler ("dokumen ini membahas…"). Stop when adding more would hurt readability.

## 6. Verify before output
- Every key point has a locator. Spot-check 3–5 of them by re-reading the source.
- No claim that isn't in the source. Your own inference is labeled **(inferensi)**.
- Numbers and names copied exactly.
- If part of the source was skipped or unreadable, say which part.
- Diagrams: every node and edge traces to the source; inferred edges are dashed and listed under "Belum jelas".
- **Order is a claim.** If the source lists conditions/steps but not their evaluation order, the order you draw is inference: say so.
- **Gap claims are inference.** "Nobody owns X", "not handled", "no follow-up" need **(inferensi)** unless the source says it.
- **Contradictions must survive a charitable read.** Quote both sides; if one reading reconciles them (e.g. "berhenti diam-diam" per tick vs "ditunda" overall), it is not a contradiction: drop it or list it as wording ambiguity.
- **Time check.** If the source is a dated snapshot, state its date vs today and flag scheduled events that are now in the past (status unknown).

## 7. Output template
Adapt headings to the material; omit empty sections. Length budget at `standard`: ≤ ~20% of the source or ~1,200 words, whichever is smaller; cut detail before cutting caveats.

```markdown
## TL;DR
<3–5 kalimat: apa ini, untuk apa, kesimpulan utama>

## Gambaran besar
<1 paragraf: konteks, siapa terlibat, masalah yang diselesaikan, bagaimana bagian-bagian terhubung>

## Poin kunci
1. **<tema>** — <penjelasan singkat> (`locator`)
2. ...

## Keputusan & angka penting      <!-- jika ada -->
| Item | Nilai/keputusan | Sumber |

## Istilah
| Istilah | Arti di konteks ini |

## Alur                            <!-- wajib jika sumber punya flow; lihat flow-diagrams.md -->
### <nama alur>
<diagram ASCII; tambah Mermaid dalam <details> hanya jika --mermaid>
<1–3 baris: pemicu, penggerak, titik akhir> (`locator`)

## Struktur                        <!-- depth=deep: hierarki/komponen, bukan urutan -->
<daftar bertingkat>

## Belum jelas / kontradiksi
- <pertanyaan terbuka, celah, konflik antar bagian> (`locator`)

## Baca lebih lanjut
- <2–3 bagian sumber yang paling layak dibaca utuh, dan kenapa>
```

End with one line offering follow-ups: drill into a section, a different `--focus`, or a shareable page (Artifact) if the summary is for other people.

## Anti-patterns
- Summarizing in source order like a table of contents ("Bab 1 membahas…, Bab 2 membahas…").
- Dropping numbers, names, or caveats for smoothness.
- Hiding uncertainty; merging contradictory statements into one.
- Spawning sub-agents for something that fits in one read.
- Pasting the full chunk notes into the final answer.
- Dashed edges for paths the source states (failure/retry paths are solid, labelled).
