# Case Study 2: correlated versus independent channel loss

This example reproduces the Case Study 2 results of the accompanying paper at
seed 1, for the LCD strategy. It compares two loss models on the user-satellite
(access) link, at the same mean loss, in the two retransmission regimes the
paper considers. Everything else in the four configurations is identical.

| Configuration | Access-link loss | Retransmissions |
|---|---|---|
| [`config_trace_m0.py`](config_trace_m0.py) | real burst-loss trace | none |
| [`config_bern_m0.py`](config_bern_m0.py) | Bernoulli at matched mean | none |
| [`config_trace_m1.py`](config_trace_m1.py) | real burst-loss trace | one per chunk |
| [`config_bern_m1.py`](config_bern_m1.py) | Bernoulli at matched mean | one per chunk |

[`viewer.html`](https://gcampobello.github.io/ICARUS-LEO/examples/ex_cs2/viewer.html)
holds all four runs, and the drop-down at the top switches between them.

All four configurations were generated with the
[config builder](https://gcampobello.github.io/ICARUS-LEO/config_builder.html)
and were not edited by hand. Each was run on its own, and the viewer was then
produced from the four result files in one step, with the simulator's viewer
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
- **Caching**: LCD with LRU replacement and 33 000 cache slots, that is 500
  chunks, or ten contents, per satellite.
- **Loss on the access link**: in the `trace` configurations a recorded trace
  (`LEO_downlink_loss-Ger0804-170001-1h.npz`); in the `bern` configurations a
  Bernoulli process with BER 1.302e-7, chosen so that the per-chunk loss on that
  link matches the mean of the trace. In all four, the satellite links carry a
  common Bernoulli residual with BER 3e-8, so the access link is the dominant
  source of loss.
- **Retransmissions**: none in the `m0` configurations; one per chunk in the
  `m1` ones, with a 300 ms timeout.

## Results

Content completion is the fraction of requested contents whose fifty chunks were
all delivered. It is the headline metric of Case Study 2, and the viewer reports
it in its header for each run.

| Run | Content completion | Hit ratio | Sessions with a retransmission |
|---|---|---|---|
| trace, no retransmissions | 61.78% | 80.05% | 0 |
| Bernoulli, no retransmissions | 55.00% | 80.70% | 0 |
| trace, one retransmission | 95.70% | 81.41% | 4546 |
| Bernoulli, one retransmission | 99.22% | 81.67% | 4842 |

**The sign reverses with retransmissions**, and this is the point of the case
study. Without them, the real trace completes about 6.8 percentage points more
contents than the Bernoulli model at the same mean loss: bursty losses cluster
on fewer contents, so more contents are delivered in full, and a memoryless
model underestimates completion. With one retransmission per chunk, the real
trace completes about 3.5 percentage points fewer: consecutive attempts fall
inside the same burst, so retransmissions recover isolated losses but not bursty
ones, and the memoryless model now overestimates how much an ARQ scheme
recovers on a real channel. The hit ratio is essentially unchanged across all
four runs, as the paper also reports.

The values measured by these runs coincide with those of the corresponding runs
in the paper's twenty-seed data set, at seed 1. A single seed is not expected to
match the twenty-seed means reported in the paper's tables.

## Reading the viewer

The viewer shows chunk-level sessions, one at a time, as a sequence of events
along the delivery path: request hops, cache or origin hits, data hops, losses
and retransmissions. In the two `m1` runs the *Retry* filter selects the
sessions in which a lost chunk was requested again, so the retransmission
mechanism can be followed step by step. The statistics in the header are
computed over all 200 000 measured chunk sessions of the selected run, while the
navigable list holds the last 500 of them. The card labelled *CACHE_HIT_RATIO*
reports the hit ratio as computed by the original Icarus collector, whose
denominator counts only the sessions that reached a serving node; the hit ratio
in the header is the one over all chunk sessions, and is the value comparable
with the paper.
