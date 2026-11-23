# Rock physics inversion (FWI elastic properties → petrophysical properties)

## Repository layout

| Folder | Role |
|--------|------|
| `01_FWI_results/` | Load FWI volumes, apply lateral smoothing, plot, and save elastic fields for inversion |
| `02_well_logs/` | Read well Excel, smooth logs, validate `model_sand_CMC`, save training/reference curves |
| `03_Bayesian_inversion/` | **Gaussian rock-physics Bayesian inversion** (`RockPhysicsGaussInversion.m`) plus 1D/2D example drivers |
| `04_NA/` | **Two-stage NA inversion**: Stage 1 for \(\phi\), quartz fraction, clay fraction; Stage 2 for **CO\(_2\) saturation `sc`** |
| `04_NA/funcs/` | NA objectives, forward-model wrappers, plotting |
| `04_NA/na_functions/` | Neighbourhood Algorithm core |

## Suggested processing order

1. **`02_well_logs/check_well_logs.m`**  
   - Input: `../../../data/Logs_qi.xlsx` (injection-well logs).  
   - Output: `Logs_qi_0_350_smooth.mat` (smoothed logs over 0–350 m: `depth`, `vpt`, `vst`, `rhot`, `port`, `qut`, `vclt`, `cot`, `Pefft`, etc.).  
   - Purpose: check forward-model consistency with logs and supply well curves for Bayesian/NA scripts.

2. **`01_FWI_results/check_fwi.m`**  
   - Input: `../../../data/snowflake_2026.mat` (baseline and monitor EFWI models `Basevp`, `Basevs`, `Baserho`, `Monvp`, `Monvs`, `Monrho`, 71 × 201).  
   - Processing: horizontal cropping and **2D lateral smoothing** (`func_smooth2D.m`).  
   - Output: `vpvsrho_base_moni.mat` (`vp_b`, `vs_b`, `rho_b`, `vp_m`, `vs_m`, `rho_m`, `xx`, `zz`).  
   - Purpose: provide **baseline / monitor** elastic sections for inversion.

3. **`03_Bayesian_inversion/stage1_inversion1D.m`** (or `stage1_inversion2D.m`)  
   - Inputs: `Logs_qi_0_350_smooth.mat`, `vpvsrho_base_moni.mat`.  
   - Method: estimate a joint Gaussian from well \((\phi, f_\mathrm{qz}, f_\mathrm{cl})\) and \((V_\mathrm{P},V_\mathrm{S},\rho)\), then apply the **analytical Gaussian posterior** (`RockPhysicsGaussInversion.m`) to FWI-derived curves.

4. **`04_NA/run1d_stage12.m`** or **`04_NA/run2d_stage12.m`**  
   
   - **Stage 1**: fit baseline \((V_\mathrm{P},V_\mathrm{S},\rho)\) with `sc = 0`; invert **`por`, `qu`, `vcl`** (search bounds in `lb1` / `ub1`).  
   - **Stage 2**: fix Stage-1 lithology and `Peff`, fit monitor data, invert **`sc`** (CO\(_2\) saturation).  
   - Data weights: `weight = [1 0 1]` sets **zero weight on \(V_\mathrm{S}\)** in the current implementation (change as needed).  
   - `run1d_stage12.m`: single trace (e.g. `trace = 61`) vs depth; `run2d_stage12.m`: trace-wise 2D grid with loops / `parfor`.

## NA and forward modelling

- Forward model: `04_NA/funcs/model_sand_CMC_vector.m` packs the parameter vector for `model_sand_CMC` (same idea as `02_well_logs/model_sand_CMC.m`, packaged for NA).  
- Objectives: `obj_stage1.m`, `obj_stage2.m` — weighted squared error.  
- Optimiser: `na_functions/NA_point.m` → `NeighbourhoodAlgorithm.m`, etc.

## How to run

- Set MATLAB’s **current folder** to the directory that contains each script (`load` / `readmatrix` / `save` use relative paths).  
- Run steps 1 and 2 first: they save `Logs_qi_0_350_smooth.mat` and `vpvsrho_base_moni.mat` next to their scripts. Steps 3 and 4 find these files and the functions through `addpath(genpath('..'))`, which adds the whole `code/rock_physics_inversion` folder.

## Data

The input files are in `code_and_data/data/`: `Logs_qi.xlsx` (injection-well logs) and `snowflake_2026.mat` (EFWI baseline and monitor models, produced by `code/elastic_FWI`).
