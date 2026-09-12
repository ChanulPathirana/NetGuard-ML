# NetGuard ML — Notebook Quality / Reproducibility Audit

## Execution Result

`notebooks/EDA.ipynb` now executes successfully from a clean Python kernel from top to bottom. It contains 103 cells: 72 code cells and 31 markdown cells. Seventy code cells contain executable content; two are empty. The final run contains 121 output objects, 14 embedded PNG outputs, and no traceback/error outputs.

A pre-fix backup is preserved at `notebooks/EDA.before_repro_fix.ipynb`.

## Findings

| Issue | Severity | Evidence | Why it matters | Recommended fix |
|---|---|---|---|---|
| Short loop variables overwrote the canonical target | **Critical — fixed** | Feature and PCA Mann–Whitney loops assigned global `y`; original clean run later failed with a 119,341-versus-175,341 index mismatch in cell 87 | Hidden interactive state made Run All fail and caused later target diagnostics to inspect PCA scores | Renamed temporary values to `normal_values` and `attack_values`; target remains `train_df["label"]` |
| Extreme-observation cell used stale `y` | **Critical — fixed** | Original cell 87 used `np.asarray(y)[top_indices]` and raised `IndexError` | Labels could be wrong or execution could stop | Replaced with `train_df["label"].to_numpy()[top_indices]` |
| Global-variable debugging loop mutated its iterator | **High — fixed** | Original cell 93 iterated over `globals().items()` and raised `RuntimeError: dictionary changed size during iteration` | Prevented clean-kernel completion | Iterate over `list(globals().items())` |
| Source filenames and notebook roles are reversed | **High** | Cell 2 loads `UNSW_NB15_testing-set.csv` as `train_df` (175,341 rows) and `UNSW_NB15_training-set.csv` as `test_df` (82,332 rows) | Ambiguity can invalidate claims about held-out evaluation and data provenance | Explicitly document and confirm the intended split convention before modeling; rename variables or paths in a separate approved cleanup |
| Outlier cutoff is inconsistent by rounding | **High** | Cell 82 uses exact `chi2.ppf(0.99, 39)` and flags 17,337; cell 95 hard-codes `62.43` and flags 17,336 | Two notebook sections report different counts for the same conceptual rule | Store one computed cutoff and reuse it; do not hard-code the rounded display value |
| Duplicate-pattern count is described imprecisely | **High** | Cell 16 uses `duplicated(subset=feature_cols).sum()` = 74,301; markdown calls these all observations that “share” patterns | Pandas counts repeated occurrences after the first, not all rows belonging to duplicated groups | Report “74,301 repeats beyond first occurrence” or compute all group-member rows separately |
| Correlation method changes without clear labeling | **High** | Cell 64/65 uses Spearman; cell 66 overwrites `corr_matrix` using default Pearson for pair/group analysis | Readers may attribute Pearson pairs to the Spearman heatmap | Use distinct names such as `spearman_corr` and `pearson_corr`; state the method beside every result |
| `id` enters extreme-feature difference ranking | **High** | Cell 99 selects all numeric columns except `label`; output ranks `id` seventh | A row key can be mistaken for explanatory network behavior | Exclude `id` and `attack_cat` using the canonical candidate list in this comparison |
| “DROP CANDIDATE” heuristic is too assertive | **Medium** | Cell 67 combines Pearson correlation with univariate rank-biserial magnitude and prints keep/drop strings | Univariate effect size does not determine multivariate model utility | Rename to investigation hypotheses and validate every removal against held-out performance |
| Outlier threshold assumptions are weak | **Medium** | 99% chi-square cutoff flags 9.89%; variables are strongly skewed, zero-heavy, discrete, and heavy-tailed | Nominal chi-square probability is not calibrated under these distributions | Label flags heuristic; consider robust sensitivity checks later without deleting records automatically |
| Debug/recovery section is long and redundant | **Medium** | Cells 85–98 inspect variables, diagnose stale `y`, recreate target, and repeat outlier tables | Obscures the authoritative analysis and increases state-collision risk | Consolidate into assertions and one canonical outlier section in a future cleanup |
| Generic globals remain heavily reused | **Medium** | Names such as `normal`, `attack`, `result`, `groups`, `pairs`, `target`, and `X` change meanings across sections | Makes maintenance and isolated execution risky | Adopt section-specific names (`pc_normal`, `corr_pairs`, `X_numeric_eda`) |
| Full train/test schema check is incomplete | **Medium** | Both frames accept the 42 feature list, but no explicit dtype/category comparison is displayed | Encoding may fail or shift if dtypes/categories differ | Add a non-modeling schema comparison before baseline work |
| EDA PCA/scaling fits all development observations | **Low / contextual** | Cell 70 fits `StandardScaler` and PCA on all 175,341 `train_df` rows | Acceptable for descriptive EDA, but these fitted objects must not enter validation | Refit preprocessing only on training partitions inside the modeling pipeline |
| Multiple-testing adjustment is absent | **Low** | 39 Mann–Whitney tests use 0.05 individually | Family-wise false-positive control is not explicit | Add adjusted p-values if confirmatory inference is intended; retain effect-size emphasis |
| Plot coverage is incomplete | **Low** | No target, attack-category, or attack-rate plots; linear histograms compress tails | Some key results are table-only and distribution shapes are hard to read | Add only if a future EDA redesign is authorized; current report does not fabricate them |
| Minor presentation issues | **Low** | Typo “Multivariative”; cell-2 comment contains `a82,332`; two empty code cells | Reduces polish | Correct in a future documentation cleanup |

## Reproducibility Verification

- Clean-kernel Run All: **PASS**
- Traceback outputs: **0**
- Executable code cells completed: **70 of 70**
- `train_df.shape`: **(175341, 45)**
- PCA target alignment: **175,341 labels, 0 missing after alignment**
- Authoritative final target: **`train_df["label"]`**
- `X_model.shape`: **(175341, 42)**
- `y_model.shape`: **(175341,)**
- Numerical/categorical predictors: **39 / 3**
- `id`, `attack_cat`, `label` excluded from `X_model`: **PASS**
- Model training performed: **No**

## Audit Decision

The notebook is now mechanically reproducible and its main EDA outputs support transition to baseline development. Before final test evaluation, the project must resolve/document the train/test filename-role inversion and avoid reusing the EDA-fitted scaler/PCA. The outlier-count rounding and correlation-method ambiguity should be corrected in a future methodology/documentation cleanup, but they do not prevent establishing a baseline with the verified 42-feature set.
