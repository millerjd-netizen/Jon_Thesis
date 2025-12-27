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
d(pH)/dt  = -0.0005 /min      (CO₂/VFA production)
d(EC)/dt  = -0.001 mS/cm/min  (ion uptake by bacteria)
d(TAN)/dt = -0.3 mg-N/L/min   (bacterial assimilation)
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

Every control cycle (1–5 minutes):
1. Read sensors (pH, EC, TAN)
2. Apply control decision based on priority
3. Dose one input if needed

### Dosing Actions

| Condition | Action | Reason |
|-----------|--------|--------|
| pH > 7.2 | Dose wine | Acid lowers pH |
| pH < 6.8 | Dose urine | Base raises pH |
| TAN < 50 AND EC < 5.0 | Dose urine | Provides both |
| TAN < 50 AND EC ≥ 5.0 | Dose spirulina | TAN without diluting EC |
| EC < 5.0 | Dose urine | High ionic content |
| All in range | No action | — |

## Outputs (Monitored, Not Controlled)

- MFC voltage
- ORP (redox potential)
- Trace metals (Fe, Zn, Cu, Mn, Se, Ni, Co)

## Trace Metals

Spirulina (15 g/L) provides:

| Metal | Concentration | Target | Status |
|-------|---------------|--------|--------|
| Fe | 4.28 mg/L | 1.0–10.0 | ✓ OK |
| Zn | 0.30 mg/L | 0.1–1.0 | ✓ OK |
| Cu | 0.92 mg/L | 0.01–0.5 | ⚠ High |
| Mn | 0.29 mg/L | 0.1–1.0 | ✓ OK |
| Ni | 0 mg/L | 0.05–1.0 | ✗ None |
| Co | 0 mg/L | 0.01–0.3 | ✗ None |

Note: Spirulina contains no Ni or Co. These would require supplementation if needed.

## Hardware

- Arduino Giga R1
- pH sensor (analog)
- EC sensor (analog)
- TAN sensor (ISE or colorimetric)
- 3× peristaltic pumps (relay-controlled)

## Files

- `MFC CONTROL SYSTEM - Arduino Giga/` — Arduino code
- `mfc_simulation.py` — Python simulation matching state-space formulation

## References

1. Batstone, D.J. et al. (2002). The IWA Anaerobic Digestion Model No 1 (ADM1). *Water Science & Technology*, 45(10):65-73.
2. Siegrist, H. et al. (2002). Mathematical model for meso- and thermophilic anaerobic sewage sludge digestion. *Water Science & Technology*, 45(10):93-100.
3. Udert, K.M. et al. (2006). Fate of nitrogen and phosphorus in source-separated urine. *Water Research*, 40(9):1803-1812.

## Author

Jonathan Miller — Boston University
