# Digester Tank 1 — Experiment Repository
## Repository Structure
### `experiment_data/`
Sensor data from all 3 experiments, organized into 3 folders (`Experiment_1` through `Experiment_3`). Each folder contains time-series CSVs logged from the digester system, including pH, EC, ORP, temperature, flow rates (kombucha, spirulina, urine), VFA, stress state/cycle, and MFC voltage.
**Experiment Windows (all times EST, UTC−5):**
| Experiment | Start | End | Duration | Firmware | MFC Sensor |
|------------|-------|-----|----------|----------|------------|
| Experiment 1 | Mar 29, 11:05 PM | Mar 31, 9:12 AM | ~34 hrs | v13 | Relay (main Arduino) |
| Experiment 2 | Apr 3, 2:56 AM | Apr 4, 8:38 AM | ~30 hrs | v21 | Chickie Arduino |
| Experiment 3 | Apr 4, 4:00 PM | Apr 5, 4:00 PM | 24 hrs | v22 | Chickie Arduino |
> **Note:** Experiment 1 MFC uses `cloud_MFC_mV` (relay-isolated on main Arduino). Spike readings >600 mV are filtered before analysis. ORP sensor was recalibrated on 2 April 2026; Experiment 1 ORP is not directly comparable to Experiments 2 and 3.
---
### `experiments_arduino_code/`
Arduino firmware that ran on the digester system during each experiment.
**Experiment Mapping:**
- **Experiment 1:** `experiment_2_hours_3.31` (v13 — pH-setpoint acid protocol, 90-min phases, urine + apple juice)
- **Experiment 2:** `experiment_4.4` (v21 — extended 120-min phases)
- **Experiment 3:** `experiment_4.4_new` (v22 — ratio-matched baseline dosing)
---
### `matrix_training_scripts/`
Python scripts (Jupyter notebooks) used to identify the **A** and **B** matrices for the discrete-time state-space model.
---
### `scripts_training_data/`
Arduino firmware run to generate the excitation data used to identify the **A** and **B** matrices.
---
### `results_analysis/`
Statistical analysis and post-hoc observer reconstruction of experimental data. Contains:
- `MFC_Statistical_Analysis.ipynb` — Welch's t-test results for MFC voltage, ORP, and pH across all 3 experiments (Table 4.11 in thesis)
- `kalman_reconstruction.ipynb` — Post-hoc reconstruction of the full internal observer state (`x_obs`) for all 3 experiments. The observer state was not logged to the cloud in real time; this notebook replays the deterministic Luenberger (Experiment 1) and Kalman filter (Experiments 2 and 3) equations against the recorded sensor and pump dose time series to recover `x_VFA`, `x_EC`, `x_pH`, the time-varying Kalman gain matrix, and the covariance diagonal. Outputs one validated CSV per experiment. See Section 3.9 of the thesis for methodology.
- `Experiment_1.zip`, `Experiment_2.zip`, `Experiment_3.zip` — pre-packaged sensor data used by the analysis notebooks
---
### `graveyard/`
Deprecated or unused files kept for reference.
