# ANAEROBIC DIGESTER DUAL-TANK CONTROL SYSTEM
## README

---

## OVERVIEW

This system runs two anaerobic digesters simultaneously:
- **Tank 1 = BASELINE GROUP** (No control - fixed ratios)
- **Tank 2 = EXPERIMENTAL GROUP** (Adaptive control)

Both tanks receive the same three feed solutions from shared 5L beakers, but the dosing strategy differs.

---

## FEED SOLUTION RECIPES (5L Beakers)

### BEAKER 1: Fermented Urine
| Component | Amount | Notes |
|-----------|--------|-------|
| Human urine | 5 L | Collect and store for 2-4 weeks |
| Storage temperature | Room temp | Allow natural ureolysis |
| Final pH | ~9.1 | Due to ammonia from urea breakdown |
| Final EC | 20-25 mS/cm | High ionic content |

**Properties:**
- pH: 9.1 (alkaline)
- EC: 22.5 mS/cm
- Buffer capacity: 490 meq/L
- Iron: 0.9 mg/L

---

### BEAKER 2: Spirulina + Iron Oxide Suspension
| Component | Amount | Notes |
|-----------|--------|-------|
| Distilled water | 5 L | Base |
| Spirulina powder | 50 g | 10 g/L concentration |
| Iron oxide powder (Feiteite) | 50 g | 10 g/L concentration |

**Preparation:**
1. Add 5L distilled water to beaker
2. Add 50g spirulina powder
3. Add 50g iron oxide powder
4. Stir thoroughly until suspended
5. Stir before each use (settles over time)

**Properties:**
- pH: ~7.8 (slightly alkaline)
- EC: 0.2 mS/cm
- Iron oxide: 10 g/L
- Provides trace metals (Zn, Cu, Mn, Ni)

---

### BEAKER 3: Molasses + Wine Solution
| Component | Amount | Notes |
|-----------|--------|-------|
| Distilled water | 4 L | Base |
| Blackstrap molasses | 500 mL | ~10% dilution |
| Red wine | 500 mL | Any dry red wine |

**Preparation:**
1. Add 4L distilled water to beaker
2. Add 500mL blackstrap molasses
3. Add 500mL red wine
4. Stir until molasses fully dissolved

**Properties:**
- pH: ~4.5 (acidic)
- EC: 1.1 mS/cm
- Carbon source for bacteria
- Provides trace metals from wine

---

## PIN ASSIGNMENTS

### PUMP/RELAY PINS (Digital Output)

| Pin | Relay | Pump Function | Tank | Group |
|-----|-------|---------------|------|-------|
| D2 | Relay 1 | Fermented Urine | Tank 1 | **BASELINE** |
| D3 | Relay 2 | Spirulina + Iron Oxide | Tank 1 | **BASELINE** |
| D4 | Relay 3 | Molasses + Wine | Tank 1 | **BASELINE** |
| D5 | Relay 4 | Fermented Urine | Tank 2 | **EXPERIMENTAL** |
| D6 | Relay 5 | Spirulina + Iron Oxide | Tank 2 | **EXPERIMENTAL** |
| D7 | Relay 6 | Molasses + Wine | Tank 2 | **EXPERIMENTAL** |

**Relay Logic:** Active-HIGH (HIGH = pump ON, LOW = pump OFF)

---

### SENSOR PINS (Analog Input)

#### Tank 1 - BASELINE GROUP
| Pin | Sensor | Units | Notes |
|-----|--------|-------|-------|
| A0 | pH | pH units | Range 0-14 |
| A1 | ORP | mV | Oxidation-reduction potential |
| A2 | EC | mS/cm | Electrical conductivity |
| A3 | MFC | mV | Microbial fuel cell voltage |
| A7 | Temperature | °C | Reactor temperature |

#### Tank 2 - EXPERIMENTAL GROUP
| Pin | Sensor | Units | Notes |
|-----|--------|-------|-------|
| A4 | EC | mS/cm | Electrical conductivity |
| A5 | pH | pH units | Range 0-14 |
| A6 | ORP | mV | Oxidation-reduction potential |
| A8 | Temperature | °C | Reactor temperature |
| A9 | MFC | mV | Microbial fuel cell voltage |

---

## WIRING DIAGRAM (Text)
```
ARDUINO GIGA R1
================

DIGITAL PINS (Relays/Pumps):
┌─────────────────────────────────────────────────────────┐
│  D2 ──── Relay 1 ──── Pump: Urine ──────► TANK 1 (BASELINE)
│  D3 ──── Relay 2 ──── Pump: Spir+FeOx ──► TANK 1 (BASELINE)
│  D4 ──── Relay 3 ──── Pump: Mol+Wine ───► TANK 1 (BASELINE)
│  D5 ──── Relay 4 ──── Pump: Urine ──────► TANK 2 (EXPERIMENTAL)
│  D6 ──── Relay 5 ──── Pump: Spir+FeOx ──► TANK 2 (EXPERIMENTAL)
│  D7 ──── Relay 6 ──── Pump: Mol+Wine ───► TANK 2 (EXPERIMENTAL)
└─────────────────────────────────────────────────────────┘

ANALOG PINS (Sensors):
┌─────────────────────────────────────────────────────────┐
│  TANK 1 - BASELINE:                                     │
│  A0 ◄──── pH Sensor                                     │
│  A1 ◄──── ORP Sensor                                    │
│  A2 ◄──── EC Sensor                                     │
│  A3 ◄──── MFC Voltage                                   │
│  A7 ◄──── Temperature Sensor                            │
├─────────────────────────────────────────────────────────┤
│  TANK 2 - EXPERIMENTAL:                                 │
│  A4 ◄──── EC Sensor                                     │
│  A5 ◄──── pH Sensor                                     │
│  A6 ◄──── ORP Sensor                                    │
│  A8 ◄──── Temperature Sensor                            │
│  A9 ◄──── MFC Voltage                                   │
└─────────────────────────────────────────────────────────┘
```

---

## CONTROL STRATEGY COMPARISON

| Parameter | BASELINE (Tank 1) | EXPERIMENTAL (Tank 2) |
|-----------|-------------------|----------------------|
| Urine ratio | Fixed 11% | Adaptive (5-80%) |
| Spir+FeOx ratio | Fixed 32% | Adaptive (10-50%) |
| Mol+Wine ratio | Fixed 57% | Adaptive (5-80%) |
| HRT (normal) | 24 hours | 24 hours |
| HRT (pH emergency) | 24 hours (no change) | 3 hours (8x faster) |
| pH correction | None | Adjusts ratios when error > 0.3 |

---

## PUMP CALIBRATION

Before running, measure the flow rate of each pump:

1. Place pump intake in water
2. Run pump for exactly 60 seconds
3. Measure volume dispensed (mL)
4. Calculate: `mL_per_sec = volume / 60`

Update these values in the code:
```cpp
const float ML_PER_SEC_URINE_1    = ???;  // Measure and enter
const float ML_PER_SEC_SPIRFEOX_1 = ???;
const float ML_PER_SEC_MOLWINE_1  = ???;
const float ML_PER_SEC_URINE_2    = ???;
const float ML_PER_SEC_SPIRFEOX_2 = ???;
const float ML_PER_SEC_MOLWINE_2  = ???;
```

---

## DOSING CALCULATION

**Total flow per dose (at 24hr HRT, 1L reactor):**
```
Q_total = 1000 mL / (24 hr × 60 min/hr) = 0.694 mL/min
```

**BASELINE doses per minute:**
| Stream | Fraction | Volume |
|--------|----------|--------|
| Urine | 11% | 0.076 mL |
| Spir+FeOx | 32% | 0.222 mL |
| Mol+Wine | 57% | 0.396 mL |
| **Total** | 100% | **0.694 mL** |

**EXPERIMENTAL (Emergency HRT = 3hr):**
```
Q_total = 1000 mL / (3 hr × 60 min/hr) = 5.56 mL/min
```
(8x higher flow rate to rapidly correct pH)

---

## SENSOR CALIBRATION CONSTANTS

Currently configured values:
```cpp
// pH Sensor
PH_SLOPE     = -5.59047
PH_INTERCEPT = 15.835

// EC Sensor
EC_SLOPE_MSPERMV = 0.00864
EC_OFFSET_MS     = 0.033

// ORP Sensor
ORP_OFFSET = -1524.0

// Temperature (TMP36)
TEMP_OFFSET = -0.5
TEMP_SCALE  = 100.0
```

---

## TARGET OPERATING RANGES

| Parameter | Min | Target | Max | Units |
|-----------|-----|--------|-----|-------|
| pH | 6.5 | 7.0 | 7.5 | - |
| EC | 0.2 | 10.0 | 25.0 | mS/cm |
| Iron Oxide | 2.0 | 3.0 | 4.0 | g/L |
| Temperature | 20 | 25 | 35 | °C |

---

## SERIAL OUTPUT FORMAT
```
================================================================================
ANAEROBIC DIGESTER DUAL-TANK CONTROL SYSTEM
Tank 1 = BASELINE (Fixed Ratios) | Tank 2 = EXPERIMENTAL (Adaptive Control)
================================================================================
Time(hr) | Tank | Phase          | Q_urine | Q_sp+fe | Q_mw    | pH     | EC     | ORP      | MFC    | Temp  | HRT
         |      |                | (mL)    | (mL)    | (mL)    |        | mS/cm  | (mV)     | (mV)   | (C)   | (hr)
--------------------------------------------------------------------------------
    0.00 | T1   | normal         |   0.076 |   0.222 |   0.396 |   7.02 |   5.12 |   -285.3 |  425.1 |  24.8 |  24.0
    0.00 | T2   | normal         |   0.104 |   0.208 |   0.382 |   6.98 |   5.08 |   -287.1 |  428.3 |  25.1 |  24.0
```

---

## TROUBLESHOOTING

| Issue | Possible Cause | Solution |
|-------|----------------|----------|
| Pump not running | Relay not triggering | Check relay wiring, verify active-HIGH |
| Wrong volume dispensed | Calibration off | Re-calibrate mL/sec for each pump |
| pH reading wrong | Sensor drift | Recalibrate pH sensor with buffers |
| ORP always same | Sensor not in solution | Check probe immersion |
| No serial output | Baud rate mismatch | Set Serial Monitor to 115200 |

---

## QUICK START CHECKLIST

- [ ] Prepare 5L Beaker 1: Fermented Urine
- [ ] Prepare 5L Beaker 2: Spirulina + Iron Oxide (50g each in 5L water)
- [ ] Prepare 5L Beaker 3: Molasses + Wine (500mL each in 4L water)
- [ ] Wire 6 pumps to relays on pins D2-D7
- [ ] Wire Tank 1 sensors to A0, A1, A2, A3, A7
- [ ] Wire Tank 2 sensors to A4, A5, A6, A8, A9
- [ ] Calibrate all 6 pumps (mL/sec)
- [ ] Calibrate all sensors
- [ ] Fill both reactors with identical starting solution
- [ ] Upload code to Arduino GIGA R1
- [ ] Open Serial Monitor at 115200 baud
- [ ] Start experiment!

---

## FILES

| File | Description |
|------|-------------|
| `DIGESTER_CONTROL_ARDUINO.ino` | Main Arduino control code |
| `BASELINE_no_control.py` | Python simulation - baseline |
| `EXPERIMENTAL_adaptive_control.py` | Python simulation - experimental |
| `README.md` | This file |
