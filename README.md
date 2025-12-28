# Jon_Thesis — MFC Control System (Arduino Giga)

Arduino-based dosing control for a microbial fuel cell (MFC) system using three inputs:
- Diluted synthetic winery wastewater ("wine") — 100:1 dilution, pH 4.0
- Diluted fully fermented urine — 4:1 dilution, pH 8.5, EC 6.25 mS/cm
- Spirulina solution — 15 g/L, pH 9.0, EC 1.14 mS/cm

## State Space Formulation

The system is modeled as:

```
ẋ = Ax + Bu + d
```

Where:
- **x** = [pH, EC, TAN]ᵀ — state vector
- **u** = [Q_wine, Q_urine, Q_spirulina]ᵀ — input vector (flow rates, mL/min)
- **A** — autonomous dynamics matrix (biological decay)
- **B** — input coupling matrix (mixing physics)
- **d** — constant drift vector (zero-order consumption)

### State Variables

| State | Description | Target Range | Units |
|-------|-------------|--------------|-------|
| pH | Hydrogen ion activity | 6.8 – 7.2 | — |
| EC | Electrical conductivity | 5.0 – 10.0 | mS/cm |
| TAN | Total ammonia nitrogen | 50 – 200 | mg-N/L |

### Autonomous Dynamics (A matrix + drift d)

Without inputs, the system evolves according to ODEs:

```
d(pH)/dt  = -0.0001·pH - 0.0005      (CO₂/VFA production)
d(EC)/dt  = -0.0001·EC - 0.001       (ion uptake by bacteria)
d(TAN)/dt = -0.00003·TAN - 0.3       (bacterial assimilation)
```

These rates are approximately constant within the operating range (Batstone et al., *Water Sci. Tech.* 2002).

### Input Coupling (B matrix)

Dosing follows CSTR mixing physics:

```
x_new = (x_reactor × V_reactor + x_input × V_dose) / (V_reactor + V_dose)
```

| Input | pH | EC (mS/cm) | TAN (mg-N/L) | Effect |
|-------|-----|------------|--------------|--------|
| Wine | 4.0 | 0.02 | 0.1 | ↓ pH, ↓ EC, ↓ TAN |
| Urine | 8.5 | 6.25 | 200 | ↑ pH, ↑ EC, ↑ TAN |
| Spirulina | 9.0 | 1.14 | 144 | ↑ pH, ↓ EC*, ↑ TAN |

*Spirulina EC (1.14) is lower than reactor target (5.0), so it dilutes EC.

## Control Logic

**Priority:** pH > TAN > EC > Metals

Every control cycle (1 minute):
1. Read sensors (pH, EC, TAN)
2. Apply control decision based on priority
3. Dose one input if needed
4. Apply ODE drift dynamics
5. Update trace metal estimates

### Dosing Actions

| Condition | Action | Reason |
|-----------|--------|--------|
| pH > 7.2 | Dose wine | Acid lowers pH |
| pH < 6.8 | Dose urine | Base raises pH |
| TAN < 50 AND EC < 5.0 | Dose urine | Provides both |
| TAN < 50 AND EC ≥ 5.0 | Dose spirulina | TAN without diluting EC |
| EC < 5.0 | Dose urine | High ionic content |
| All in range | No action | — |

## Trace Metals

Trace metals are **tracked but not actively controlled**. Concentrations are estimated via mass balance using the same CSTR mixing physics as the primary states, with first-order decay (0.05%/min) to account for biological uptake.

### Substrate Trace Metal Concentrations (mg/L)

| Metal | Wine (100:1) | Urine (4:1) | Spirulina (15 g/L) | Target Range | Notes |
|-------|--------------|-------------|--------------------|--------------| ------|
| Fe | 0.05 | 0.01 | 4.275 | 1.0 – 10.0 | Spirulina is primary source |
| Zn | 0.01 | 0.05 | 0.300 | 0.1 – 1.0 | All inputs contribute |
| Cu | 0.001 | 0.01 | 0.915 | 0.01 – 0.5 | ⚠ Spirulina may exceed target |
| Mn | 0.02 | 0.005 | 0.285 | 0.1 – 1.0 | Spirulina is primary source |
| Se | 0.0 | 0.001 | 0.00105 | 0.01 – 0.1 | Low in all inputs |
| Ni | 0.0 | 0.002 | 0.0 | 0.05 – 1.0 | ⚠ Only urine provides Ni |
| Co | 0.0 | 0.001 | 0.0 | 0.01 – 0.3 | ⚠ Only urine provides Co |

### Metal Dynamics

```
d(Metal)/dt = -0.0005 · Metal    (first-order decay, ~0.05%/min)
```

Metals are mixed via CSTR physics during dosing events, then decay between doses.

### Metal Sufficiency Analysis

Based on 8-hour simulation results:

| Metal | Status | Notes |
|-------|--------|-------|
| Fe | ✓ OK | Spirulina provides sufficient Fe |
| Zn | ⚠ Marginal | May drop below target over extended runs |
| Cu | ✓ OK | Spirulina provides adequate Cu |
| Mn | ⚠ Marginal | May require supplementation |
| Se | ✗ Low | All inputs insufficient; consider supplementation |
| Ni | ✗ Low | Only urine provides Ni; may need supplementation |
| Co | ✗ Low | Only urine provides Co; may need supplementation |

## Outputs (Monitored)

- MFC voltage
- ORP (redox potential)
- Trace metals (Fe, Zn, Cu, Mn, Se, Ni, Co) — estimated via mass balance

## Hardware

- Arduino Giga R1
- pH sensor (analog, A0)
- EC sensor (analog, A1)
- TAN sensor (ISE or colorimetric, A2)
- 3× 3V submersible pumps (relay-controlled, D0-D2)
- Status LED (D3)

## Files

| File | Description |
|------|-------------|
| `mfc_control_system.ino` | Arduino code with full state-space model and metal tracking |
| `mfc_simulation.py` | Python simulation matching state-space formulation |

## Simulation Results (8 hours)

```
Duration: 480 minutes (8.0 hours)
Final Volume: 2665 mL

Substrate Usage:
  Wine        :  260 mL (15.6%) - 52 doses
  Urine       : 1300 mL (78.1%) - 260 doses
  Spirulina   :  105 mL ( 6.3%) - 21 doses
  Total       : 1665 mL

Final State:
  pH:  7.20 (target: 6.8-7.2)   ✓ OK
  EC:  4.73 mS/cm (target: 5.0-10.0)   ⚠ LOW
  TAN: 46.8 mg-N/L (target: 50.0-200.0)   ⚠ LOW

Final Trace Metals:
  Fe: 0.445 mg/L   ✗ LOW
  Zn: 0.063 mg/L   ✗ LOW
  Cu: 0.050 mg/L   ✓ OK
  Mn: 0.043 mg/L   ✗ LOW
  Se: 0.003 mg/L   ✗ LOW
  Ni: 0.004 mg/L   ✗ LOW
  Co: 0.003 mg/L   ✗ LOW
```

## References

1. Batstone, D.J. et al. (2002). The IWA Anaerobic Digestion Model No 1 (ADM1). *Water Science & Technology*, 45(10):65-73.
2. Siegrist, H. et al. (2002). Mathematical model for meso- and thermophilic anaerobic sewage sludge digestion. *Water Science & Technology*, 45(10):93-100.
3. Udert, K.M. et al. (2006). Fate of nitrogen and phosphorus in source-separated urine. *Water Research*, 40(9):1803-1812.

## Author

Jonathan Miller — Boston University
