# Blood-count changes before myelodysplastic syndrome (MDS)

Analysis code for the study "Blood-count changes years before myelodysplastic syndrome is diagnosed: prodrome, personal setpoints and a silent-alert simulation in 102,901 hospital patients".

Author: Banan Jamil Awrahman, Department of Information Technology, Halabja Technical Institute, Sulaimani Polytechnic University, Iraq (banan.awrahman@spu.edu.iq).

Study protocols v1.0 and v2.0 with all amendments are on Zenodo: https://doi.org/10.5281/zenodo.24423167

This repository holds code only. It contains no patient data. MIMIC-IV v3.1, MIMIC-IV-Note v2.2 and EHRSHOT v3.2 must be obtained by each user through PhysioNet and Redivis under their own credentialed-access agreements.

## Layout

- `src/config.py`: paths, CBC item definitions and code lists. Data are expected in `data/raw/hosp` (MIMIC-IV hosp module), and outputs go to `data/processed` and `results`.
- `src/09`, `10`, `12`, `16`: extract all CBCs, admission and outpatient CBCs, and haematology work-up events from MIMIC-IV.
- `src/13_prediag_models.py`: single-CBC model (SNAP), nested cross-validation.
- `src/30`–`46`: MDS analyses (cohort, prodrome, setpoints, silent alert, post hoc analyses P1–P3 and R1–R25).
- `sql/`: BigQuery template for the MIMIC-IV-Note discharge summaries (R25).
- `external_validation/kit/`: the stand-alone validation kit (v1.0, 16 October 2026). It writes aggregate results only; `make_example_data.py` creates synthetic test data.
- `external_validation/ehrshot/`: builds the kit's input files from an EHRSHOT (OMOP) export (R26).
- `figures/`: scripts for Figures 1, 2 and 4. They read the result files in `results/mds/v2`; adjust the path at the top of each script to your folder layout.

## Run order

```
python src/09_extract_all_cbc.py
python src/10_build_admission_cbc.py
python src/12_build_outpatient_cbc.py
python src/16_extract_heme_workup.py
python src/30_mds_cohort.py
python src/30_mds_cohort.py --control-followup none --out data/processed/mds_cohort_ctrl_none.csv.gz --resultsdir results/mds/ctrl_none
python src/13_prediag_models.py --cohort data/processed/mds_cohort.csv.gz --outdir results/mds/models
python src/31_mds_secondary.py
python src/32_mds_inspire.py
python src/33_mds_history.py
python src/34_mds_prodrome.py
python src/35_mds_setpoint.py
python src/36_mds_silent.py
python src/37_mds_prodrome_posthoc.py
python src/37_mds_prodrome_posthoc.py --10y
python src/35_mds_setpoint.py --outpatient-baseline
python src/38_mds_p3_decomposition.py
python src/39_mds_covariates.py
python src/41_mds_negative_control.py
python src/40_mds_reviewer.py
python src/42_mds_drugs.py
python src/40_mds_reviewer.py --drugs
python src/43_mds_revision.py
python src/44_mds_other_outcomes.py
python src/45_mds_revision2.py
python src/46_mds_notes.py          # after the BigQuery note export (sql/)
python src/45_mds_revision2.py --only=r25_notes
```

External validation (EHRSHOT):

```
python external_validation/ehrshot/prep_ehrshot.py      # run in the folder holding the EHRSHOT exports
python external_validation/kit/run_validation.py --patients patients_kit.csv --cbc cbc_kit.csv --out results_kit
python external_validation/kit/run_validation.py --patients patients_strict.csv --cbc cbc_kit.csv --out results_strict
```

## Software

Python 3.10, with the package versions in `requirements.txt`.

## Licence

MIT (see `LICENSE`).
