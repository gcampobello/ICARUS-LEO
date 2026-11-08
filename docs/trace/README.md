# Loss traces

This folder contains the real loss trace used in Case Study 2 of the
accompanying paper. In the config builder, select the `TRACE` sub-model of the
`COMPOSITE` loss model for a link type and write the file name in
`trace_npz_path`.

| File | Description | Packets | Losses |
|---|---|---|---|
| `LEO_downlink_loss-Ger0804-170001-1h.npz` | Starlink downlink, Germany, about one hour, rainy conditions | 333,176 | 1,596 (0.48%) |

## Origin

The trace was derived from the LENS dataset
(<https://github.com/clarkzjw/LENS>) by the authors of:

> T. Wang, T. Liu, Y. Li, J. Zhao, R. Gao, S. Wu and J. Pan, "Packet Loss
> Modeling and Forward Erasure Correction for LEO Satellite Networks,"
> *IEEE Transactions on Communications*, 2026.

It was provided to us by T. Liu as a text file, one per-packet outcome per line
(1 = lost, 0 = received), and is published here with the authors' permission.
We only converted it to the NumPy `.npz` format, to compress it and to match the
input format of the simulator; the sequence of outcomes is unchanged.

## Format

A `.npz` file with a single array, `data`, of type `uint8`: a one-dimensional
sequence of per-packet outcomes, where 1 means lost and 0 means received. The
simulator does not use timestamps: each link reads the sequence from an
independent random offset, and every packet transmitted on the link consumes
the next entry.

```python
import numpy as np

trace = np.load("LEO_downlink_loss-Ger0804-170001-1h.npz")["data"]
print(trace.size, trace.sum(), trace.mean())   # 333176 1596 0.00479...
