# MRI-Phantom-Simulation-UTE-ZTE-SWIFT-
# Examining MRI Methods for Imaging Short T2

## Overview

Conventional gradient- and spin-echo MRI can't see tissues with very short T2 (bone, tendon, ligament, dental enamel) because the signal has decayed to nothing by the time a standard echo time is reached. This project reviews three sequence families built to solve that problem — **UTE**, **ZTE**, and **SWIFT** — and includes a Python simulation comparing them on a synthetic multi-T2 phantom.

**Applications:** musculoskeletal imaging (bone has short T2), dentistry (teeth and adjacent soft tissue).

## Methods Compared

### Ultra-short Echo Time (UTE)
- Gradients switch on immediately after RF excitation; TE is 100–1000× shorter than conventional gradient echo.
- Half-sinc excitation pulse lets readout start even sooner.
- Long-T2 tissue must be suppressed for contrast — three common variants:
  - Dual-echo UTE with echo subtraction
  - UTE with long-T2 saturation
  - Adiabatic inversion recovery
- k-space traversal: radial, spiral, or cone.

### Zero Echo Time (ZTE)
- Gradients ramp up **before** the RF pulse, so encoding is effectively underway during excitation (TE ≈ 0). Silent scanning.
- 3D radial, center-out k-space sampling; no phase-encode gradient needed.
- Can't sample the very center of k-space directly (dead-time gap) — filled in with hybrid trajectories: **PETRA**, **WASPI**, **HYFI**.
- RF pulses must be very short, which caps the achievable flip angle (~4° max) and can introduce slice-selection/blurring artifacts.

### Sweep Imaging with Fourier Transform (SWIFT)
- Frequency-swept ("chirp") RF excitation overlaps with acquisition, achieving TE ≈ 0 via time-interleaving — mechanistically distinct from ZTE.
- Cross-correlation separates the true spin response from the overlapping excitation/acquisition signal.
- Radial k-space sampling; gradient direction steps each TR.
- Quiet, robust to off-resonance and motion artifacts, no dead-time penalty.

### Comparison Summary

| | UTE | ZTE | SWIFT |
|---|---|---|---|
| Slice selection | Possible | Non-selective (3D only) | Non-selective (3D only) |
| Flip angle range | Wide (2–40°) | Limited (~4° max) | Wide, variety of angles |
| Off-resonance behavior | Blurring/ringing | Blurring/ringing | Swept RF encodes off-resonance info, reducing artifacts |
| k-space center | Collected in-trajectory | Needs extra scan time (dead-time gap) | Collected during sweep, no dead time |
| Gradient demands | Near system limits | Quiet, gentle on gradients | Quiet, no rapid switching |

## Simulation

`MRI UTE:ZTE:SWIFT Phantom Simulator.ipynb` implements a simplified comparison of the three sequences (plus conventional GRE as a baseline) on synthetic phantoms.

**Approach:**
1. **Phantom generation** — synthetic 2D phantoms (a "smiley face" and a "Mickey Mouse" design) built from geometric regions, each assigned a proton density and T2 value spanning ultra-short (~0.5–1 ms) to long (~80 ms) T2, to mimic bone/tendon vs. soft tissue.
2. **Signal model** — two approaches are used across the notebook:
   - A full Bloch-equation simulation (hard-pulse excitation + free precession/relaxation to TE) via a `bloch` solver.
   - A simplified closed-form decay model, `S = ρ · exp(−TE / T2)`, evaluated at representative TEs for each method (GRE: 5 ms, UTE: 80 µs, ZTE: 5 µs, SWIFT: 15 µs).
3. **k-space sampling + reconstruction** — each method's image is transformed to Cartesian k-space (FFT), resampled along **radial** and **spiral** trajectories (via `scipy.interpolate.griddata`) to emulate real acquisition geometry, then re-gridded and reconstructed with an inverse FFT.
4. **Quantification** — mean signal is compared between short-T2 and long-T2 regions per method to show how much better UTE/ZTE/SWIFT preserve short-T2 signal relative to conventional GRE.

**Run it:**
```bash
pip install numpy matplotlib scipy
jupyter notebook "MRI UTE:ZTE:SWIFT Phantom Simulator.ipynb"
