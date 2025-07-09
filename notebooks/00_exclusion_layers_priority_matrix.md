# Exclusion Layers and Constraint Prioritization – General Framework

This document defines the general framework used to compute exclusion scores across the Area of Interest (AOI) in the submarine cable routing project. The model distinguishes between **hard exclusion zones** (non-negotiable constraints) and **soft constraints** (accumulative risk factors). Each spatial constraint is associated with a penalty weight `wᵢ`, and the final exclusion score is computed per grid cell on a 0–100 scale.

---

## Hard Exclusion Rule (Veto Constraint)

The **2× water depth buffer** around existing submarine cables is treated as a **non-negotiable exclusion zone** following the guidelines established by the **International Cable Protection Committee (ICPC)**. Any overlap with this zone makes a location technically invalid for route planning, regardless of other conditions.

**Rule applied:**  
If a grid cell intersects this buffer:  
- `flag_exclusion = True`  
- `final_score = 100`  

This constraint overrides all others and establishes a hard veto.

---

## Soft Constraints and Risk Accumulation

All other spatial constraints are treated as **soft penalties** and are accumulated into a `raw_risk` value only for cells **not flagged for hard exclusion**. Each constraint contributes a fixed penalty weight `wᵢ`. The accumulated risk is then normalized to a scale of **0–90**, ensuring the total risk never surpasses the hard exclusion threshold.

**Final Score Logic:**

- If `flag_exclusion = True` → `final_score = 100`  
- Else → `final_score = (raw_risk / W) × 90`, where `W = ∑ wᵢ` over all soft constraints  

---

## Constraint Layers and Assigned Weights (`wᵢ`)

| Constraint Layer                         | Description                                 | Weight (`wᵢ`) |
|------------------------------------------|---------------------------------------------|---------------|
| `telecom_cables_exclusion_zone_2WD`      | Hard exclusion: 2× water depth buffer       | — *(veto)*    |
| `telecom_cables_exclusion_zone_3WD`      | Caution zone: 3× water depth buffer         | 95            |
| `rocky_outcrops_clipped_depth` (shallow) | Rocky seabed at depth < 1000 m              | 90            |
| `rocky_outcrops_clipped_depth` (deep)    | Rocky seabed at depth ≥ 1000 m              | 60            |
| `fishing_areas_clipped`                  | Trawling and protected fisheries            | 80            |
| `military_areas_clipped`                 | Naval or military use zones                 | 70            |
| `natura2000_clipped`                     | Natura 2000 environmental protection areas  | 80            |
| `wrecks_clipped`                         | Submerged shipwrecks or obstacles           | 90            |
| `coralligenous_outcrops_clipped`         | Sensitive coral or biogenic reef areas      | 70            |
| `fluid_emissions_clipped`                | Gas seeps, fluid discharge zones            | 40            |
| `pockmarks_clipped`                      | Depressions from past fluid activity        | 30            |
| `mud_mounts_clipped`                     | Mud volcanoes, unstable sediment mounds     | 30            |

---

## Scoring Interpretation

| Final Score        | Interpretation                          |
|--------------------|------------------------------------------|
| 100                | Hard exclusion (veto: 2×-WD buffer)       |
| 70–90              | High risk (multiple severe soft constraints) |
| 40–70              | Medium risk                              |
| 5–40               | Low to moderate concern                  |
| 0                  | No spatial risk identified               |

---

## Summary of Benefits

- **Guarantees compliance with ICPC guidelines** through a non-negotiable veto rule.
- **Ensures proportional scaling** of soft constraints while preventing overaccumulation.
- **Facilitates interpretation** by stakeholders with intuitive 0–100 scores.
- **Supports route screening** with both binary (excluded/allowed) and gradient (risk level) logic.

---

> **Disclaimer**:  
> The exclusion weights (`wᵢ`) or scorign interpretation presented in this file are not meant to be normative or prescriptive.  
> They have been defined specifically for this project as an illustrative framework, based on general routing considerations and example constraints.  
>  
> In real-world cable routing projects, these weights should be carefully reviewed and adjusted to reflect:
> - Client-specific priorities
> - Local environmental and operational conditions
> - Technical feasibility and regulatory constraints  
>  
> Always align the weighting criteria with the project context and stakeholder requirements.