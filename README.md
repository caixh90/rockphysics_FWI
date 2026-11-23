# Code overview

The code has two parts that run in sequence: elastic FWI recovers Vp, Vs, and density from the VSP data, and rock-physics inversion converts them to porosity, mineral fractions, and CO₂ saturation. All input data are in `../data/`. Each subfolder has its own README with details.

## 1. `elastic_FWI/` — time-lapse elastic FWI (Python, PyTorch)

**Main script:** `Main_AC_only_bm.py`

```bash
cd elastic_FWI
python Main_AC_only_bm.py
```

- Stage 1 updates Vp and density. Stage 2 updates Vs and is commented out at the end of the script; to run it, comment out Stage 1, uncomment Stage 2, and run the script again.
- Results are written to `../../results/`.
- A CUDA GPU is strongly recommended. The full inversion is too slow on a CPU.

## 2. `rock_physics_inversion/` — rock-physics inversion (MATLAB)

Run the scripts in this order, with MATLAB's current folder set to each script's folder:

| Step | Script | Purpose |
|------|--------|---------|
| 1 | `02_well_logs/check_well_logs.m` | Smooth the well logs and check the rock-physics model |
| 2 | `01_FWI_results/check_fwi.m` | Smooth the baseline and monitor FWI models laterally |
| 3 | `03_Bayesian_inversion/stage1_inversion2D.m` | Bayesian inversion for porosity and mineral fractions |
| 4 | `04_NA/run2d_stage12.m` (**main script**) | Neighborhood-algorithm inversion: Stage 1 for rock properties, Stage 2 for CO₂ saturation |

In `run2d_stage12.m`, `weight` selects the elastic parameters used for saturation (for example, `[1 0 0]` uses Vp only and `[1 0 1]` uses Vp and density). `run1d_stage12.m` and `stage1_inversion1D.m` are single-trace versions for quick tests.
