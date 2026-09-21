# ICARUS-LEO User Guide

> [!NOTE]
> An online service that will let you run ICARUS-LEO yourself is in
> preparation.

This guide explains how to configure simulations with **ICARUS-LEO**, the
simulator of Information-Centric Networking (ICN) caching over LEO satellite
constellations presented in the accompanying paper. It covers three things:

1. how to describe your scenario with the [**config builder**](https://gcampobello.github.io/ICARUS-LEO/config_builder.html) and obtain a
   configuration file;
2. how to send that file to us for execution;
3. what you receive back and how to read it.

## Contents

- [1. How access works](#1-how-access-works)
- [2. Opening the config builder](#2-opening-the-config-builder)
- [3. Quick start: your first request](#3-quick-start-your-first-request)
- [4. The builder at a glance](#4-the-builder-at-a-glance)
- [5. Configuration reference](#5-configuration-reference)
- [6. Parametric sweeps](#6-parametric-sweeps)
- [7. Seeds and reproducibility](#7-seeds-and-reproducibility)
- [8. Reloading and editing a configuration](#8-reloading-and-editing-a-configuration)
- [9. Sending your request](#9-sending-your-request)
- [10. What you receive](#10-what-you-receive)
- [11. Warnings and common mistakes](#11-warnings-and-common-mistakes)

## 1. How access works

1. **Prepare** your scenario in the [config builder](https://gcampobello.github.io/ICARUS-LEO/config_builder.html), a single HTML page that
   runs in your browser.
2. **Download** the configuration file that the builder generates: a small
   Python file (`.py`) describing one experiment or a sweep of experiments.
3. **Email** the file, zipped and attached to the message, to
   Prof. G. Campobello
   ([gcampobello@unime.it](mailto:gcampobello@unime.it)). Mail services
   usually block `.py` attachments, so the zip archive is not optional.
4. **Receive** a link to download the output: a results file (`.pkl`) and/or
   an interactive session viewer (`.html`).

Requests are reviewed individually and accepted where the computational effort
involved is limited. A web service that will let you run ICARUS-LEO online is
under development; this guide will be updated when it becomes available.

## 2. Opening the config builder

The config builder is a single HTML page. You can open it in either of two
ways:

- from the project's GitHub Pages site, at
  [`https://gcampobello.github.io/ICARUS-LEO/config_builder.html`](https://gcampobello.github.io/ICARUS-LEO/config_builder.html), which
  publishes the repository's `docs/` folder;
- by downloading [`docs/config_builder.html`](https://github.com/gcampobello/ICARUS-LEO/blob/main/docs/config_builder.html) from the repository and opening it
  locally in any recent web browser.

The builder works entirely offline. It does not send data anywhere: the
configuration is assembled in your browser and saved to your computer when you
click *Download .py*.

## 3. Quick start: your first request

The builder opens with a complete, valid configuration already filled in: a
static Iridium-like constellation, a chunked content workload, LRU caches, the
retransmission-enabled LCD strategy and Bernoulli channel loss. A first
request can therefore be prepared in a few steps.

1. In **Experiment**, give the file a `filename` (for example
   `config_iridium_lcd.py`) and a short description in `desc`.
2. In **Topology**, choose a constellation in `name`.
3. In **Workload**, set the catalogue size, popularity and number of requests.
4. In **Strategy**, choose a caching strategy; the `RETRY_*` strategies are
   the ones used in the paper.
5. In **Loss model**, choose how packets are lost on each type of link.
6. In **Collectors**, tick `SESSION_LOG` if you want the interactive viewer.
7. Open **Output**, check the preview, and click *Download .py*.
8. Zip the file and email it as described in
   [Section 9](#9-sending-your-request).

The generated file for the default settings looks like this (header omitted):

```python
DATA_COLLECTORS = ['CACHE_HIT_RATIO', 'LATENCY', 'CHUNK_LEVEL', 'LEO_METRICS']

EXPERIMENT_QUEUE = []

# --- Single experiment ---
experiment = Tree()
experiment['topology'] = {
        'name': 'LEO_IRIDIUM',
        'seed': 42,
    }
experiment['workload'] = {
        'name': 'STATIONARY_CHUNKED',
        'n_contents': 100000,
        'n_chunks': 4,
        'alpha': 1.0,
        'n_warmup': 5000,
        'n_measured': 20000,
        'rate': 1.0,
        'seed': 42,
    }
experiment['cache_placement'] = {
        'name': 'UNIFORM',
        'network_cache': 0.05,
    }
experiment['content_placement'] = {
        'name': 'UNIFORM_CHUNKED',
        'seed': 42,
    }
experiment['cache_policy'] = {
        'name': 'LRU',
    }
experiment['strategy'] = {
        'name': 'RETRY_LCD_LOSSY',
        'max_retries': 2,
        'retry_timeout_ms': 300.0,
        'content_size_bytes': 8192,
        'loss_model': {
            'name': 'BERNOULLI',
            'ber_gsl': 5e-6,
            'ber_isl': 5e-7,
            'ber_access': 1e-9,
            'seed': 42,
        },
    }
experiment['desc'] = 'My experiment'
EXPERIMENT_QUEUE.append(experiment)
```

The file contains parameters only. Its header also lists the commands used to
run it with the simulator engine; you do not need to run them yourself.

## 4. The builder at a glance

**Tabs.** The parameters are grouped in tabs: *Experiment*, *Topology*,
*Workload*, *Cache*, *Strategy*, *Loss model* and *Collectors*. Two further
tabs, *Load* and *Output*, reload an existing file and show the generated one.
Sub-sections appear only when they are relevant: for example, the handover
parameters are shown only for dynamic topologies, and the retransmission
parameters only for `RETRY_*` strategies.

**Live preview.** The configuration is regenerated whenever you change a field
or switch tab, so the *Output* tab always reflects the current state of the
form. You can download the file or copy the text from the preview.

**Banner.** The banner at the top shows how many experiments the configuration
contains and an estimate of the number of sessions that will be recorded
(`n_measured` × `n_chunks` per experiment). It turns red if a field contains an
invalid value, and switches to a warning style if a sweep exceeds 50
experiments.

**Validation.** Numeric fields accept plain numbers and scientific notation
(`0.05`, `1.5e6`, `5e-6`). Use a full stop as the decimal separator: `0,8` is
rejected and flagged under the field.

**Emitted only when needed.** Several parameters are written to the file only
when they differ from the simulator's default (for example `n_receivers_per_gs`
or `min_elevation_deg`). A parameter missing from the preview is therefore not
lost: it takes its default value.

## 5. Configuration reference

Field names in this section are those shown in the builder. Where a field is
written to the file under a different key, the key is given as well.

### 5.1 Experiment

| Field | Meaning |
|---|---|
| `filename` | Name of the file you download, for example `config_my_experiment.py`. |
| `desc` | Free-text description of the experiment, stored with the results. In a sweep, the values of the swept parameters are appended automatically, so that each experiment remains identifiable. |
| `LOG_LEVEL` | Verbosity of the simulator's log. `INFO` is appropriate in almost all cases. |
| `PARALLEL_EXECUTION`, `N_PROCESSES` | Execution settings for our server. You can leave them at their defaults; we may adjust them to suit the machine the request runs on. |

Each configuration runs every experiment once (`N_REPLICATIONS = 1`) and stores
results in Python pickle format. To repeat an experiment with different random
seeds, sweep a seed, as described in [Section 6](#6-parametric-sweeps).

### 5.2 Topology

#### Constellation

The `name` field selects the constellation. Two families are available.

| Family | Names | Description |
|---|---|---|
| Static | `LEO_IRIDIUM`, `LEO_STARLINK_MINI`, `LEO` (custom) | Satellite positions are computed analytically and the topology does not change during the run. |
| Dynamic | `LEO_DYN_IRIDIUM`, `LEO_DYN_STARLINK_MINI`, `LEO_DYN` (custom) | Satellites move. The topology is updated through a sequence of precomputed snapshots, and receivers and ground stations experience handovers. |

`LEO_IRIDIUM` is an Iridium-like constellation of 66 satellites in 6 near-polar
planes of 11 satellites, at 780 km altitude and 86.4° inclination.
`LEO_STARLINK_MINI` is a reduced Starlink-like shell at 550 km.

The custom entries (`LEO`, `LEO_DYN`) show a *Custom Walker constellation*
sub-section where you define the constellation yourself:

| Field | Default | Meaning |
|---|---|---|
| `n_planes` | 6 | Number of orbital planes. |
| `n_sats_per_plane` | 11 | Satellites per plane. |
| `altitude_km` | 780 | Orbital altitude, in km. |
| `inclination_deg` | 86.4 | Orbital inclination, in degrees. |
| `isl_type` | `plus_grid` | Inter-satellite links (ISLs). `plus_grid`: each satellite is linked to its neighbours in the same plane and in the adjacent planes. `intra_only`: links within the same plane only. |
| Walker pattern | Delta (360°) | How the orbital planes are spread in right ascension. *Delta* spreads them over 360° (for example Starlink), *Star* over 180° (for example Iridium). Written to the file as `raan_span_deg` only when *Star* is chosen. |
| `phase_offset_deg` | 0.0 | Phase offset, in degrees, between satellites in adjacent planes (Walker phasing, F × 360 / total number of satellites; 10.909 for Iridium). |

`seed` (default 42) initialises the random elements of the topology.

#### Common parameters

These apply to every constellation.

| Field | Default | Meaning |
|---|---|---|
| `n_receivers_per_gs` | 4 | Number of receivers (user terminals) associated with each ground station. |
| `n_sources` | 3 | Number of content sources (origin servers). |
| `n_gsl_per_gs` | 2 | Number of ground-to-satellite links (GSLs) per ground station. |
| `min_elevation_deg` | 5.0 | Minimum elevation angle, in degrees, for a satellite to be usable from the ground. |
| `allow_gs_transit` | on | When on, a ground station can also relay traffic between two satellites, its two GSLs acting as a two-hop bridge. When off, the ground station is a pure gateway and inter-satellite traffic stays on the ISLs. This affects routing only, not link delays. |

#### Link rates

`isl_rate_bps`, `gsl_rate_bps`, `user_radio_rate_bps` and
`source_link_rate_bps` set the data rate, in bit/s, of each type of link (see
the link types in [Section 5.6](#56-loss-model)). Leave a field empty to keep
the default of the selected constellation.

#### Receiver placement

| `receiver_distribution` | Meaning |
|---|---|
| `colocated` (default) | Receivers are placed with the ground stations (backhaul scenario). |
| `uniform_global_safe` | Receivers are spread over the globe (direct-to-device scenario). |

With `uniform_global_safe` three further fields appear:
`receiver_placement_seed` (the seed for the placement; empty uses the topology
seed), `receiver_lat_cap_deg` (maximum absolute latitude of a receiver, default
85°) and `receiver_max_attempts` (maximum number of placement attempts per
receiver, default 10).

#### Dynamic topologies and handover

These fields appear for the `LEO_DYN_*` constellations.

| Field | Default | Meaning |
|---|---|---|
| `n_snapshots` | 100 | Number of topology snapshots used to follow the satellites' motion. The snapshots cover one orbital period, which the simulation repeats cyclically. More snapshots give a finer time resolution at a higher computational cost. Next to the field, the builder suggests a minimum value, chosen so that the interval between snapshots stays well below the time a satellite remains visible from the ground; it depends on the constellation and on `min_elevation_deg`, not on the length of the workload. |
| `disable_handover` | off | Removes all handover effects. Equivalent to setting the blackout duration to zero. |
| `handover_source` | `analytic` | How handover instants are determined. `analytic`: a closed-form schedule derived from the constellation geometry. `snapshot`: the serving satellite is re-evaluated at each snapshot. |
| `handover_period_s` | empty | `analytic` only. Forces a handover every N seconds instead of the geometric schedule. Leave empty to use the geometry. |
| `handover_blackout_distribution` | `fixed` | Duration of the link interruption at each handover: `fixed` (a single value) or `uniform` (random between a minimum and a maximum). |
| `handover_blackout_s` | 0.05 | `fixed` only. Blackout duration, in seconds. |
| `handover_blackout_min_s`, `handover_blackout_max_s` | empty | `uniform` only. Bounds of the blackout duration, in seconds. |
| `handover_blackout_seed` | 42 | `uniform` only. Seed for the random durations; empty uses the topology seed. |

A request or data packet transmitted during a blackout is lost.

### 5.3 Workload

The workload describes the content catalogue and the stream of requests.

| Field | Default | Meaning |
|---|---|---|
| `name` | `STATIONARY_CHUNKED` | `STATIONARY_CHUNKED`: each content is split into chunks, and each chunk is requested and delivered as a separate Data packet, as in ICN. `STATIONARY`: the original Icarus workload, one request per content. |
| `n_contents` | 100000 | Size of the content catalogue. |
| `n_chunks` | 4 | Chunks per content (`STATIONARY_CHUNKED` only). |
| `alpha` | 1.0 | Exponent of the Zipf popularity distribution. Larger values concentrate requests on fewer contents. |
| `n_warmup` | 5000 | Content requests issued before measurement starts, to fill the caches. |
| `n_measured` | 20000 | Content requests that are measured. |
| `rate` | 1.0 | Mean content request rate, in requests per second. |
| `chunk_size_bytes` | 8192 | Size of a chunk, that is, of an ICN Data packet, in bytes. Written to the file as `content_size_bytes` in the strategy. |
| `seed` | 42 | Seed for the request stream. |

In the results, a **session** is the request and delivery of one chunk. An
experiment with the chunked workload therefore records `n_measured` ×
`n_chunks` sessions.

### 5.4 Cache

| Field | Default | Meaning |
|---|---|---|
| `cache_placement.name` | `UNIFORM` | How the total cache budget is distributed over the satellites: `UNIFORM`, `DEGREE`, `BETWEENNESS_CENTRALITY` or `CONSOLIDATED`. |
| `network_cache` | 0.05 | Total number of cache slots, as a multiple of the catalogue size (`n_contents`). Each slot holds one chunk, so with the chunked workload the fraction of the catalogue that fits in the caches is `network_cache` / `n_chunks` (1.25% with the defaults). |
| `content_placement.name` | `UNIFORM_CHUNKED` | How contents are assigned to the sources. `UNIFORM_CHUNKED` is intended for the chunked workload; `UNIFORM` and `WEIGHTED` are the original Icarus placements. |
| `content_placement.seed` | 42 | Seed for the content placement. |
| `cache_policy.name` | `LRU` | Replacement policy of each cache: `LRU`, `FIFO`, `RAND`, `CLIMB`, `IN_CACHE_LFU`, `PERFECT_LFU`, `SLRU` or `NULL`. |

Caches are placed on satellites only. Below `network_cache` the builder
estimates the cache size per satellite. If the budget would give less than one
unit per satellite, the estimate is shown in red: the simulator would round
every cache up to one unit, which distorts the experiment. Increase `n_contents` or `network_cache` until the warning
disappears.

### 5.5 Strategy

The strategy decides where content is cached along the delivery path.

| Group | Names | Notes |
|---|---|---|
| Retry | `RETRY_LCE_LOSSY`, `RETRY_LCD_LOSSY`, `RETRY_PROB_CACHE_LOSSY`, `RETRY_CL4M_LOSSY` | LCE, LCD, ProbCache and CL4M, with the loss model applied on every hop and retransmission of unanswered requests. These are the strategies used in the paper. |
| Legacy / no-loss | `LCD`, `LCE`, `NO_CACHE`, `EDGE`, `PARTITION`, `NRR`, `RAND_BERNOULLI`, `RAND_CHOICE` | Original Icarus strategies. |
| Hash-routing | `HASHROUTING`, `HR_EDGE_CACHE`, `HR_ON_PATH`, `HR_CLUSTER`, `HR_SYMM`, `HR_ASYMM`, `HR_MULTICAST`, `HR_HYBRID_AM`, `HR_HYBRID_SM` | Original Icarus hash-routing strategies. |

Only the `RETRY_*` strategies apply the loss model, the handover blackouts and
the topology updates of the dynamic constellations. Use them for any scenario
with loss or with a `LEO_DYN_*` constellation. The other strategies are useful
as lossless baselines on static constellations.

**Retry parameters** (`RETRY_*` only):

| Field | Default | Meaning |
|---|---|---|
| `max_retries` | 2 | Maximum number of retransmissions of a request. `0` gives a single attempt. |
| `retry_timeout_ms` | 300.0 | Time, in ms, after which an unanswered request is retransmitted. Keep it larger than the round-trip time, otherwise requests are retransmitted even when nothing was lost. |

**ProbCache parameter** (`RETRY_PROB_CACHE_LOSSY` only): `t_tw`, the time
window parameter of ProbCache (default 10).

**Latency.** The latency of each hop always includes the propagation delay and
the transmission time of the packet, that is, its size in bits divided by the
link rate set in *Topology*. Queueing delays are not modelled.

### 5.6 Loss model

The loss model decides whether each packet is lost on each link it traverses.
ICARUS-LEO distinguishes five **link types**:

| Link type | Between |
|---|---|
| `user_radio` | Receiver (user terminal) and satellite. |
| `gsl` | Ground station and satellite. |
| `isl_intra` | Satellites in the same orbital plane. |
| `isl_inter` | Satellites in adjacent planes. |
| `access` | Ground station and content server (cable or fibre). |

Bit-error rates (BERs) are per bit: a packet of L bits is lost with probability
1 − (1 − BER)<sup>L</sup>. Requests and Data packets are therefore affected
differently, because of their different sizes.

Choose the model in `name`; its parameters appear below it. `seed` (default 42)
initialises the loss process; with `COMPOSITE`, each sub-model has its own seed.

**`NONE`.** No channel loss. With a dynamic constellation and an active
handover, the builder writes a Bernoulli model with all BERs set to zero
instead, and says so in a comment in the file. This is intended: the handover
blackout is applied by the loss model, and without one it would have no effect.
The substitute adds no channel loss.

**`BERNOULLI`.** Independent losses with a fixed BER per link type.

| Field | Default | Link type |
|---|---|---|
| `ber_gsl` | 5e-6 | `gsl` |
| `ber_isl` | 5e-7 | `isl_intra` and `isl_inter` |
| `ber_access` | 1e-9 | `access` |
| `ber_user_radio` | empty | `user_radio`; empty uses `ber_gsl` |

**`GILBERT_ELLIOTT`.** A two-state Markov model producing bursts of losses. The
channel alternates between a Good and a Bad state, each with its own BER.

| Field | Default | Meaning |
|---|---|---|
| `p_gb` | 0.02 | Probability of moving from Good to Bad. |
| `p_bg` | 0.15 | Probability of moving from Bad to Good. |
| `ber_good`, `ber_bad` | 1e-8, 1e-4 | BER in each state, used for any link type without a specific value. |
| `ber_good_<type>`, `ber_bad_<type>` | empty | Specific BERs for `gsl`, `isl` (both ISL types), `access` and `user_radio`. A specific value takes precedence over the scalar one; `user_radio` falls back to `gsl`. |
| `handover_forces_bad` | off | Forces the Bad state during handovers. Leave it off with dynamic constellations: the blackout is already applied, and forcing the Bad state as well would count the same event twice. |

**`COMPOSITE`.** A different model for each link type, for example a recorded
trace on the radio link, Bernoulli on the satellite links and no loss on the
cabled links. Each link type has its own collapsible section with a
`sub-model`:

- `NONE`: no loss on that link type (the default for every link type);
- `BERNOULLI`: a single BER for that link type, and an optional seed;
- `GILBERT_ELLIOTT`: `p_gb`, `p_bg`, `ber_good`, `ber_bad` and an optional
  seed, applied to that link type;
- `TRACE`: losses replayed from a recorded trace, a `.npz` file given in
  `trace_npz_path`, with an optional seed. Because the simulation runs on our
  server, the trace must be available there: give the name of one of the
  traces we provide, or ask about using your own trace in your email.

### 5.7 Collectors

Collectors decide which measurements are stored in the results. Select at least
one.

| Collector | Default | Content |
|---|---|---|
| `CACHE_HIT_RATIO` | on | Cache hit ratio as computed by the original Icarus collector. |
| `LATENCY` | on | Latency as computed by the original Icarus collector. |
| `CHUNK_LEVEL` | on | Chunk-level metrics of ICARUS-LEO, including the chunk hit ratio. |
| `LEO_METRICS` | on | ICARUS-LEO metrics, including end-to-end latency and delivery counts. |
| `LINK_LOAD` | off | Load on each link (original Icarus collector). |
| `PATH_STRETCH` | off | Path stretch (original Icarus collector). |
| `SESSION_LOG` | off | A per-session log of events. **Required for the interactive viewer.** It makes the results file larger. |

## 6. Parametric sweeps

A sweep runs the same scenario for several values of one or more parameters.
You define it directly in the builder.

**Numeric fields.** Type a JSON array instead of a single value, for example
`[1e-6, 5e-6, 1e-5]` in `ber_gsl`. The field is highlighted to show that it is
swept.

**Choice fields.** Selections such as the topology, the strategy, the cache
placement, the content placement, the cache policy and the loss model have an
*or sweep* box underneath. Type a JSON array of names, in double quotes, for
example `["RETRY_LCE_LOSSY", "RETRY_LCD_LOSSY"]`. A valid array there overrides
the selection above it.

**Several swept fields** produce every combination of their values. For
example, sweeping four strategies and three workload seeds

```text
strategy, or sweep:  ["RETRY_LCE_LOSSY", "RETRY_LCD_LOSSY", "RETRY_PROB_CACHE_LOSSY", "RETRY_CL4M_LOSSY"]
workload seed:       [1, 2, 3]
```

gives 3 × 4 = 12 experiments. The banner shows the count and the estimated
number of sessions before you download. In the generated file, the sweep is a
loop over all combinations, and each experiment's description records its own
values, for example `My experiment [wl_seed=2, strat_name=RETRY_LCD_LOSSY]`.

**Replications.** To repeat an experiment with independent randomness, sweep a
seed, typically the workload seed, over as many values as the replications you
need.

**Size.** The cost of a request grows with the number of experiments and
sessions. The banner flags sweeps of more than 50 experiments; large sweeps may
need to be discussed with us before they can be accepted.

## 7. Seeds and reproducibility

Several components draw random numbers: the topology, the receiver placement,
the request stream, the content placement and the loss model. Each has its own seed field, set to 42 by default.

If a seed is left empty, that component is initialised from system entropy and
the run can no longer be reproduced. The builder then shows a warning above the
preview in the *Output* tab, listing the empty seeds. Set every seed unless you
deliberately want an irreproducible run.

## 8. Reloading and editing a configuration

The *Load* tab reads a `.py` file previously generated by the builder, single
experiment or sweep, and fills in the form with its values. You can then change
any field and download the modified file from *Output*. This is the simplest
way to prepare variants of a scenario.

Only files generated by the builder can be loaded, including those generated by
earlier versions. Files edited by hand may not be read correctly.

After loading, a message next to the *Load* button reports the outcome. If some
values in the file cannot be restored, for example an option that the builder
no longer offers, the message turns orange and lists them: the configuration you
download will differ from the file on those points. Parameters that have no
effect in the current version are ignored and listed in the message as well.
Check the preview in *Output* before downloading.

## 9. Sending your request

Email the configuration file, as an attachment, to Prof. G. Campobello,
Department of Engineering, University of Messina, Italy:
[gcampobello@unime.it](mailto:gcampobello@unime.it).

**Send the file inside a zip archive.** Most mail services block or strip `.py`
attachments, so a configuration sent as it is may never reach us.

In the message, please include:

- the configuration file (`.py`), unchanged from the builder, zipped;
- which output you would like: the results file, the session viewer, or both.
  For the viewer, make sure `SESSION_LOG` is ticked in *Collectors*;
- a sentence on the purpose of the simulations and your affiliation;
- anything the builder cannot express, such as a parameter you would like to
  vary that has no field, or a loss trace of your own.

Please do not edit the generated file by hand. If something cannot be set in
the builder, describe it in your message instead.

## 10. What you receive

When the simulation is complete you receive a link to download the output.

### 10.1 The results file (`.pkl`)

The results file holds, for each experiment in your configuration, that
experiment's parameters together with the measurements of the collectors you
selected, grouped by collector. It is a Python pickle, so it is read with the
standard `pickle` module; the message that carries your download link states
what the file needs in order to be opened and names the fields of the metrics
below, so that you can go straight to the numbers you care about.

As with any pickle, only open files from a source you trust, such as the
download link we send you.

The measurements reported in the paper are:

| Metric | Meaning |
|---|---|
| Chunk hit ratio | Fraction of chunk requests served by an in-network cache rather than by the origin, over all requests. Produced by the `CHUNK_LEVEL` collector. |
| Delivery ratio | Fraction of chunk requests that were delivered, that is, delivered sessions over total chunk requests. Produced by the `LEO_METRICS` collector. |
| End-to-end latency | Mean latency of the delivered sessions, in ms, including retransmission timeouts. Produced by the `LEO_METRICS` collector. |

The `CACHE_HIT_RATIO` collector of the original Icarus computes the hit ratio
over the sessions that reached a serving node only. With losses, that
denominator excludes lost requests, so its value is not comparable with the
chunk hit ratio. For hit ratios, use the chunk hit ratio of the `CHUNK_LEVEL`
collector.

### 10.2 The session viewer (`.html`)

The session viewer is a self-contained HTML page that opens in any web browser,
offline. It shows the recorded sessions one at a time, as a sequence of events
along the path: request hops, cache or origin hits, data hops, losses,
retransmissions and handover blackouts, each with its timing. Filters let you
select, for example, the sessions with retransmissions or those affected by a
handover.

The viewer includes a sample of the recorded sessions, to keep the file
manageable. If you need all of them, say so in your email. The viewer can only
be produced if `SESSION_LOG` was selected in the configuration.

## 11. Warnings and common mistakes

| What you see | Cause | What to do |
|---|---|---|
| A field outlined in red, *not a valid number* | Letters, spaces or a comma decimal separator in a numeric field. | Use plain numbers with a full stop, for example `0.8` or `1e-6`. |
| *malformed JSON array* | A sweep array that is not valid JSON, for example with single quotes or a missing bracket. | Use square brackets, commas, and double quotes for names: `["LRU", "FIFO"]`. |
| Red cache estimate under `network_cache` | Less than one cache unit per satellite. | Increase `n_contents` or `network_cache`. |
| Warning listing empty seeds (*Output* tab) | One or more seed fields are empty. | Fill them in, unless an irreproducible run is intended. |
| A note that `NONE` has been replaced by a lossless Bernoulli | `NONE` selected with an active handover on a dynamic constellation. | Nothing: this is intended (see [Section 5.6](#56-loss-model)). |
| Banner in warning style | The sweep has more than 50 experiments. | Consider reducing it, or discuss it with us first. |
| No viewer in the output | `SESSION_LOG` was not selected. | Tick `SESSION_LOG` and send the request again. |
| Orange message after loading a file (*Load* tab) | Some values in the file cannot be restored, for example an option the builder no longer offers. | Check the listed items and set them again before downloading. |

---

*This guide is licensed under
[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). The config builder
is licensed under GPL-2.0-or-later. See
[`NOTICE`](https://github.com/gcampobello/ICARUS-LEO/blob/main/NOTICE)
for details.*
