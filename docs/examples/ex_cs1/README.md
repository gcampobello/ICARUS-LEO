# Case Study 1: four caching strategies on a dynamic constellation

This example reproduces one point of the Case Study 1 results in the
accompanying paper: the four caching strategies compared on a dynamic
constellation with a lossy channel, seed 1.

- [`config.py`](config.py): the configuration, generated with the
  [config builder](https://gcampobello.github.io/ICARUS-LEO/config_builder.html)
  and not edited by hand. It is a single file: a parametric sweep over the
  strategy name produces the four experiments in one run.
- [`viewer.html`](https://gcampobello.github.io/ICARUS-LEO/examples/ex_cs1/viewer.html):
  the interactive session viewer. The drop-down at the top switches between the
  four strategies, so the same scenario can be inspected under each of them.

This example is a good place to see what a **parametric sweep** looks like: one
configuration, four experiments, one result file and one viewer. The user guide
describes sweeps in its section on parametric sweeps.

## Scenario

- **Constellation**: `LEO_DYN_IRIDIUM`, an Iridium-like Walker-Star
  constellation of 66 satellites, this time fully dynamic: 100 snapshots follow
  the satellites along one orbital period, and each handover interrupts the
  access link for one second. Twelve ground stations, one receiver each, spread
  over the globe.
- **Workload**: 100 contents of one chunk of 8 KB each, Zipf exponent 1.0,
  20 000 warm-up and 30 000 measured requests, seed 1.
- **Caching**: LRU replacement, 660 cache slots, that is ten contents per
  satellite. The four strategies are LCE, LCD, ProbCache and CL4M, each in its
  retransmission-aware variant, with retransmissions switched off so that the
  comparison is between the caching decisions alone.
- **Channel**: Bernoulli loss with bit error rate 2e-6 on the user-satellite,
  ground-satellite and inter-satellite links.

## Results

| Strategy | Hit ratio | Delivered | Mean latency |
|---|---|---|---|
| LCE | 65.65% | 66.52% | 101.770 ms |
| LCD | 73.89% | 69.25% | 93.947 ms |
| ProbCache | 72.19% | 69.50% | 94.489 ms |
| CL4M | 72.72% | 70.04% | 92.228 ms |

The values measured by these runs coincide with those of the corresponding runs
in the paper's twenty-seed data set, at seed 1. The ranking of the strategies is
the one reported in the paper: LCD gives the highest hit ratio, followed by CL4M
and ProbCache, with LCE last, while the delivery ratio and the latency follow the
opposite order at the top, CL4M delivering the most and with the lowest latency.
A single seed is not expected to match the twenty-seed means reported in the
paper's table.

## Reading the viewer

The viewer shows session-level events along the delivery path: request hops,
cache or origin hits, data hops, losses and handover blackouts, each with its
timing. The statistics in the header are computed over all 30 000 measured
sessions of the selected strategy, while the navigable list holds the last 500
of them. Because each content here is a single chunk, one session is one
content request. Filters let you select, for example, the sessions affected by
a handover.
