# MFC Feedback Control Simulation

Python simulation for Master's thesis on Microbial Fuel Cell (MFC) control systems. Compares experimental feedback controller vs bang-bang baseline controller.

## System Overview

- **Reactor Volume**: 1 L
- **Flow Rate**: 45 mL/L/h (0.75 mL/min)
- **HRT**: ~22 hours
- **Simulation Duration**: 2 days (2880 minutes)
- **Perturbation**: 10g citric acid at day 1

## Inputs (3 Substrates)

| Substrate | pH | EC (mS/cm) | COD (mg/g) | Notes |
|-----------|-----|------------|------------|-------|
| Fermented Urine | 9.3 | 45.0 | 3.0 | Alkaline, very high EC |
| Fermented Winery WW | 3.5 | 8.0 | 200.0 | Acidic, lowest EC |
| Lysed Spirulina | 7.2 | 12.0 | 1100.0 | Neutral, high trace metals |

## Target Ranges

| Parameter | Target Range |
|-----------|-------------|
| pH | 6.8 - 7.2 |
| EC | 5.0 - 10.0 mS/cm |
| ORP | -300 to -100 mV |
| Trace Metals | 2.0 - 5.0 mg/L |
| Voltage | > 200 mV |

## Control Strategies

### Bang-Bang (Control Group)
- Only doses when parameter is OUT OF RANGE
- Simple threshold-based switching
- No proportional control

### Feedback (Experimental Group)
- Priority-based decision making
- Considers EC impact before dosing urine
- Balances pH, EC, and trace metal targets

## Output

Arduino-style serial monitor output every minute:
```
[00d 01h 30m] t=90
  CTRL: pH=6.95 EC=7.02 ORP=-186mV T=25.0C V=185mV
  CTRL_DOSE: urine=0.0mL wine=0.0mL spir=0.8mL
  EXP:  pH=7.01 EC=6.85 ORP=-195mV T=25.1C V=190mV
  EXP_DOSE:  urine=0.0mL wine=0.0mL spir=0.8mL
  STATUS: CTRL=OK EXP=OK
```

## Summary Table

At end of simulation, prints:
- Total power (mWh)
- Final pH, EC, ORP, voltage vs target ranges
- Trace metal breakdown (Fe, Mn, Co, Ni, Cu, Zn, Se)
- Pump engagement count (times activated)
- Total substrate used (mL)

## Usage

```bash
python mfc_simulation.py
```

## Configuration

Edit these parameters in `run_simulation()`:
- `duration_minutes`: Simulation length (default 2880 = 2 days)
- `perturbation_time`: When to add citric acid (default 1440 = day 1)
- `perturbation_mass`: Grams of citric acid (default 10.0)
- `print_interval`: Output frequency (default 1 = every minute)

## Physics Equations

- **ORP**: ORP ≈ -200 - 59·(pH - 7) + 5·ln(EC)
- **Voltage**: Quadratic pH penalty around optimum (7.0)
- **pH-EC Coupling**: β = κ·EC when ORP > -100 mV AND pH < 6.8
- **Trace Metal Decay**: First-order kinetics with input additions

## Author

Jonathan Miller - Master's Thesis
