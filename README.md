# City-scale construction and demolition waste: data and code

This repository contains the compiled data, extraction evidence and analysis code supporting *From fragmented public disclosures to auditable panel data: a city-scale construction and demolition waste forecasting workflow*.

The study covers 21 prefecture-level cities in Guangdong, China. The compiled outcome dataset contains 139 city-year observations from 2010–2024. The forecasting analysis uses 2016–2024 records, including 103 training observations and nine 2024 Greater Bay Area targets with complete inputs. The data files preserve the values and row order used in the study.

**Release:** 1.0.0 · **Prepared:** 2026-11-04 · **Project identifier:** `cdw_city_panel`

## Download

The complete versioned distribution is [cdw_city_panel_v1.0.0.zip](cdw_city_panel_v1.0.0.zip). Download and extract it, then run the commands below from the extracted `cdw_city_panel` directory. The paths in this README refer to files inside that archive. The archive includes the data, code, evidence records, Supplementary Information S1–S8 and file-level checksums.

Verify the download against [the archive checksum](cdw_city_panel_v1.0.0.sha256):

```bash
shasum -a 256 -c cdw_city_panel_v1.0.0.sha256
unzip cdw_city_panel_v1.0.0.zip
cd cdw_city_panel
```

## Package contents

| Directory or file | Contents |
|---|---|
| `data/compiled/` | Outcome and covariate inputs used in the study |
| `data/processed/` | Forecast-ready and comparison panels |
| `data/screening/` | Predictor-screening panels |
| `data/lagged_features/` | Contemporaneous and lagged covariate exports |
| `evidence/extraction/` | Extraction prompt, JSON schema and configuration |
| `evidence/records/` | Source registry, outcome ledger, candidate queue, extraction attempts, validation and audit records |
| `evidence/screening/` | Archived candidate-screening and deterministic-validation code |
| `scripts/` | Main reproduction and validation entry points |
| `analyses/` | Self-contained analyses for extraction evaluation, predictor comparisons and sensitivity checks |
| `results/` | Saved forecasting, screening and diagnostic results |
| `docs/` | Data dictionary, Supplementary Information, analysis notes and attribution |
| `metadata/` | Dataset description, source lineage, file manifest and naming map |
| `verification/` | Saved numerical and release-validation records |
| `requirements.txt` | Pinned Python dependencies |
| `SHA256SUMS.txt` | Checksums for the distributed files |

## Data records

| File | Records | Description |
|---|---:|---|
| `data/compiled/cdw_generation.csv` | 139 | Standardised annual C&DW generation, in cubic metres |
| `data/compiled/city_covariates.csv` | 252 | Covariates for 21 cities, 2013–2024 |
| `data/processed/forecast_panel.csv` | 131 | Outcome panel with the five forecasting features, 2016–2024 |
| `evidence/records/outcome_source_ledger.csv` | 139 | Original units, evidence locations, reconciliation decisions and unresolved provenance fields |
| `evidence/records/extraction_audit_labels.csv` | 120 | Stratified deployment-audit decisions; not a document-disjoint benchmark |

The observation key is `district_name` and integer `year`. Blank numeric CSV fields denote missing values, not observed zeros. Natural logarithms are used for the outcome and the indicated covariates; urbanisation is retained in levels. Definitions, transformations and units are given in `docs/data_dictionary.md` inside the archive.

These are the actual compiled experimental inputs. Original workbook hashes are retained in the lineage metadata; the corresponding English CSV exports are distributed and used by the reproduction code. Full government documents and statistical-yearbook volumes are not redistributed. Chinese evidence excerpts and prompt text are retained verbatim for grounding. The modelling panel and extraction-evidence archives have distinct provenance; the later extraction batch is not asserted to have regenerated every earlier modelling observation.

## System requirements

The recorded environment uses Python 3.13.5, NumPy 2.1.3, pandas 2.2.3, SciPy 1.15.3 and scikit-learn 1.6.1. Joblib and threadpoolctl versions are also pinned. A Python virtual environment is recommended:

```bash
python -m venv .venv
# Activate the environment using the command appropriate for your operating system.
python -m pip install -r requirements.txt
```

The main reproduction and arithmetic checks run offline. They require no API key, model download or government-site access. Re-running Stata diagnostics requires Stata and `xtabond2`. The archived BERT interfaces require their external NLP dependencies and checkpoint; they are not invoked by the commands below.

## Reproduce the main results

Run commands from the repository root, using a separate Python process for each analysis:

```bash
python scripts/reproduce.py
python scripts/validate_analysis_outputs.py
python scripts/validate_model_comparisons.py
python analyses/density_sensitivity/main.py
```

The first command reconstructs features, verifies the original candidate ranking and audit counts, and refits the selected Ridge validation folds and final model. It also checks the recorded unweighted Ridge comparison. It does not rerun the full search. The remaining commands check saved outputs and fixed-prediction arithmetic. Generated files are kept separate from the distributed records; the main scripts write to `outputs/`.

Expected selected-model results on the nine 2024 targets are log-scale R² = **0.923785**, RMSE = **0.290351** and MAE = **0.243986**. Original-scale MAPE is **28.2483%**. Small differences at floating-point precision can depend on numerical-library versions.

The main reproduction command also verifies the distribution manifest. File-level checksums and source-to-distribution mappings are documented in `metadata/`.

## Additional analyses

| Analysis | Directory | Entry point |
|---|---|---|
| Rules-only comparison | `analyses/rule_comparison/` | `main.py` |
| Initial rule baseline | `analyses/rule_baseline/` | `main.py` |
| Excluded-passage audit | `analyses/excluded_passage_audit/` | Confirmed decisions and stratum summaries |
| Training-fold checks of fixed predictors | `analyses/fixed_predictors/` | `main.py --mode ridge` |
| Full-pool foldwise screening | `analyses/foldwise_screening/` | `main.py` |
| Matched Ridge comparison | `analyses/ridge_ablation/` | `main.py` |
| Original-grid model comparison | `analyses/model_comparison/` | `main.py` |
| Density sensitivity | `analyses/density_sensitivity/` | `main.py` |
| Historical screening-sample definitions | `analyses/screening_sample_audit/` | Sample keys and diagnostic definitions |
| Original-scale forecast errors | `analyses/forecast_errors/` | Saved city-level error tables |

Complete model searches are optional and can take substantially longer than the main reproduction:

```bash
python scripts/reproduce.py --full-search
python analyses/model_comparison/main.py --jobs 6
```

The latter searches 685 structural-only and 2,969 dynamic-only configurations and reuses the authenticated original combined-model search. It writes a new timestamped run rather than overwriting the archive. Instructions and saved-result locations are in each analysis directory.

The final manuscript comparison is supplied as `analyses/model_comparison/results/main_run/manuscript_table4_values.csv`. It contains two fixed references, three matched-Ridge rows and three rows selected under the original search rule.

## Interpretation and validation scope

The workflow's evidence is divided between the 120-candidate extraction audit, a separate 60-passage excluded-screening check and the forecasting evaluation. Their denominators are not pooled. Refined rules were developed using the audited corpus; their comparison with archived LLM outputs is retrospective.

The original four-predictor specification and 2024 outcomes were historically known. Training-only checks do not remove that design influence or create an untouched external test. The model-comparison results therefore describe this retrospective target set, not performance in cities without outcome histories.

The 112-row modelling cohort contains 104 confirmed native-volume records, seven confirmed mass-origin records and one undocumented unit origin. All seven mass-origin records are 2024 targets. Their complete fixed-prediction density sensitivity and a separately labelled unknown-origin scenario are supplied; neither changes the primary panel or selected predictions.

Methods, sample definitions, uncertainty and limitations are described in `docs/supplementary_information.md` (S1–S8) inside the archive. Saved validation results are under `verification/`.

## File naming and provenance

Primary files use descriptive English names and underscores. Calendar years in names refer to observations or evaluation periods. S, D and DS in analysis records identify structural-only, dynamic-only and combined inputs. Analysis-specific identifiers and frozen records remain stable for traceability.

`metadata/distribution_lineage.csv` maps each imported file to its previous location and checksum. Documentation and executable path references were updated for this distribution. Scientific CSV and JSONL content is preserved; naming changes do not alter observations, labels, candidate identifiers or predictions. Frozen protocol hashes are retained and checked separately from the current distribution manifest.

## Access, citation and reuse

This public repository hosts version 1.0.0 of the data and code distribution. The archived package retains the review-stage metadata used when the distribution was prepared; the hosting account is publicly visible. This repository URL is therefore not an anonymous reviewer access route. No DOI has been assigned.

See `docs/citation.md` and `docs/reuse_and_attribution.md` inside the archive. No new data or software licence has been assigned by packaging. Source-provider rights and quoted evidence attribution are retained.
