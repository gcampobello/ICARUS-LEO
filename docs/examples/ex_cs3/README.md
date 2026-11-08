# Case Study 3: the four caching strategies on Starlink-mini

This example reproduces the Case Study 3 results of the accompanying paper at
seed 1. Case Study 3 repeats Case Study 1 on a different constellation,
Starlink-mini, to check whether the change in the ranking of the caching
strategies between an ideal and a realistic network also holds there. The two
configurations cover the two scenarios compared in the paper:

| Configuration | Topology | Channel |
|---|---|---|
| [`config_static_ideal.py`](config_static_ideal.py) | static: one snapshot, no handover | ideal, no loss |
| [`config_dynamic_lossy.py`](config_dynamic_lossy.py) | dynamic: handovers with a one-second blackout | lossy |

Each configuration is a parametric sweep over the strategy name, so each
produces four experiments, one per caching strategy.
[`viewer.html`](https://gcampobello.github.io/ICARUS-LEO/examples/ex_cs3/viewer.html)
holds all eight, and the drop-down at the top switches between them.

Both configurations were generated with the
[config builder](https://gcampobello.github.io/ICARUS-LEO/config_builder.html)
and were not edited by hand. Each was run on its own, and the viewer was then
produced from the two result files in one step.

The dynamic configuration is the one of the [`ex_cs1`](../ex_cs1/) example
with only the constellation and the cache size changed, so the two examples
can be compared directly: `ex_cs1` gives the Iridium side of the same scenario.

## Scenario

- **Constellation**: `LEO_DYN_STARLINK_MINI`, a reduced version of the first
  Starlink shell: twelve orbital planes of eleven satellites each, 132
  satellites in all, at 550 km and 53° inclination. Link capacities are those
  of the preset: 100 Mb/s on the user-satellite links, 10 Gb/s on the
  inter-satellite links and 2 Gb/s on the ground-satellite links. Twelve ground
  stations, one receiver each, spread over the globe.
  - In `config_static_ideal.py` the constellation is kept static, with a
    single snapshot and no handover blackout.
  - In `config_dynamic_lossy.py` it is fully dynamic: 100 snapshots follow the
    satellites along one orbital period, and each handover interrupts the
    access link for one second.
- **Workload**: 100 contents of one chunk of 8 KB each, Zipf exponent 1.0,
  20 000 warm-up and 30 000 measured requests, seed 1.
- **Caching**: LRU replacement, 1320 cache slots, that is ten contents per
  satellite, as in Case Study 1; with twice as many satellites, the total cache
  capacity is twice that of the Iridium-like constellation. The four
  strategies are LCE, LCD, ProbCache and CL4M, each in its retransmission-aware
  variant, with retransmissions switched off.
- **Channel**: in `config_dynamic_lossy.py`, Bernoulli loss with bit error
  rate 2e-6 on the user-satellite, ground-satellite and inter-satellite links;
  in `config_static_ideal.py`, all bit error rates are zero.

## Results

| Strategy | Static, ideal: hit ratio | Dynamic, lossy: hit ratio | Dynamic, lossy: delivered | Dynamic, lossy: mean latency |
|---|---|---|---|---|
| LCE | 70.63% | 70.81% | 66.63% | 46.452 ms |
| LCD | 80.68% | 74.88% | 68.83% | 43.011 ms |
| ProbCache | 85.07% | 72.09% | 68.43% | 43.627 ms |
| CL4M | 72.86% | 73.34% | 69.66% | 41.219 ms |

In the static scenario with ideal channel every request is delivered.

The values measured by these runs coincide with those of the corresponding runs
in the paper's ten-seed data set, at seed 1. The ranking change reported in the
paper is already visible at this seed. In the static scenario with ideal
channel, ProbCache and LCD give the highest hit ratios, well above CL4M and
LCE. In the dynamic scenario with lossy channel, LCD gives the highest hit
ratio, followed by CL4M and ProbCache, and CL4M delivers the most requests
with the lowest latency. A single seed is not expected to match the ten-seed
means reported in the paper.

## Other seeds

The paper averages ten runs, with seeds 1 to 10. In each run the same seed is
used for the workload and for the loss model. To obtain run *s*, load either
configuration in the builder (*Load* tab), set *s* both as the workload seed
and as the loss model seed, and download the file again.

## Reading the viewer

The viewer shows session-level events along the delivery path: request hops,
cache or origin hits, data hops, losses and handover blackouts, each with its
timing. The statistics in the header are computed over all 30 000 measured
sessions of the selected experiment, while the navigable list holds the last
500 of them. Because each content here is a single chunk, one session is one
content request. In the dynamic experiments, filters let you select, for
example, the sessions affected by a handover.
