# Writing Pipeline PoC — SPF, DKIM, and DMARC

Proof of concept for a staged, AI-driven writing pipeline — research → information outline → learner journey → initial writing → review → targeted revision → review → final material — evaluated against a one-shot generated baseline. [VISION.md](VISION.md) is the authoritative description of the experiment; this README is the navigation reference for the directory. Every artefact path below is relative to `dns/fiddling-with-spf-dkim-dmarc/`.

## Pipeline stages

Each stage produces one artefact. Stages marked *(expected)* have not run yet; their path is the convention the stage will follow.

0. **Baseline** — one-shot generated tutorial, pre-pipeline reference → `baseline/one-shot.md`
1. **Research** — sourced technical knowledge corpus → `research/knowledge-corpus.md`
2. **Information outline** — compact reference model of the intended knowledge → `outline/information-outline.md`
3. **Learner journey** — the ordered learning sequence → `outline/learner-journey.md`
4. **Initial writing** — first complete draft → `draft/v1.md`
5. **Review** — findings on the draft, one report per review perspective → `review/alignment-review.md` *(expected)*, `review/compression-review.md`, `review/editorial-review.md`
6. **Targeted revision** — draft revised to resolve the review findings → `draft/v2.md` *(expected)*
7. **Review** — verification that the findings were resolved → `review/verification-review.md` *(expected)*
8. **Final material** — the final concise learner material → `final/material.md` *(expected)*
9. **Evaluation** — verdict on the pipeline output vs. the baseline → `evaluation/poc-evaluation.md`

The review perspectives of stage 5 are the three from VISION.md § Review Pipeline: alignment and coverage, compression and information density, final editorial and learning.

## Directory layout

```text
fiddling-with-spf-dkim-dmarc/
├── README.md
├── VISION.md
├── baseline/
│   └── one-shot.md                # stage 0 → baseline/one-shot.md
├── research/
│   └── knowledge-corpus.md        # stage 1 → research/knowledge-corpus.md
├── outline/
│   ├── information-outline.md     # stage 2 → outline/information-outline.md
│   └── learner-journey.md         # stage 3 → outline/learner-journey.md
├── draft/
│   ├── v1.md                      # stage 4 → draft/v1.md
│   └── v2.md                      # stage 6 → draft/v2.md (expected)
├── review/
│   ├── alignment-review.md        # stage 5 → review/alignment-review.md (expected)
│   ├── compression-review.md      # stage 5 → review/compression-review.md
│   ├── editorial-review.md        # stage 5 → review/editorial-review.md
│   └── verification-review.md     # stage 7 → review/verification-review.md (expected)
├── evaluation/
│   └── poc-evaluation.md          # stage 9 → evaluation/poc-evaluation.md
└── final/
    └── material.md                # stage 8 → final/material.md (expected)
```

## Questions

The PoC is set up to produce evidence for the questions in VISION.md § Questions. The following are the ones this pipeline directly exercises (findings recorded in `evaluation/poc-evaluation.md`):

- Does building the information outline before writing materially improve the final result?
- Does retaining the outline prevent information loss during compression?
- Can AI design a sensible learning sequence from a technical knowledge outline?
- How much useful compression can be achieved without reducing technical completeness?
- Do specialized judge passes produce better results than generic editing prompts?
- Are three review stages useful, excessive, or insufficient?
- Does targeted ticket-based revision outperform complete document regeneration?

## Success

The pipeline output (`final/material.md`) is **meaningfully better** than the baseline (`baseline/one-shot.md`) when both of the following hold:

1. **Word count** — the pipeline output has fewer words than the baseline.
2. **Coverage** — an evaluator using `outline/information-outline.md` as a reference can verify that no technical concept present in the outline is absent from the pipeline output.

Both conditions must hold; either failing alone is not sufficient. The verdict is binary: `true` when both hold, `false` otherwise — there is no middle value.
