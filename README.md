# Current-Mirror-Loaded Differential Amplifier (5T OTA) — 0.18 µm CMOS

Course: IC Design Lab (EC705), MTech (Research) — VLSI Design, National Institute of Technology Karnataka (NITK), Surathkal. Joint work with Vishwas A Shanbhag, under Dr. Rekha S. Designed and simulated in **Cadence Virtuoso** using the **GPDK 180 nm** CMOS process design kit.

> Note: the report cover page is titled "Assignment 2: Differential Amplifiers"; the source file in this folder is named `Assignment 3 up.pdf`. This README describes the content of that file as submitted.

## Contents

| File | Description |
|---|---|
| `Assignment 3 up.pdf` | Full lab report: objective, design parameters, specification table, hand-calculated transistor sizing for a current-mirror-loaded differential amplifier (5T OTA), the Cadence schematic, DC operating-point dumps, and AC/transient simulation results (gain, gain-bandwidth, slew rate, power dissipation) at both ICMR corners. |

## Objective

> "This assignment aims to simulate and analyse the performance of differential amplifier circuits using matched nMOS and pMOS transistors. The objective is to understand their operating principles, evaluate key characteristics such as gain and explore various differential amplifier configurations..."

A differential amplifier amplifies the difference between two input signals while rejecting common-mode components. The report analyses a basic MOSFET differential amplifier and evaluates its key performance parameters — gain, bandwidth, slew rate, ICMR and power.

## Design Constraints / Assumptions

- Model file: **gpdk180 nm**
- Widths and lengths reported to **2 decimal places**
- Minimum channel length **L = 1 µm**, minimum width **W_min = 0.4 µm**
- Supply voltage **V_DD = 1.8 V**

## Specification Table

| Parameter | Target |
|---|---|
| Gain-Bandwidth | ≥ 5 MHz |
| A_v | ≥ 120 |
| C_L | 12 pF |
| ICMR_max | 1.6 V |
| ICMR_min | 1 V |
| Slew rate | 6 V/µsec |
| Power dissipation | < 0.5 mW |

## Process / Device Parameters used in the hand calculations

```
t_ox   = 4.0 nm
ε_r    = 3.9
ε_0    = 8.85 × 10⁻¹² F/m
ε_ox   = ε_r · ε_0        = 3.4515 × 10⁻¹¹ F/m
C_ox   = ε_ox / t_ox      = 8.62875 × 10⁻³ F/m²
u_n    = 0.046 m²/Vs        V_thn = 0.49 V
u_p    = 0.01  m²/Vs        V_thp = −0.45 V
```

## Circuit Topology

The design is the classical **current-mirror-loaded differential amplifier (5-transistor OTA)**, built and simulated in Virtuoso:

- **N1, N2** — the NMOS source-coupled input pair. N1's gate is driven by one differential input (`V1`), N2's gate by the other (`V2` DC common-mode + `V3` AC test signal). Their sources are tied together at the tail node.
- **P1, P2** — PMOS current-mirror active load. P1 is diode-connected (gate tied to its own drain, which is also N1's drain); this gate is shared with P2, mirroring the current from the N1 branch into the N2 branch. The single-ended output **V_out** is taken at the P2/N2 drain junction, loaded by **C_L = 12 pF**.
- **N3** — the tail current-source transistor, biasing the N1/N2 pair from the shared source node down to ground.
- **N4** — a diode-connected NMOS reference transistor fed by an ideal 72 µA bias current source (`I8`); it mirrors its gate-source bias into N3, setting the tail current. (N4 is a bias-generation device, not part of the core signal path of the 5-transistor OTA — the report's simulated schematic includes it as the on-chip mirror reference for N3.)

Supply `V0 = 1.8 V` (V_DD), all transistors constrained to operate in **saturation**.

## Hand-Calculated Sizing

### Step 0 — Tail current from the slew-rate spec

```
SR = dV0/dt = 6×10⁶ V/s
I_C = C_L · dV0/dt = 6×10⁶ × 12×10⁻¹² = 72 µA
```

So the tail current is set: **I_D,N3 = 72 µA**.

### (a) N1 — saturation / ICMR_max boundary

```
V_d,N1 ≥ V_g,N1 − V_thn = V_in − V_thn ≥ ICMR_max − V_thn = 1.6 − 0.49 = 1.11 V
```

### (b) P1/P2 sizing

```
V_g,P1 = V_d,P1 = V_d,N1 = 1.11 V,   V_s,P1 = 1.8 V  ⇒  V_gs,P1 = 1.11 − 1.8 = −0.69 V

I_DS,P1 = ½ · u_p · C_ox · (W_P1/L_P1) · (|V_gs,P1| − |V_thp|)²  ⇒  W_P1 = W_P2 ≈ 20 µm
```

### (c) N1 — sizing from g_m / GBW requirement

```
A_v = g_m,N1 · (r_on ‖ r_op)
BW  = 1 / [2π·(r_on ‖ r_op)·C_L]
A_v · BW ≥ 5 MHz  ⇒  g_m,N1 / (2π·C_L) ≥ 5 MHz
g_m,N1 ≥ 0.377 mA/V  →  take g_m,N1 = 0.4 mA/V

2·I_d,N1 / (V_gs,N1 − V_thn) = g_m,N1  ⇒  V_gs,N1 = V_thn + 2·I_d,N1/g_m,N1 = 0.67 V

W_N1 = W_N2 = 2·I_D,N1·L_N1 / [u_n·C_ox·(V_GS,N1 − V_thn)²]  ⇒  W_N1 = W_N2 = 5.6 µm
```

### (d) N2 — ICMR_min boundary (sets V_dsat,N3)

```
I_DS,N2 = ½·u_n·C_ox·(W/L)·(V_gs,N2 − V_thn)²  ⇒  V_gs,N2 = 0.67 V

V_in > V_gs,N2 + V_dssat,N3  ⇒  ICMR_min > V_gs,N2 + V_dssat,N3  ⇒  V_dssat,N3 < 1 − 0.67 = 0.33 V
```

Take **V_dssat,N3 = 0.3 V**.

### (e) N3 sizing

```
I_DS,N3 = ½·u_n·C_ox·(W_N3/L_N3)·(V_dssat,N3)²  ⇒  W_N3 = W_N4 = 4 µm
```

## Hand-Calculated (Initial) Sizing Table

| Device | Width |
|---|---|
| N1, N2 | 5.6 µm |
| N3, N4 | 4 µm |
| P1, P2 | 20 µm |

### Post-layout tuning (as noted in the report)

> "N3, N4 transistor current mirror was giving current of 65 µA, so to get 72 µA width of N3, N4 was varied to 20 µm to get current approximate to 71 µA, and N1, N2 was varied to 10 µm to get a gain of more than 120, whereas I got around 132."

I.e., after the initial hand calculation, the tail-mirror devices (N3, N4) were widened from 4 µm to **20 µm** to bring the tail current up to ≈71 µA (target 72 µA), and the input pair (N1, N2) was widened from 5.6 µm to **10 µm** to push the gain above the ≥120 spec, landing at ≈132.

## DC Operating Points (Cadence op-point analysis)

Per-device operating-point dumps (`gm`, `gds`, `rout`, `self_gain`, `Vgs`, `Vds`, `Vdsat`, etc.) were extracted for all six devices. Consolidated:

| Operating point | N1 | N2 | N3 | N4 | P1 | P2 |
|---|---|---|---|---|---|---|
| g_m,eff = g_m+g_mb | 401.68 µS | 401.68 µS | 1115.90 µS | 1153.84 µS | 355.33 µS | 355.33 µS |
| r_out | 761.973 kΩ | 761.973 kΩ | 73.1782 kΩ | 255.534 kΩ | 633.414 kΩ | 633.414 kΩ |
| I_d | 34.9853 µA | 34.9853 µA | 69.9707 µA | 72 µA | 34.9902 µA | 34.9902 µA |
| V_gs | 0.738273 V | 0.738273 V | 0.738273 V* | 0.584153 V | −0.683695 V | −0.683695 V |
| V_ds | 0.854578 V | 0.854578 V | 0.261727 V | 0.584153 V | −0.683695 V | −0.683695 V |
| V_dsat | 0.209809 V | 0.209809 V | 0.148900 V | 0.148955 V | −0.214609 V | −0.214609 V |

\* As printed in the summary table in the PDF. The individual per-device operating-point dump for N3 (self_gain 63.4628, r_out 73.1782 kΩ) instead lists V_gs = 0.584153 V, matching N4 — consistent with N3/N4 forming the tail-current mirror pair (V_ds,N3 = 0.261727 V, self-gain 63.46, distinct from the input-pair values). This is reproduced here exactly as it appears in the source document.

Other extracted per-device figures: self-gain of N1/N2 = 242.173, N3 = 63.4628, N4 = 229.16, P1/P2 = 169.591; V_early of N1/N2 = 26.6579, N3 = 5.12033, N4 = 18.3984, P1/P2 = 22.1633.

## Simulation Results

AC, transient (slew-rate) and power-dissipation simulations were run at both ICMR corners.

### At ICMR_min (V_icm = 1 V)
- **Gain (AC response):** 132.147 (linear, V/V) — marker read directly off the magnitude trace.
- **Slew rate:** `slewRate(VT("/Vout")) = 3.750E6` → 3.75 V/µs
- **Gain-Bandwidth:** 5.208 MHz
- **Bandwidth (−3 dB):** 39.201 kHz
- **Power dissipation:** 246.5 µW (average, from the transient current-consumption waveform)

### At ICMR_max (V_icm = 1.6 V)
- **Gain (AC response):** 84.788
- **Slew rate:** `slewRate(VT("/Vout")) = 2.100M` → 2.1 V/µs
- **Gain-Bandwidth:** 5.433 MHz
- **Bandwidth (−3 dB):** 63.655 kHz
- **Power dissipation:** 246.5 µW

Both corners' AC-response and gain-bandwidth traces show a single dominant pole rolling off from the low-frequency gain plateau (flat below ~10s of kHz, −20 dB/decade thereafter); the transient power-dissipation waveform shows a periodic (input-signal-rate) ripple/spike pattern around the ~246.5 µW average level.

### Obtained Results (as tabulated in the report)

| Parameter | ICMR_min | ICMR_max |
|---|---|---|
| Gain | 132.147 | 84.788 |
| Slew Rate | 3.75 V/µs* | 2.1 V/µs* |
| Gain-Bandwidth | 5.208 MHz | 5.433 MHz |
| Bandwidth | 39.201 kHz | 63.655 kHz |
| Power Dissipation | 246.5 µW | 246.5 µW |

\* Printed in the report as "V/µA" — a unit slip; the underlying Cadence `slewRate()` expression values (3.750E6 and 2.100M V/s) confirm these are V/µs figures.

### Spec compliance (against the target table)

- **Gain-Bandwidth ≥ 5 MHz:** met at both corners (5.208 MHz, 5.433 MHz).
- **A_v ≥ 120:** met at ICMR_min (132.147); **not met** at ICMR_max (84.788).
- **Slew rate ≥ 6 V/µs:** **not met** at either corner (3.75 V/µs, 2.1 V/µs).
- **Power dissipation < 0.5 mW:** comfortably met at both corners (246.5 µW).
- **C_L = 12 pF, ICMR 1–1.6 V:** used as the simulation conditions themselves.

## Conclusion

> "Current Mirror Loaded Differential Amplifiers are designed with given specifications"

The 5T OTA was hand-designed from the slew-rate/tail-current relation and the g_m–GBW relation, sized (with post-hand-calc tuning of the input pair and tail mirror widths), and verified in Cadence Virtuoso (GPDK180) via DC operating-point, AC, and transient (slew-rate, power) analyses at both ICMR corners.
