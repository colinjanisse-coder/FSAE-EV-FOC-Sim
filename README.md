# FSAE EV High Voltage Powertrain — FOC Simulation
**University of Windsor | ELEC-4000 Capstone | Team 20**

Field-Oriented Control (FOC) simulation for the Windsor FSAE Electric Vehicle powertrain.
Built in MATLAB/Simulink with LapSim data as the driving input.

---

## System Overview

| Component | Specification |
|---|---|
| Motor | EMRAX 228-HV Liquid Cooled (axial flux PMSM) |
| Inverter | DTI HV-500LC |
| Battery Pack | 112s5p — Samsung INR18650P28A (~403 V nominal, ~470 V max) |
| Encoder | RLS RM44 (absolute SSI) |
| Gear Ratio | 4.3:1 |
| Wheel Radius | 0.2032 m |

---

## Motor Parameters (EMRAX 228-HV LC)

| Parameter | Value |
|---|---|
| Rs (phase resistance @ 25°C) | 15.48 mΩ |
| Ld | 177 µH |
| Lq | 183 µH |
| ψ_f (flux linkage) | 0.094 Wb |
| Pole pairs (p) | 10 |
| Peak torque | 220 Nm |
| Continuous torque (LC) | 112 Nm |
| Peak current (inverter limit) | 235 A_rms |
| Max speed | 5500 RPM |
| Rotor inertia | 0.02521 kg·m² |

> **Note:** Characteristic current (ψ_f / Ld ≈ 531 A_peak) far exceeds the inverter limit
> (~332 A_peak). Full flux cancellation is not achievable — torque drops off above ~2500–3000 RPM
> in the field weakening region.

---

## Repository Structure

```
/
├── motor_params_init.m        # Run this first — loads all parameters to workspace
├── NewTable_Copy.slx          # Main Simulink FOC model
├── README.md
└── .gitignore
```

> LapSim `.mat` data files are **not included** in this repo (file size).
> Load your own OpenLAP output and update the `lapsim_file` variable in `motor_params_init.m`.

---

## Control Architecture

- **Current control:** Field-Oriented Control (FOC) with dq-frame PI controllers
- **Speed input:** `engine_speed` from LapSim fed directly as `w_mech` (quasi-static assumption)
- **Id strategy:** Id = 0 below base speed; negative Id injection for field weakening
- **MTPA:** Deferred (future improvement — constrained by simulation timestep)
- **Position feedback:** RLS RM44 encoder → electrical angle `θ_e = p × θ_mech`

### Simulink Model Inputs (from LapSim)

| Signal | Description |
|---|---|
| `engine_speed` | Motor speed [RPM] |
| `engine_torque` | Commanded torque [Nm] |
| `wheel_torque` | Wheel torque demand [Nm] |
| `throttle` | Throttle position [0–1] |
| `Fx_eng`, `Fx_aero`, `Fx_roll` | Longitudinal forces [N] |
| `Fz_aero`, `Fz_mass`, `Fz_total` | Vertical forces [N] |

---

## Requirements

### MATLAB / Simulink Version
- Developed on **MATLAB R20XX** *(update with your version)*
- **Simulink** required

### Required Toolboxes
- Simulink
- Control System Toolbox (PI tuning)
- Signal Processing Toolbox (optional, for scope analysis)

---

## How to Run

1. Clone the repo:
   ```bash
   git clone https://github.com/youruser/FSAE-EV-Powertrain-FOC-Sim.git
   cd FSAE-EV-Powertrain-FOC-Sim
   ```

2. Open MATLAB and navigate to the project folder.

3. Run the parameter init script:
   ```matlab
   run('motor_params_init.m')
   ```

4. Load your LapSim data (OpenLAP `.mat` output) into the workspace.

5. Open the Simulink model:
   ```matlab
   open('NewTable_Copy.slx')
   ```

6. Hit **Run**. Stop time is set to match the LapSim lap duration (~101 s).

---

## Known Limitations / Notes

- `B_drag = 0.005 Nm·s/rad` is a numerical stabilization value only — replace with real vehicle drag from LapSim for accurate energy estimates.
- DRV faults (0x03) on the DTI HV-500 are latching — require LV power cycle to clear. Typically caused by incomplete precharge or low bus voltage.
- Field weakening is approximate; full MTPA optimization is a future improvement.
- `.slx` files are binary — GitHub cannot diff them. Commit often with descriptive messages.

---

## Team

| Name          | Role |
| Colin Janisse | HV Powertrain Co-Lead |


**Supervisor:** University of Windsor ECE Department
**Competition:** FSAE Michigan 2025–2026

---

## References

- EMRAX 228 Datasheet v1.6
- DTI HV-500 User Manual V2.6
- DTI CAN Manual V2.4
- RLS RM44 Encoder Datasheet
- OpenLAP Lap Simulation Documentation
