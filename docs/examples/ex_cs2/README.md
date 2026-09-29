# Case Study 2: correlated versus independent channel loss

This example reproduces one point of the Case Study 2 results in the
accompanying paper: the LCD strategy with no retransmissions, seed 1. It
compares two loss models on the user-satellite (access) link, at the same mean
loss, and everything else in the two configurations is identical.

- [`config_trace.py`](config_trace.py): a real burst-loss trace on the access link.
- [`config_bern.py`](config_bern.py): a Bernoulli model at matched mean loss.
- [`viewer.html`](https://gcampobello.github.io/ICARUS-LEO/examples/ex_cs2/viewer.html):
  the interactive session viewer. The two runs are both in it, and the drop-down
  at the top switches between them, so the same scenario can be inspected under
  the two loss models.

Both configurations were generated with the
[config builder](https://gcampobello.github.io/ICARUS-LEO/config_builder.html)
and were not edited by hand. Each was run on its own, and the viewer was then
produced from the two result files in one step, with the simulator's viewer
tool: given several result files, it puts all their experiments behind the
drop-down, exactly as it does for the several experiments of a single
parametric sweep.

## Scenario

- **Constellation**: `LEO_DYN_IRIDIUM`, an Iridium-like Walker-Star
  constellation of 66 satellites, kept static with a single snapshot and no
  handover blackout, so that the effect of the loss correlation is isolated from
  that of handover. Twelve ground stations, one receiver each, spread over the
  globe.
- **Workload**: 100 contents of 50 chunks of 8 KB each, that is 400 KB per
  content, Zipf exponent 1.0, 6000 warm-up and 4000 measured requests, seed 1.
- **Caching**: `RETRY_LCD_LOSSY` with no retransmissions, LRU replacement,
  330 cache slots, that is 500 chunks or ten contents per satellite.
- **Loss on the access link**: in `config_trace.py` a recorded trace
  (`LEO_downlink_loss-Ger0804-170001-1h.npz`); in `config_bern.py` a Bernoulli
  process with BER 1.302e-7, chosen so that the per-chunk loss on that link
  matches the mean of the trace. In both, the satellite links carry a common
  Bernoulli residual with BER 3e-8, so the access link is the dominant source of
  loss.

## Results

Content completion is the fraction of requested contents whose fifty chunks were
all delivered. It is the headline metric of Case Study 2, and the viewer reports
it in its header for each of the two runs.

| Metric | Trace | Bernoulli | Paper, LCD over 20 seeds |
|---|---|---|---|
| Content completion | 61.78% (2471 of 4000) | 55.00% (2200 of 4000) | 0.6143 plus or minus 0.0095, against 0.5414 plus or minus 0.0075 |
| Hit ratio, all chunks | 80.05% | 80.70% | 0.8066 plus or minus 0.0067, against 0.8118 plus or minus 0.0060 |
| Chunk delivery | 98.77% | 98.76% | not reported |

At seed 1 the real trace completes about 6.8 percentage points more contents
than the Bernoulli model at the same mean loss, which is the effect reported in
the paper: bursty losses cluster on fewer contents, so more contents are
delivered in full, whereas a memoryless model spreads the same losses over more
distinct contents and so underestimates completion. The hit ratio is
essentially unchanged between the two models, as the paper also reports. A
single seed is not expected to match the twenty-seed mean, and both values lie
within about one standard deviation of it.

## Reading the viewer

The viewer shows chunk-level sessions, one at a time, as a sequence of events
along the delivery path: request hops, cache or origin hits, data hops, losses
and retransmissions. The statistics in the header are computed over all 200 000
measured chunk sessions of the selected run, while the navigable list holds the
last 1000 of them. The card labelled *CACHE_HIT_RATIO* reports the hit ratio as
computed by the original Icarus collector, whose denominator counts only the
sessions that reached a serving node; the hit ratio in the header is the one
over all chunk sessions, and is the value comparable with the paper.
