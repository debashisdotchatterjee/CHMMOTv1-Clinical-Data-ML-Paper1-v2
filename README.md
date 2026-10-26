# CHMMOTv1: Joint Monotone Ordinal Learning of Cardiac and Hepatic Iron Overload

**Reproducible code and computational outputs **

This repository contains the Google Colab implementation of a **joint, two-task cumulative-link ordinal model** for cardiac and hepatic iron-overload severity using the CHMMOTv1 clinical dataset. It preserves the manuscript's shared clinical representation, organ-specific monotone MRI effects, within-development-patient fused regularization, and conformal prediction framework. Independent ordinal XGBoost models are included as **comparators**, not substituted for the proposed joint model.

> **Research-use qualification:** This repository reproduces the analyses of source-supplied MRI-associated severity categories. It does not establish an independent clinical diagnosis, causal effect, external validity, or prospective prediction of future disease progression.

## Repository contents

| File | Purpose |
|---|---|
| [`CHMMOT_JOINT_COMPLETE_COLAB_Revised_code.ipynb`](CHMMOT_JOINT_COMPLETE_COLAB_Revised_code.ipynb) | Self-contained Colab notebook: installs dependencies, creates the analysis scripts, requests the workbook, fits the joint and comparator models, generates outputs, and downloads a results ZIP. |
| [`CHMMOT_Joint_Methodology_Revision_Results.zip`](CHMMOT_Joint_Methodology_Revision_Results.zip) | Results archive containing generated figures, tables, and available reproducibility metadata. |
| [`LICENSE`](LICENSE) | MIT license for the **author-generated code**; the original clinical dataset has its own third-party license. |

The source patient-level workbook is **not stored in this repository**. Download it separately from the data publisher (see below).

## Original public dataset and citation

**CHMMOTv1: Cardiac and Hepatic Multi-Echo (T2*) MRI Images and Clinical Dataset for Iron Overload on Thalassemia Patients**, deposited by Iraj Abedi, Maryam Zamanian, Hamidreza Bolhasani and Milad Jalilian in Mendeley Data:

- **Original public record, version 1 (2023):** https://data.mendeley.com/datasets/3d8bwn2fsh/1  
  DOI: https://doi.org/10.17632/3d8bwn2fsh.1
- **Version 4 (published 15 September 2026):** https://data.mendeley.com/datasets/3d8bwn2fsh/4  
  DOI: https://doi.org/10.17632/3d8bwn2fsh.4
- **Dataset descriptor article:** Abedi I, Zamanian M, Bolhasani H, Jalilian M. *CHMMOTv1—cardiac and hepatic multi-echo (T2*) MRI images and clinical dataset for iron overload on thalassemia patients.* **BMC Research Notes** 16, 339 (2023). https://doi.org/10.1186/s13104-023-06607-5

**Versioning caution:** The original descriptor refers to 124 patients, while the Excel workbook analyzed for this revision has 126 distinct `Patient Code` values and 219 records. This discrepancy has not been independently reconciled. Furthermore, Mendeley Data releases have different versions and may have different license terms. **Do not silently substitute a newer workbook for the analysis input.** Verify the exact downloaded file and its license. The input-file checksum for the reported revision run is given below; an exact match is the preferred way to confirm that you are reproducing those numerical results.

## Expected input file and structure

Download the clinical Excel data, save it as:

```text
CHMMOTv1_Clinical_Data.xlsx
```

The notebook expects these two worksheet names **exactly**:

```text
With Follow-Up
Without Follow-Up
```

Each worksheet must contain the following columns (spelling and punctuation matter):

```text
Patient Code
Age
Sex
T2*(Cardiac)
R2*(Cardiac)
MIC
Cardiac Overload
T2*(Liver)
R2*(Liver)
LIC
Liver Overload
Ferritin
AST
ALT
ALP
BILL D
BILL T
```

For the clinical workbook supplied to the analysis, the two worksheets contain 170 and 49 rows, respectively (219 records, 126 distinct patient identifiers). The source labels are five ordered categories: Normal, Mild, Moderate, Severe, and Very Severe.

**SHA-256 of the workbook used to generate the reported results:**

```text
e3a006deb18cb3d50259028bf3e78a7babb45532fe17310b5532c855c6ade9ba
```

To check a local copy:

```python
import hashlib
from pathlib import Path
path = Path("CHMMOTv1_Clinical_Data.xlsx")
print(hashlib.sha256(path.read_bytes()).hexdigest())
```

A different hash does not automatically mean an invalid workbook, but it means that exact numerical reproducibility has **not** been established. Check the dataset version and any data cleaning or spreadsheet changes before comparing results.

## Run in Google Colab (recommended)

1. Open the notebook directly in Colab:  
   **https://colab.research.google.com/github/debashisdotchatterjee/CHMMOTv1-Clinical-Data-ML-Paper1-v2/blob/main/CHMMOT_JOINT_COMPLETE_COLAB_Revised_code.ipynb**
2. Select **Runtime > Run all** (a standard Python runtime is sufficient; a GPU is not required).
3. The installation cell installs or updates the notebook's Python dependencies. PyTorch is imported from the Colab runtime.
4. When prompted, upload `CHMMOTv1_Clinical_Data.xlsx` from your computer.
5. The notebook recreates `reference_xgboost_pipeline.py` and `joint_multitask_revision.py` from its embedded source-code cells. **No second repository or code download is required.**
6. It fits the patient-disjoint reference models and the joint-model variants; it then generates figures, tables, diagnostic files and selected inline previews.
7. At completion, the notebook automatically downloads **`CHMMOT_Joint_Methodology_Revision_Results.zip`**. If the browser blocks the download, retrieve the ZIP from Colab's file sidebar.

**Run the whole notebook from the beginning after restarting the runtime.** Do not evaluate only the later cells in a fresh session; they depend on scripts created in earlier cells.

## Dependencies

The Colab notebook's installation cell requests the following minimum versions:

```bash
pip install "numpy>=1.26" "pandas>=2.1" "scipy>=1.11" \
            "matplotlib>=3.7" "scikit-learn>=1.3" \
            "xgboost>=2.0" "openpyxl>=3.1"
```

The joint model additionally requires **PyTorch (`torch`)**, which is ordinarily preinstalled in Google Colab. In a local environment where it is absent, install an appropriate PyTorch build using https://pytorch.org/get-started/locally/.

The reported full-run manifest recorded the following environment: Python 3.13.5; PyTorch 2.10.0+cpu; NumPy 2.3.5; pandas 2.2.3; scikit-learn 1.8.0; and XGBoost 3.1.3. The notebook uses minimum-version constraints rather than a fully locked environment, so exact floating-point agreement is not guaranteed across future software releases or hardware. Consult the results ZIP's `metadata/JOINT_RUN_MANIFEST.json` for the run configuration.

## Random seed, analysis partitions and full-run settings

The common **random seed is `20261026`**, as declared in the code. The full Colab execution explicitly uses:

```python
revision.main(
    input_file=workbooks[0],
    output="CHMMOT_Joint_Methodology_Results",
    fast=False,
    skip_reference=False,
    epochs=440,
    bootstrap=600,
)
```

The 126 patients are split **by patient**, with 68 for training, 20 for hyperparameter tuning, 19 for independent conformal calibration, and 19 for held-out testing. The primary test unit is the **first workbook-listed record per held-out patient**. Patient-weighted model fitting allows multiple development records without treating each record as an independent person. The calibration partition is separate from model selection.

The joint model's tested components include shared low-dimensional clinical representation, organ-specific ordered cumulative-logit thresholds, constrained own-organ MRI effect, fused regularization across *listed* development records, and patient-disjoint marginal and simultaneous cardiac–hepatic split-conformal prediction. The analyses also include joint-versus-independent, monotone-versus-unconstrained, fused-versus-unfused, and MRI-only ablations, together with XGBoost reference comparisons, measurement audits and uncertainty intervals.

**Exploratory speed settings are not the paper's full-run settings.** The notebook executes the full configuration; any `--fast`/reduced-epoch run should be labelled an exploratory smoke test.

## Outputs

The notebook creates a local `CHMMOT_Joint_Methodology_Results/` directory with the following important components:

```text
CHMMOT_Joint_Methodology_Results/
├── figures/         # Individual publication figures: 300-dpi PNG and vector PDF
├── tables/          # Machine-readable CSV and typeset LaTeX tables
├── metadata/        # Run manifest, preprocessing and model metadata
├── private_review/  # Local sensitive/row-level data-quality review (not zipped)
└── README_RESULTS.txt
```

The ZIP includes exportable results while intentionally **excluding `private_review/`**. Output labels beginning with `J` correspond to joint-model analyses; the remaining figures/tables include earlier independent-model and dataset-diagnostic comparators. Principal outputs include:

- `tables/J03_joint_and_ablation_test_performance.csv`
- `tables/J04_paired_joint_model_ablations.csv`
- `tables/J06_split_conformal_joint_and_marginal.csv`
- `tables/J07_monotonicity_analytic_and_numerical.csv`
- `tables/J14_patient_bootstrap_joint_metrics.csv`
- `tables/J15_primary_joint_vs_XGBoost_comparison.csv`
- `metadata/JOINT_RUN_MANIFEST.json`

Inspect the actual tables when reporting results: the joint model and its regularization components **must not be described as superior to simpler comparators unless supported by the measured comparisons**.

## Methodological and clinical limitations

- **Labels:** Severity classes are supplied MRI-associated labels. Predicting them from the same MRI variables can approximate the source grading criteria and does not establish independent clinical utility.
- **Dependence:** Cardiac/hepatic T2* and R2* encode related relaxometry information (`R2* = 1000/T2*` when T2* is expressed in milliseconds and R2* in s^-1). Derived LIC/MIC quantities are not independent measurements simply because they occur in separate columns. The primary model uses own-organ T2*; alternative representations are studied separately.
- **No time stamps:** The workbook does not contain usable visit dates. Fused penalties use **row/list order**, not verified elapsed time. Do not describe this as estimating time-specific progression rates.
- **New-patient prediction:** Visit offsets estimated for development patients are **not** transferred to unseen patients. At test, offsets are set to zero.
- **Calibration uncertainty:** Patient-level split conformal coverage depends on exchangeability and independent calibration; with 19 calibration and 19 test patients, empirical coverage is imprecise and is neither conditional nor guaranteed for an external population.
- **Data provenance:** The public 124-versus-analyzed-126 patient discrepancy requires source-version reconciliation, not a post hoc change to the results.
- **Generalization:** No independent external-center validation or independently adjudicated prospective clinical endpoint is supplied.

## Data governance and licensing

The MIT license in this repository covers the **author-generated code**, **not** the CHMMOTv1 data. Respect the terms stated on the **specific Mendeley Data version** used. Avoid committing unreviewed row-level patient records, private diagnostics or identifiers to this repository. The provided output workflow excludes the local private-review directory from the published ZIP.

## Citation

If you use the underlying dataset, cite its relevant Mendeley Data version and the dataset paper:

> Abedi I, Zamanian M, Bolhasani H, Jalilian M (2023). CHMMOTv1—cardiac and hepatic multi-echo (T2*) MRI images and clinical dataset for iron overload on thalassemia patients. *BMC Research Notes*, **16**, 339. https://doi.org/10.1186/s13104-023-06607-5

For the code, cite this repository URL and, when available, a future archived release DOI:

https://github.com/debashisdotchatterjee/CHMMOTv1-Clinical-Data-ML-Paper1-v2

**Manuscript context:** PLOS ONE submission PONE-D-25-66355; joint monotone ordinal learning of cardiac and hepatic iron-overload categories from MRI and clinical biomarkers.
