# Prompthen Bench

An empirical benchmark measuring how **prompt engineering format**, **language**
(English vs Portuguese), **execution environment** (Direct API vs CLI Harness)
and **thinking configuration** affect the reasoning accuracy of AI models.

- **18,900 evaluated responses** — 36 complete runs of 525 cells each · **29 distinct models** · 6 providers
- Deterministic verification (exact match against answer key — no human judge, no LLM judge)
- Real cost ≈ $900 · one week of testing
- Test window: July–August 2026 (main window concluded 2026-07-22; two paused runs
  resumed as their weekly quotas renewed — GPT-5.3 Codex Spark on 2026-07-27 and
  Kimi K2.7 on 2026-08-11)

Full case study (in Portuguese):
**https://ficaadica.com.br/novidades/ntc-modelos-performam-melhor/**

The Prompthen Bench was created by **Paulo Teixeira**, a Brazilian AI
specialist — the same author behind **NTC**, a proprietary prompt engineering
he created, and the [Prompthen](https://prompthen.ai) method.
[About Paulo Teixeira](https://ficaadica.com.br/sobre/).

## The three test groups

Each run = 3 groups × 5 puzzles × 7 prompt formats × 5 repetitions = **525 cells**.

| Group | What it measures |
|---|---|
| **GRID** | Visual induction (ARC-AGI-2 style): infer a hidden rule from 3-4 grid→grid examples and apply it to a new input. Pure induction from minimal evidence. |
| **CASCADE** | Compositional reasoning: three chained transformations (Alpha→Beta→Gamma) where each output feeds the next. One early mistake destroys the final answer. |
| **VOLTGRID** | An **invented system** that does not exist anywhere on the internet: a documented spec with ONE hidden rule to infer from execution traces, then simulate. Designed to minimize training-data contamination (it did not exist before this study was published). The closest group to real-world engineering. |

Difficulty is a property of the puzzle, not the group: the hardest puzzle
(CASCADE/p2) was solved by **no model in 36 runs (0%)**; the easiest sits at ~84%.
The top overall score in the study is 77.5% — no model came close to the ceiling.

## The seven prompt formats

The same puzzles rendered in seven engineered prompt formats — numeric data byte-identical across all seven; instructions semantically equivalent per language:

| Format | Language | Included here |
|---|---|---|
| `md_en` / `md_pt` | Markdown, EN / PT | ✅ `prompts/` |
| `yaml_en` / `yaml_pt` | YAML, EN / PT | ✅ `prompts/` |
| `xml_en` / `xml_pt` | XML, EN / PT | ✅ `prompts/` |
| `ntc` | NTC (proprietary) | ❌ results only |

**NTC** is a proprietary prompt engineering created by **Paulo Teixeira, a
Brazilian AI specialist** — the same author of this benchmark. Its results and
token compression are published in full; the notation itself is not.

## Repository layout

```
prompts/
  GRID/p0..p4/{md,yaml,xml}_{en,pt}.txt      # 90 ready-to-use prompts
  CASCADE/p0..p4/...
  VOLTGRID/p0..p4/...
results/
  per_cell.csv       # 18,900 rows — every single cell: run × group × puzzle ×
                     #   format × language × repetition × correct (0/1)
  overall.csv        # general ranking (36 runs, 525 cells each)
  grid.csv           # ranking on the GRID group (175 cells)
  cascade.csv        # ranking on the CASCADE group (175 cells)
  voltgrid.csv       # ranking on the VOLTGRID group (175 cells)
  by_format.csv      # per-run × per-format accuracy (all 7 formats, NTC included)
  by_format_tokens.csv # accuracy × token cost per format (cl100k), globally and
                     #   per group, with the efficiency-frontier flag
  ablation.csv       # 2,100 rows — thinking on/off ablation for the two Lite
                     #   models (NOT part of the 36 ranked runs; see below)
```

`per_cell.csv` contains the full paired data: it supports McNemar tests,
paired bootstrap, or hierarchical models by anyone who wants to run their
own statistical analysis. A cell not answered by a model counts as 0
(official scoring rule: no answer = incorrect).

Answer keys are intentionally **not** published to keep the benchmark usable
for future model evaluations. Scoring for the published runs was fully
deterministic (exact match).

### About `ablation.csv`

The thinking finding (+41.7 pts) compares the **same model** with reasoning
enabled and disabled. The *enabled* runs are part of the 36 ranked runs and
live in `per_cell.csv`. The *disabled* runs are a configuration ablation, not
competitors — putting them in the ranking would list the same model twice for
a setting rather than a capability — so they are published separately here.

| Model | Thinking off | Thinking on | Delta |
|---|---:|---:|---:|
| Gemini 3.5 Flash-Lite | 6.5% (34/525) | 48.2% (253/525) | **+41.7** |
| Gemini 2.5 Flash-Lite | 0.0% (0/525) | 7.8% (41/525) | +7.8 |

Same 525 byte-identical prompts, same deterministic verification. The `on`
rows in `ablation.csv` reconcile 1:1 with the corresponding runs in
`per_cell.csv`. Reading: thinking amplifies latent capability, it does not
create it — the 2.5 Lite spent *more* reasoning tokens (10.9M vs 6.2M) and
recovered almost nothing.

## Headline findings

1. **Environment matters more than prompt format.** The same model, same
   prompts: Sonnet 5 scored 49.0% inside Claude Code vs 28.4% via Direct API
   (+20.6 pts). Opus 4.8: +8.4. No format choice in the study comes close to
   this effect. That is why rankings are published per environment.
2. **Thinking beats everything.** Gemini 3.5 Flash-Lite: 6.5% → 48.2%
   (+41.7 pts) just by enabling thinking — paired McNemar over 525 cells,
   219 discordant pairs to 0, p ≈ 2e-66. Ranked by what survives testing:
   thinking (>40 pts) > environment (8-21 pts) > language (1.2 pts aggregate,
   up to 12.9 on individual models) > format (no reliable separation at the top).
3. **There is no universal best format** — and, at the top, no measurable
   difference at all. NTC vs xml_pt: p = 0.83. NTC vs md_pt: p = 0.64.
   Per-family tendencies exist in the data (GPT toward Portuguese/Markdown,
   xAI toward XML, Moonshot toward YAML), but with 75 cells per format per run
   they are leads to test on *your* model family, not settled results.
4. **Portuguese beat English overall** on identical data: 48.56% vs
   47.48% across the md/yaml/xml pairs, ahead in all three engineerings
   (paired McNemar, p = 0.034). But the per-run scoreboard is essentially a
   tie: 17 runs favored PT, 17 favored EN, 2 drew. PT wins on aggregate
   because its gains are larger, not more frequent — the largest single case
   is Sonnet 5 at +12.9 pts (p = 0.0008). "Always prompt in English" did not
   survive the data.
5. **When accuracy ties, cost decides.** Prompt engineering has two metrics —
   getting it right and costing little — and this study only published the first
   until now. Measured with `cl100k`: md_en 473 tokens/prompt, ntc 507,
   yaml_pt 531, md_pt 537, xml_en 604, xml_pt 686. So xml_pt buys **+45% tokens**
   for an accuracy difference of 0.22 points against NTC — statistically nothing.
   Only two formats sit on the efficiency frontier: **ntc** (highest accuracy)
   and **md_en** (cheapest). Everything else is dominated. The effect is sharpest
   where instruction dominates: on VOLTGRID, ntc has the best accuracy (45.3%)
   at 444 tokens — 2 tokens above the cheapest, while xml_pt needs 50% more to
   score 2 points lower. Where it does not pay off: on GRID (85% raw grid data)
   ntc costs 594 tokens against md_en's 471 with no accuracy edge to justify it.
   Full numbers in `results/by_format_tokens.csv`.
6. **NTC** holds the top of the global format average (48.96% vs xml_pt at
   48.74% — six cells apart in 2,700, statistically indistinguishable). It
   also leads the per-run win count (9 of 36, vs 8 for xml_pt and 7 for
   xml_en and md_pt), but with seven formats the chance expectation is ~5 wins each, so
   that count separates nothing. Its distinctive property is compactness: the
   smallest of the seven on instruction-dense prompts (XML needs up to +67%
   more characters for the same content on VOLTGRID). Six of the seven formats
   fall within 0.85 point; only yaml_en (46.04%) breaks away, and that gap
   does hold up (p = 0.0004).
7. **Open-weight models arrived in force**: five open models within 4.3 points
   (Kimi K2.6 55.2 · K3 54.7 · DeepSeek v4-pro 53.0 · GLM 5.2 52.4 ·
   Qwen3.8-max 50.9), at the level of closed frontier models from ~6 months ago.
8. **Same family, different habitat.** Kimi was measured in both worlds: K2.6
   and K3 through the Direct API (55.2% and 54.7%), K2.7 inside its own CLI
   (48.2%). Two variables move at once there — model version and environment —
   so the gap is not a verdict on the model: it is a reading of how it performs
   in the tool people actually work in, which is what the CLI Harness runs are
   for. Same for Qwen3.8-max (qwen CLI) and MiniMax M3, the one model measured
   in both (30.3% API vs 24.0% Claude Code).

## Statistical caveat (read before ranking anything)

With **525 cells per run**, the 95% margin of error on an overall score is about
**±6 points**. That means the top of the leaderboard — GPT-5.6 sol (77.5%), terra
(73.0%), sol via API (71.2%) — is a **statistical tie**, and this dataset cannot
separate 1st from 2nd. It separates capability *bands* (77% vs 17% is beyond
argument), not neighbours.

Two more consequences worth stating plainly:

- **Max-minus-min is a noise-inflated statistic.** Simulating the null (format
  has no effect at all), the expected spread across 7 formats is ~14.7 points at
  75 cells and ~28 points at 25 cells. Observed spreads at or below that range
  are not evidence of a format effect.
- **Effects that do survive paired McNemar** on this data: thinking (p≈2e-66),
  environment on Sonnet 5 (p<1e-6) and Opus 4.8 (p=3e-5), language on Sonnet 5
  (p=0.0008), language in aggregate (p=0.019), and yaml_en trailing the rest
  (p=0.0003). Format differences at the top do not (p=1.00 for NTC vs xml_pt).

None of this is unique to this benchmark — a public benchmark with 120 or 500
tasks carries a larger margin, and most never publish the number. The difference
here is that `per_cell.csv` lets you verify every claim, including the ones that
go against the author.

## Reproducing / citing

Every number in the case study can be recomputed from `results/per_cell.csv`
(sum `correct` by run, group or format). To cite:

> Teixeira, Paulo (Brazil). *Prompthen Bench: 18,900 evaluated AI responses,
> 29 models, 7 prompt engineerings, 2 languages, deterministic verification.*
> Fica a Dica, August 2026.
> https://ficaadica.com.br/novidades/ntc-modelos-performam-melhor/

## License

Prompts and result data: **CC BY 4.0** (see [`LICENSE`](LICENSE)) — free to use,
share and adapt, including commercially, with attribution.

Copyright (c) 2026 Paulo Teixeira — https://ficaadica.com.br/sobre/

**Scope:** the license covers `prompts/` and `results/`. It does **not** cover the
NTC prompt engineering notation itself, which is proprietary and is not published
in this repository — only its results and its measured token cost are.

**Answer keys** are also withheld, so the benchmark stays usable against future
models.
