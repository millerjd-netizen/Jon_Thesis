# Digester Tank 1 — Experiment Repository

## Repository Structure

### `experiment_data/`
Sensor data from all 4 experiments, organized into 4 folders (`Experiment_1` through `Experiment_4`). Each folder contains time-series CSVs logged from the digester system, including pH, EC, ORP, temperature, flow rates (kombucha, spirulina, urine), VFA, stress state/cycle, and MFC voltage.

### `experiments_arduino_code/`
Arduino code that was run on the digester system during all 4 experiments.

### `matrix_training_scripts/`
Python scripts (Jupyter notebooks) used to train the **B** and **A** matrices for the system model.

### `scripts_training_data/`
Arduino code that was run to generate the data used to construct the **B** and **A** matrices.

### `results_analysis/`
Statistical analysis performed on the experimental data, including MFC voltage analysis across experiments.

---

### `graveyard/`
Deprecated or unused files kept for reference.
