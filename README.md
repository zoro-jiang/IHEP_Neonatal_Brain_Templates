# IHEP Neonatal Brain Templates

**Gestational age-specific DTI templates of the neonatal brain: Application in preterm developmental study**


## 1. Introduction
This repository contains a set of **Gestational Age (GA)-specific DTI templates** constructed from 161 neonates. Unlike single full-term templates, these resources are stratified into four WHO-defined subgroups to minimize registration errors caused by rapid developmental changes:

* **EPT:** Extremely Preterm (< 28 weeks)
* **VPT:** Very Preterm (28 - < 32 weeks)
* **MLPT:** Moderate-to-Late Preterm (32 - < 37 weeks)
* **FT:** Full-Term (> 37 weeks)

Each group contains:
* **DTI Templates:** Fractional Anisotropy (FA), Mean Diffusivity (MD), and b0 images.
* **Parcellation Atlas:** 130 sub-regions (based on JHU-neonate-ss atlas), manually corrected for each GA group.

---

## 2. Recommended Use Cases & Workflows

To facilitate the application of these templates, we provide recommended workflows for three common analysis scenarios: **Spatial Normalization**, **Voxel-wise Analysis**, and **ROI-based Analysis**.

### Scenario A: Registration Targets (Spatial Normalization)
**Goal:** Align individual preterm data to a standard space with minimal deformation error.

**Recommendation:**
1.  **Select the Matched Template:** Choose the template group (EPT, VPT, MLPT, or FT) that best matches your subject's corrected gestational age.
2.  **Driver Image:** We strongly recommend using the **b0 image** (individual-to-template) to drive the registration. As detailed in our manuscript, neonatal b0 images provide robust T2-like contrast, particularly for CSF–tissue differentiation, and are often more reliable than FA maps images for driving non-linear registration in the neonatal brain.


**Example Workflow (ANTs):**
```bash
#!/bin/bash
# Example: Registering an individual b0 image to the MLPT template

# --- 1. Define Inputs & Outputs ---
fixed_image="Templates/MLPT/IHEP_MLPT_b0_template.nii.gz"  # The Template
moving_image="sub-01_b0.nii.gz"                            # Individual Subject
output_prefix="output/sub-01_to_MLPT_"                     # Output naming

# --- 2. Run Registration (Rigid + Affine + SyN) ---
# We use antsRegistrationSyN.sh for a standard robust pipeline
antsRegistrationSyN.sh \
  -d 3 \
  -f $fixed_image \
  -m $moving_image \
  -o $output_prefix \
  -n 4 # Number of threads
  
# Output will be:
# ${output_prefix}Warp.nii.gz      (Non-linear warp)
# ${output_prefix}0GenericAffine.mat (Linear transform)
```
### Scenario B: Voxel-wise Analysis
**Goal:** Perform whole-brain voxel-wise statistical analysis of white matter integrity.

**Recommendation:**
Standard adult templates (e.g., FMRIB58_FA) are unsuitable for neonatal data. We strongly recommend using our GA-specific FA templates as the registration target to ensure anatomical accuracy.

Note: While TBSS was originally developed for adult datasets, it has been widely adapted for neonatal studies when used with age-appropriate templates and adjusted FA thresholds.

**Example Workflow:**
```bash
#!/bin/bash
# Example: Running voxel-wise analysis using the VPT FA template as the target

# --- 1. Preparation ---
# Move all your subject FA images into a folder (e.g., ./my_analysis)
cd my_analysis

# --- 2. Preprocessing ---
# Resizes slices and erodes slightly to remove edge artifacts
# (This step utilizes FSL's standard preprocessing tools)
tbss_1_preproc *.nii.gz

# --- 3. Registration to IHEP Template ---
# IMPORTANT: Use the '-t' flag to specify our custom template!
# Do NOT use the default adult template.
target_template="../Templates/VPT/IHEP_VPT_FA_template.nii.gz"

tbss_2_reg -t $target_template

# --- 4. Post-registration & Skeletonization ---
# Projects data onto the mean skeleton
tbss_3_postreg -S
tbss_4_prestats 0.2  # Threshold of 0.2 is commonly used for neonates

# Result: stats/all_FA_skeletonised.nii.gz is ready for statistical inference (e.g., randomise)
```
### Scenario C: ROI-based Analysis
**Goal:** Extract mean DTI metrics (FA/MD) from specific white matter tracts (e.g., Corpus Callosum, Internal Capsule).

**Recommendation:**
Each template comes with a corresponding label map (Atlas) containing 130 regions. You can warp the individual to the template space (using the transform from Scenario A) and utilize the provided Atlas.

**Example Workflow (Individual -> Template):**
```bash
#!/bin/bash
# Example: Extracting mean FA from the Genu of Corpus Callosum (Label 115)

# --- 1. Define Inputs ---
atlas="Atlases/IHEP_MLPT_Atlas_130.nii.gz"
# This is the subject's FA map, already warped to template space (from Scenario A)
# Ensure this matches the output filename from the previous step
warped_subject_FA="output/sub-01_to_MLPT_Warped_FA.nii.gz" 

# --- 2. Option 1: Extract Single ROI (e.g., Label 115) ---
# 115 = Genu of corpus callosum (See IHEP_Neonatal_Atlas_Lookup_Table.csv)
roi_label=115
mean_fa=$(fslstats $warped_subject_FA -k $atlas -l 114.5 -u 115.5 -M)

echo "Subject 01 - ROI ${roi_label} Mean FA: $mean_fa"

# --- 3. Option 2: Extract ALL ROIs to a text file ---
# This generates a text file with 130 rows (one mean value per region)
fslmeants -i $warped_subject_FA --label=$atlas -o sub-01_all_ROI_stats.txt

echo "Extraction complete. See sub-01_all_ROI_stats.txt"
```
## 3. Repository Structure (Preview)
Upon release, the file structure will be organized as follows:

```text
.
├── Templates
│   ├── EPT (Extremely Preterm)
│   │   ├── IHEP_EPT_b0_template.nii.gz
│   │   ├── IHEP_EPT_FA_template.nii.gz
│   │   └── IHEP_EPT_MD_template.nii.gz
│   ├── VPT (Very Preterm)
│   ├── MLPT (Moderate-to-Late Preterm)
│   └── FT (Full-Term)
├── Atlases
│   ├── IHEP_EPT_Atlas_130.nii.gz
│   ├── IHEP_VPT_Atlas_130.nii.gz
│   ├── ...
│   └── IHEP_Neonatal_Atlas_Lookup_Table.csv  <-- Region ID to Name mapping
└── README.md
```
## 4. Citation
If you use these templates or atlases in your research, please cite our paper:

> **Gestational age-specific DTI templates of the neonatal brain: Application in preterm developmental study**
> *Authors:* Xiaochen Jiang, Mengyi Wang, et al.
> *Journal:* NeuroImage
> *Year:* 2026

## 5. Contact
For questions regarding the usage of these templates, please open an Issue in this repository or contact the corresponding authors.

* **Xiaochen Jiang**: jiangxc@ihep.ac.cn
* **Binbin Nie**: niebb@ihep.ac.cn

---
*Created by the Xiaochen jiang.*
