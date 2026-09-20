# ICARUS-LEO

🚧 **Work in Progress**
This project is currently under active development.

This repository accompanies the paper *"Understanding In-Network Caching for LEO
Satellite Networks: A Physical-Layer Perspective"* by G. Campobello and M. Amadeo
(submitted to Computer Networks, 2026). The paper investigates in-network
caching for LEO satellite networks from a physical-layer perspective and presents
**ICARUS-LEO**, a simulator that extends the [Icarus](https://github.com/icarus-sim/icarus) in-network-caching framework
(Saino, Psaras and Pavlou, SIMUTOOLS 2014) to content delivery over LEO satellite
constellations.


## Contents

This repository provides documentation, worked examples and validation material:

- **Documents**
  - **Architecture** (docs/ICARUS-LEO-Architecture.pdf)— what the simulator does and its architecture (in docs/ICARUS-LEO-Architecture.pdf) 
  - **User guide** — how to use the simulator (in docs/ICARUS-LEO-UserGuide.md) 
- **Examples** — worked examples, each with the exact configuration and the
  resulting interactive session viewer.
- **Validation Material** — the simulator's output
  compared against independent analytical models, as reported in the paper.

## Accessing the simulator

A web service that will let you run ICARUS-LEO online is under development. In the
meantime, you can request results for your own scenarios: prepare a configuration
with the config builder, as described in the user guide
([`docs/ICARUS-LEO-UserGuide.md`](docs/ICARUS-LEO-UserGuide.md)), and email it, as
an attachment, to the address below.

Prof. G. Campobello, Department of Engineering, University of Messina, Italy.
Email: [gcampobello@unime.it](mailto:gcampobello@unime.it)

Requests are reviewed individually and accepted where the computational effort
involved is limited. In response, you will receive a link to download the output
produced by the simulator (a `.pkl` file and/or an interactive session viewer).

## How to cite

If you use ICARUS-LEO, please cite the accompanying paper (see the reference at
the top). Citation metadata is provided in [`CITATION.cff`](CITATION.cff); on
GitHub you can use the "Cite this repository" button.

## Licence

- The source code in this repository is licensed under GPL-2.0-or-later
  (see [`LICENSE`](LICENSE)), the same licence as
  [Icarus](https://github.com/icarus-sim/icarus), which ICARUS-LEO extends.
- The documentation and text are licensed under
  [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).
- Content reproduced from the accompanying paper, such as figures and tables,
  follows its copyright and the publisher's sharing policy.

See [`NOTICE`](NOTICE) for details.
