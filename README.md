# Quant alpha pipeline — method only

A reproducible way to run quantitative research on the [WorldQuant BRAIN](https://platform.worldquantbrain.com/)
platform: everything is configured in one file, every result is written to disk as it arrives,
a human decides which fields are worth testing, and the platform's own verdict is read instead
of being re-implemented.

The notebook is the method. The signal is not published.

![method](https://img.shields.io/badge/notebook-method%20only-1F4E3D)

## Why this exists

Research done in a browser tab is not reproducible. The simulation queues, one idea needs
several fields and windows, and everything — the experiment, the metrics, the reason a
candidate was dropped — lives in the page you are about to close.

Moving the same work into a script changes the property that matters: every run leaves a
record behind. Each simulation writes its expression, settings, returned identifier and
metric set to disk; a run resumes where it stopped; the experiment log ties a result set to
a configuration and a commit.

Two further pieces of the design came out of using it:

- **A request gate.** The platform throttles. All processes share one lock, keep a minimum
  interval between calls and back off with the retry hint. This is a requirement: after an
  account is throttled, nothing works for a while.
- **One human gate.** The machine builds the field library and filters it by coverage.
  Which fields are worth testing, and in which direction, is a judgement about economics.
  The pipeline exports a review sheet and stops there.

## What is in the notebook

| Step | What it does |
| --- | --- |
| 0 | Configuration — every knob in one JSON file |
| 1 | Credentials from the environment, one API client, one request gate |
| 2–3 | Dataset discovery and the field library (bounded per dataset, so coverage is not dominated by one dataset) |
| 4–5 | Coverage profiling and tagging fields by economic meaning |
| 6 | The human review gate |
| 7 | Deterministic candidate generation |
| 8 | Simulate and collect, writing to disk as it goes, resumable |
| 9–10 | Cheap screen and an in-sample/out-of-sample stability proxy |
| 11 | Correlation clustering — deduplication by similarity, not by threshold |
| 12–13 | Export schema and the experiment log |
| 14 | Reading the platform's own checks and failure messages |

## What is deliberately not here

- the field list, and the expression grammar with my operator and window choices
- any per-candidate metric, alpha identifier, or submitted portfolio
- the failure analysis and the classification of my results

Those stay local. If you want the same skeleton for your own research, the
configuration file is where your choices belong.

## Running it

```bash
pip install -r requirements.txt
cp factory_config.example.json factory_config.json   # then fill in your choices
export WORLDQUANT_USERNAME=...  WORLDQUANT_PASSWORD=...
jupyter lab alpha_pipeline_method.ipynb
```

Credentials come from the environment. They are never read from the notebook and never
written to a file that could be committed.

The first run stops at the review gate: open the generated `field_review.csv`, approve the
fields you want, and continue.

## Known limits

- The rolling-window robustness check is described but not implemented — the platform returns
  two segments rather than a return series.
- The clustering step compares metric vectors, not return correlations. It is a proxy for
  similarity, and it is the step that tends to bind before the score threshold does.
- The pipeline does not submit anything on its own. Submission is a decision made with the
  failure reasons in front of you.

## A note on the write-up

The reasoning behind this pipeline, including what it cost to get wrong, is written up in
[From manual submissions to a pipeline](https://voskamp099.github.io/writing/quant-alpha-pipeline/).

## Licence

MIT — see [LICENSE](LICENSE).
