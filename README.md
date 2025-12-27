# Jon_Thesis — MFC Control System (Arduino Giga)

Arduino-based dosing control for a microbial fuel cell (MFC) system using three inputs:
- Diluted synthetic winery wastewater (“wine”)
- Diluted fully fermented urine
- Diluted dried Spirulina

The controller prioritizes maintaining safe chemical operating ranges using measured state variables:
- pH
- EC (mS/cm)
- TAN (Total Ammonia Nitrogen, mg-N/L)

Outputs (monitored separately / not directly controlled here):
- MFC voltage
- ORP (redox potential)

## Control logic (high-level)
**Priority:** pH > TAN > EC

Every 5 minutes:
1. Read sensors (pH, EC, TAN)
2. Print status + flags (LOW/HIGH/OK)
3. Dose one input based on priority and error magnitude

### Dosing actions
- **pH high:** dose winery wastewater (acidify / dilute)
- **pH low:** dose urine (alkalinize)
- **TAN low:** dose Spirulina (nitrogen source)
- **TAN high:** dose winery wastewater (dilution)
- **EC low:** dose urine (ionic strength)
- **EC high:** dose winery wastewater (dilution)

## Hardware assumptions
- Arduino Giga
- Relay-controlled pumps (active HIGH)
- Analog sensors: pH, EC, TAN (connected to A0/A1/A2)

## Pin mapping
| Function | Pin |
|---|---|
| Wine pump relay | D0 |
| Urine pump relay | D1 |
| Spirulina pump relay | D2 |
| Status LED | D3 |
| pH sensor | A0 |
| EC sensor | A1 |
| TAN sensor | A2 |

## Parameters to calibrate
Before running, update these based on your sensor calibration:
- `PH_SLOPE`, `PH_OFFSET`
- `EC_SLOPE`, `EC_OFFSET`
- `TAN_SLOPE`, `TAN_OFFSET`

Also review:
- setpoint ranges (`PH_MIN/MAX`, `EC_MIN/MAX`, `TAN_MIN/MAX`)
- dosing interval (`DOSE_INTERVAL`)
- pump rate (`PUMP_RATE`)
- gains (`K_PH`, `K_EC`, `K_TAN`)

## Safety notes
This code is a prototype controller. Add your own safety interlocks as appropriate (max daily dosing limits, tank level sensing, fault handling, watchdog behavior, etc.) before unattended operation.

## How to use
1. Open `arduino/MFC_Control_System/MFC_Control_System.ino` in the Arduino IDE
2. Select Arduino Giga board + correct COM port
3. Upload
4. Monitor via Serial at 115200 baud
