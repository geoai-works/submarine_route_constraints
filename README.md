
# Submarine Route Constraints – Exclusion Scoring Framework

This repository contains a geospatial analysis framework to identify spatial constraints for submarine cable route planning. The project defines and computes **exclusion scores** across a 100 m resolution grid, based on thematic constraint layers such as wrecks, rocky outcrops, fisheries, protected areas, and proximity to existing cables.

The scoring model implements a two-tier system:

- A **hard exclusion rule** for critical constraints that prohibit cable routing (based on ICPC guidelines)
- A **soft constraint accumulation** for other spatial risks, producing a proportional risk score

---

## Motivation

When designing submarine cable routes, avoiding spatial conflicts is essential for technical viability, permitting, and cost optimization. This framework provides a reproducible and explainable method for:

- Enforcing **non-negotiable exclusion rules** (e.g., minimum cable separation)  
- Quantifying **relative risk** in areas where routing is physically possible but environmentally or operationally sensitive  
- Supporting engineers and planners with **transparent spatial scores** on a 0–100 scale  

---

## Structure

### Notebooks

| Notebook                                     | Title                                      | Description                                                                                 |
|----------------------------------------------|--------------------------------------------|---------------------------------------------------------------------------------------------|
| `01_generate_grid.ipynb`                     | Generate Base Grid                         | Creates the 100 m resolution grid over the AOI                                              |
| `02_prepare_layers.ipynb`                    | Prepare and Clip Constraint Layers         | Loads and clips all thematic constraint datasets                                            |
| `03_assign_depth_to_rocky_outcrops.ipynb`    | Classify Rocky Outcrops by Depth           | Categorizes rocky seabed into shallow/deep using a 1000 m threshold                         |
| `04_define_cable_exclusion_zones.ipynb`      | Define Cable Buffers Based on Depth        | Uses raster-based depth to create 2× and 3× water depth buffer zones around existing cables |
| `05_generate_point_constraint_buffers.ipynb` | Create 500 m Buffers for Point Constraints | Converts point-based hazards (wrecks, pockmarks, mud mounts) into polygon buffers           |
| `06_assign_constraint_scores.ipynb`          | Assign Exclusion Scores to Grid Cells      | Computes a hard/soft exclusion score per cell                                               |

---

## Scoring Logic

### 1. Hard Exclusion – 2× Water Depth (ICPC)
- Cells intersecting the 2× water depth buffer around existing cables are marked with `flag_exclusion = True`  
- These receive a `final_score = 100` and are considered non-viable for routing

### 2. Soft Constraints Accumulation
- Other layers (e.g. 3×-WD buffer, rocky seabed, fishing zones, military areas) contribute weighted penalties (`wᵢ`)  
- Accumulated in `raw_risk` only for non-excluded cells  
- Normalized to `[0–90]` range to ensure no soft constraint combination outweighs the veto rule

### 3. Final Score per Cell
- `final_score = 100` if excluded  
- Else `final_score = (raw_risk / W) × 90`, where `W = sum of wᵢ for all soft constraints`

> See [`00_exclusion_layers_priority_matrix.md`](notebooks/00_exclusion_layers_priority_matrix.md) for a full justification of constraint weights and exclusion logic.

---

## Outputs

- `06_grid_constraint_scores.gpkg`: Main output with per-cell scores and flags  
- Intermediate layers in `/processed_data/constraints/` and `/processed_data/cable_buffers/`  
- Summary plots for inspection (optional)

Each cell includes:
- `flag_exclusion`: Boolean hard veto  
- `raw_risk`: Sum of applicable weights  
- `final_score`: Normalized exclusion score [0–100]  
- One column per individual constraint (`*_score`)

> Output files generated by the notebooks — including `06_grid_constraint_scores.gpkg` — are **not included** in the repository due to GitHub file size limitations (some exceed 100 MB).  
> However, **all outputs are fully reproducible** by running the notebooks in sequence, using the provided inputs and processing steps.

---

## Reproduce the Full Pipeline

All outputs used in this project are fully reproducible using the provided notebooks and input files. Follow the steps below to execute the entire pipeline from raw spatial data to exclusion scoring.

### 1. Clone the repository

```bash
git clone https://github.com/geoai-works/submarine_route_constraints.git
cd submarine_route_constraints
```

### 2. Set up the environment

We recommend using `conda` or `pip`.  
Example with `conda`:

```bash
conda create -n geo_env python=3.10
conda activate geo_env
pip install -r requirements.txt
```

### 3. Prepare input data

Some layers are provided as examples in `/processed_data/constraints/`.  
Others (e.g., military zones, geological layers) must be replaced by project-specific sources or public equivalents.  
See [`data_sources.md`](./data_sources.md) for a full reference.

### 4. Run the notebooks in order

Execute the notebooks sequentially to reproduce all outputs:

- `01_generate_grid.ipynb`  
- `02_prepare_layers.ipynb`  
- `03_assign_depth_to_rocky_outcrops.ipynb`  
- `04_define_cable_exclusion_zones.ipynb`  
- `05_generate_point_constraint_buffers.ipynb`
- `06_assign_constraint_scores.ipynb`

### 5. Inspect the results

The final scored grid will be saved to:

```
processed_data/05_grid_with_exclusion_scores.gpkg
```

Intermediate layers will be created in `/processed_data/constraints/` and `/processed_data/cable_buffers/`.

> Large output files are **not included** in the repository due to GitHub file size limitations.  
> However, **all outputs are fully reproducible** by running the notebooks as described above.

---

## Documentation

- `00_exclusion_layers_priority_matrix.md`: Conceptual framework and rationale for spatial constraint weighting (`wᵢ`)
- Each notebook includes a detailed header and is fully modular

---

## Collaboration and reuse

This repository is part of an open portfolio of spatial analytics for submarine cable planning.  
All notebooks and methods are designed to be reused and adapted.

If you're interested in using this scoring framework for a different region,  
**we can help you adapt it to your own data, seabed conditions, and regulatory context.**

> Contact: [Alejandra L. Cameselle](https://www.linkedin.com/in/alejandralcameselle/)

---

## License

This work is licensed under a [Creative Commons Attribution-NonCommercial 4.0 International License](https://creativecommons.org/licenses/by-nc/4.0/).  
You are free to reuse, adapt, and build upon this work **for non-commercial purposes**, as long as you provide attribution.

For commercial use, please contact the author directly.

---

## Developed by

This repository was developed by **Alejandra L. Cameselle** under the personal brand  
**[GeoAI Works](https://github.com/geoai-works)** — a portfolio of geospatial analytics for marine infrastructure, data science, and spatial risk assessment.