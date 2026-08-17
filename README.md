# Risk Quantifier: Heatmap to Histogram

An interactive tool for converting qualitative risk matrices into quantitative probability distributions using Monte Carlo simulation.

**Live:** https://rootcawsllc.github.io/risk-quantifier/

![Two risks placed on a 5x5 heat map. The first has been loaded from a UK financial-services data-breach benchmark, labelled "GBP · starter with source backed UK direct parameters · module governed · 6 medium" and showing its frequency and impact ranges, a "not good for" caveat, and all six cited sources with their individual limitations. Below, the two risks produce separate loss distributions in GBP and USD, and the portfolio total is withheld because the currencies differ](preview.png)

The same two risks, twice. On the heat map, two colored cells — no amounts, no ranges, nothing to decide on. Below, two loss distributions with percentiles you can budget against. That gap is the whole point of the tool.

The first risk also shows the second point: its numbers came from a benchmark, so every one of them arrives with the source that produced it and the limitation that source carries.

## Features

- **Interactive 5×5 Risk Heatmap** - Click to place up to 5 risks across likelihood and impact dimensions
- **Risk Parameterization** - Define min/likely/max values for frequency (events per year) and impact (per event) as PERT distributions, pre-filled from where you clicked on the heat map
- **Source-backed benchmarks** - Load a governed starting range for a country, sector and threat instead of typing one. Eleven shards across eight countries; each parameter shows its source, publication, confidence level and stated limitation, and each shard states what it is *not* good for
- **Currency handling** - Benchmarks are priced in their own currency (AUD, CAD, GBP, USD). Results are labelled accordingly, and the portfolio total is withheld rather than summing across currencies
- **Per-Risk Result Cards** - Each risk displays a loss histogram plus median (P50), expected value, and bad-year (P90) figures
- **Compound Poisson Simulation** - 10,000 iterations per risk; annual frequency drives a Poisson-distributed event count, and every event draws its own independent impact
- **Aggregate Results** - Combined annual loss statistics across all risks, summed year by year rather than by percentile, when every risk shares a currency
- **Responsive Design** - Works on desktop and tablet
- **Professional Color Palette** - Cohesive, accessible design system

## Usage

1. Click cells on the heatmap to place up to 5 risks
2. Each placed risk gets a parameter panel, pre-filled from the cell you clicked
3. Optionally pick a benchmark to replace those defaults with source-backed ranges, then read what that shard is not good for
4. Adjust frequency and impact parameters (min/likely/max) if you want to move off either starting point
5. Click "Run Simulation (10,000 iterations)"
6. View each risk's loss distribution, and — with more than one risk placed, all in the same currency — the portfolio total

## How It Works

Each risk starts life as a heat-map click, and the cell you click sets a real starting range, not a placeholder: "Rare / Negligible" and "Almost Certain / Severe" produce genuinely different frequency and impact ranges, not the same three numbers with a different color. That's the actual point of routing the heat map into a simulation instead of stopping at it — the familiar click becomes an on-ramp into quantification rather than the answer itself.

Those cell ranges are still invented, though, which is the honest weakness of every tool like this. So each risk can instead be loaded from a benchmark: a governed shard for a country, sector and threat where every parameter traces to a named public source. Picking one replaces the range, sets the currency, and shows the six citations behind it alongside the shard's own statement of what it will not support. The numbers get better and the caveats arrive with them — a range with no provenance and a range with six sourced parameters should not look equally authoritative, and here they don't.

From there, each simulated year:

1. **Frequency** is drawn from a PERT distribution over your min/likely/max — PERT rather than a plain triangle because it weights the "most likely" point instead of treating it as no more probable than the edges.
2. That frequency becomes the rate of a **Poisson draw**, which is what produces the actual event count for the year. A frequency of "3 times a year" does not mean exactly 3 — most years will be near it, some will be 0, a few will be considerably more.
3. **Every event that year gets its own independent impact draw**, also from a PERT distribution, and the year's total is the sum of them.

That last step is the one that matters most. The common shortcut in tools like this is `total = numEvents × oneImpactDraw` — treating a year with three events as a year with one event that cost three times as much. Those are different distributions: the shortcut suppresses variance and understates the tail, because the tail is exactly the case where several bad things land in the same year. This tool sums independent draws instead.

When more than one risk is placed, the portfolio total is the year-by-year **sum** of every risk's simulated losses — not the sum of their individual percentiles, which is a different and larger error (percentiles don't add; sums of random variables aren't the sum of their quantiles).

## Deployment

This is a standalone HTML file with no dependencies. Deploy to any web server or use locally by opening `index.html` in your browser.

### GitHub Pages

Deployed at https://rootcawsllc.github.io/risk-quantifier/.

## Honest limits

- **Per-event impacts are drawn fully independently within a year.** Real losses are rarely that clean — a breach severe enough to trigger a large notification cost is also more likely to trigger a large liability claim. This tool doesn't model that correlation; yeetmap's priced loss modules do, via shared per-event latents.
- **The default range for each heat-map cell is illustrative, not calibrated.** They exist so a click produces a real, distinct starting range instead of an arbitrary flat one — adjust them, or load a benchmark, before treating any output as a real estimate for your organisation.
- **The benchmarks are starting points, not benchmark-grade figures.** Each shard carries its own status, and most are labelled governed starters rather than reviewed benchmarks. Several borrow a frequency from another country because no local per-firm rate is published — the US data-breach shard uses a UK survey as a bridge, and says so in its own caveat. Read the per-parameter limitations before quoting any of it.
- **Benchmark data is a snapshot.** The figures are baked into `index.html` from one specific upstream commit. They do not update themselves; regenerate when the shards change.
- **No FX conversion.** Shards are priced in their own currency and there is no rate table, which is why a mixed-currency portfolio shows no total rather than a converted one.
- **No controls model, no reproducible seeding, and no provenance for anything you type by hand.** Benchmark parameters carry their sources; values you enter yourself carry nothing. This is a single-file teaching tool, not an audited engine. It has no tests. For the real thing — FAIR-CAM controls, an inverse pass, AI-assisted extraction with a strict tool schema, 300+ tests — see yeetmap.
- **Not a substitute for a real risk assessment.** It exists to make one specific point vivid: a heat map cell and a loss distribution are not the same kind of answer, and only one of them supports arithmetic.

## Attribution

Benchmark shards come from [RiskShard](https://github.com/raviaxo/RiskShard) by [raviaxo](https://github.com/raviaxo) — an evidence-governed cyber risk quantification project, AGPL-3.0, where every parameter traces to a reviewed public source. The frequency and impact ranges, source citations, confidence levels, and "not good for" statements in this tool are RiskShard's, carried through unchanged; the picker and the simulation are not. Each shard's underlying sources are credited individually in the tool itself and remain the property of their publishers.

Uses the same compound-Poisson approach — frequency as a Poisson rate, independent per-event magnitudes summed, not multiplied — as yeetmap, this author's full FAIR (Factor Analysis of Information Risk) quantification engine. FAIR itself is a model published by the [FAIR Institute](https://www.fairinstitute.org/); this tool does not implement or reproduce FAIR's published standards, only the same underlying simulation principle.

## License

Copyright (c) 2026 RootCaws LLC.

[GNU AGPL v3 or later](LICENSE). If you modify this and run it as a network service, the AGPL requires you to offer your users the modified source under the same terms.
