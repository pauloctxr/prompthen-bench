# Prompthen Bench

An empirical benchmark measuring how **prompt engineering format**, **language**
(English vs Portuguese), **execution environment** (Direct API vs CLI Harness)
and **thinking configuration** affect the reasoning accuracy of AI models.

- **17,850 evaluated responses** — 34 complete runs of 525 cells each · **29 distinct models** · 6 providers
- Deterministic verification (exact match against answer key — no human judge, no LLM judge)
- Real cost ≈ $900 · one week of testing
- Test window: July 2026 (concluded 2026-07-22)

Full case study (in Portuguese):
**https://ficaadica.com.br/novidades/ntc-modelos-performam-melhor/**

Created by [Paulo Teixeira](https://ficaadica.com.br/sobre/) — creator of the
[Prompthen](https://prompthen.ai) method.

## The three test groups

Each run = 3 groups × 5 puzzles × 7 prompt formats × 5 repetitions = **525 cells**.

| Group | What it measures |
|---|---|
| **GRID** | Visual induction (ARC-AGI-2 style): infer a hidden rule from 3-4 grid→grid examples and apply it to a new input. Pure induction from minimal evidence. |
| **CASCADE** | Compositional reasoning: three chained transformations (Alpha→Beta→Gamma) where each output feeds the next. One early mistake destroys the final answer. |
| **VOLTGRID** | An **invented system** that does not exist anywhere on the internet: a documented spec with ONE hidden rule to infer from execution traces, then simulate. Designed to minimize training-data contamination (it did not exist before this study was published). The closest group to real-world engineering. |

Difficulty is a property of the puzzle, not the group: the hardest puzzle
(CASCADE/p2) was solved by **no model in 34 runs (0%)**; the easiest sits at ~82%.
The top overall score in the study is 77.5% — no model came close to the ceiling.

## The seven prompt formats

The same puzzles rendered in seven engineered prompt formats — numeric data byte-identical across all seven; instructions semantically equivalent per language:

| Format | Language | Included here |
|---|---|---|
| `md_en` / `md_pt` | Markdown, EN / PT | ✅ `prompts/` |
| `yaml_en` / `yaml_pt` | YAML, EN / PT | ✅ `prompts/` |
| `xml_en` / `xml_pt` | XML, EN / PT | ✅ `prompts/` |
| `ntc` | NTC (proprietary) | ❌ results only |

**NTC** is a proprietary prompt engineering created by Paulo Teixeira. Its
results are published in full; the notation itself is not.

## Repository layout

```
prompts/
  GRID/p0..p4/{md,yaml,xml}_{en,pt}.txt      # 90 ready-to-use prompts
  CASCADE/p0..p4/...
  VOLTGRID/p0..p4/...
results/
  per_cell.csv       # 17,850 rows — every single cell: run × group × puzzle ×
                     #   format × language × repetition × correct (0/1)
  overall.csv        # general ranking (34 runs, 525 cells each)
  grid.csv           # ranking on the GRID group (175 cells)
  cascade.csv        # ranking on the CASCADE group (175 cells)
  voltgrid.csv       # ranking on the VOLTGRID group (175 cells)
  by_format.csv      # per-run × per-format accuracy (all 7 formats, NTC included)
```

`per_cell.csv` contains the full paired data: it supports McNemar tests,
paired bootstrap, or hierarchical models by anyone who wants to run their
own statistical analysis. A cell not answered by a model counts as 0
(official scoring rule: no answer = incorrect).

Answer keys are intentionally **not** published to keep the benchmark usable
for future model evaluations. Scoring for the published runs was fully
deterministic (exact match).

## Headline findings

1. **Environment matters more than prompt format.** The same model, same
   prompts: Sonnet 5 scored 49.0% inside Claude Code vs 28.4% via Direct API
   (+20.6 pts). Opus 4.8: +8.4. No format choice in the study comes close to
   this effect. That is why rankings are published per environment.
2. **Thinking beats everything.** Gemini 3.5 Flash-Lite: 6.5% → 48.2%
   (+41.7 pts) just by enabling thinking. Measured effect hierarchy:
   thinking (>40 pts) > environment (7-27) > format (5-12) > language (2-4).
3. **There is no universal best format.** GPT models favor Portuguese/Markdown,
   xAI favors XML, Moonshot favors YAML, Claude favors NTC/XML-PT. Test on
   *your* model family.
4. **Portuguese beat English overall** on identical data: 49.19% vs
   47.88% across the md/yaml/xml pairs — and wins in all three engineerings.
   "Always prompt in English" did not survive contact with the data.
5. **NTC** ties for the top of the global format average (49.29%), shares the
   lead in per-run wins (a triple tie at 7 wins with xml_en and xml_pt), and is
   the most compact format on instruction-dense prompts (XML needs up to +67%
   more characters for the same content on VOLTGRID).
6. **Open-weight models arrived in force**: five open models within 4.3 points
   (Kimi K2.6 55.2 · K3 54.7 · DeepSeek v4-pro 53.0 · GLM 5.2 52.4 ·
   Qwen3.8-max 50.9), at the level of closed frontier models from ~6 months ago.

## Reproducing / citing

Every number in the case study can be recomputed from `results/per_cell.csv`
(sum `correct` by run, group or format). To cite:

> Teixeira, Paulo. *Prompthen Bench: 34 runs, 7 prompt engineerings, 2
> languages, deterministic verification.* Fica a Dica, July 2026.
> https://ficaadica.com.br/novidades/ntc-modelos-performam-melhor/

## License

Prompts and result data: CC BY 4.0 — use them freely, attribution appreciated.
