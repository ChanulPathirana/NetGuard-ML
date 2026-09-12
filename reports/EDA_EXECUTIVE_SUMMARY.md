# NetGuard ML — EDA Executive Summary

The cleanly executed notebook analyzes 175,341 UNSW-NB15 flow records with 45 columns. It also loads an 82,332-row comparison table. The larger table is read from a file named `UNSW_NB15_testing-set.csv` and assigned to `train_df`; this deliberate or accidental naming inversion must be confirmed before final evaluation.

The binary target contains 119,341 Attack records (68.06%) and 56,000 Normal records (31.94%). This moderate 2.13:1 imbalance calls for stratification and class-specific evaluation, not observation removal. Attack categories are uneven: Generic, Exploits, and Fuzzers account for 76.74% of attacks, while Worms contributes only 130 records (0.11%).

Data completeness is strong: there are no missing values, exact full-row duplicates, negative numerical values, or constant columns. Repeated predictor patterns are common, however: Pandas identifies 74,301 repeated occurrences beyond the first. More importantly, 229 identical 42-feature patterns have conflicting labels, involving 940 records (0.54%). Those ambiguous patterns should be retained and investigated during model error analysis.

The numerical data are highly heterogeneous, zero-heavy, and heavy-tailed. For example, `sbytes` has median 430 but mean 8,844.8 and skewness 45.3; `dbytes` has median 164 but mean 14,928.9 and skewness 39.8. Sparse fields such as `is_ftp_login`, `ct_ftp_cmd`, and `is_sm_ips_ports` are over 98% zero, but zeros often encode valid absence. Several zero rates also differ sharply by class, making automatic sparse-feature deletion inappropriate.

Normal and Attack traffic show substantial numerical separation. The largest absolute rank-biserial effects are `sttl` (0.7629), `ct_state_ttl` (0.7412), `dload` (0.7306), `dmean` (0.6376), `dbytes` (0.5734), and `dpkts` (0.5658). All 39 numerical Mann–Whitney tests are significant, but p-values are not feature importance in a dataset this large.

Categorical behavior is also informative. TCP and UDP dominate protocols, with observed attack rates of 51.07% and 78.00%. Service rates range from SSH at 0.84% to POP3 at 99.64%; DNS is 84.16% and HTTP 71.44%. State INT is 93.05% Attack, FIN 52.23%, and CON 8.01%. Rare 100% categories require denominator-aware interpretation and are not automatically leakage.

Pearson correlation identifies eight strict (at least 0.90) redundancy groups, including `dbytes`/`dloss`/`dpkts`, `sbytes`/`sloss`/`spkts`, `tcprtt`/`synack`/`ackdat`, `dwin`/`swin`, contextual count groups, and the identical `is_ftp_login`/`ct_ftp_cmd` pair. These are hypotheses for controlled selection experiments, not final deletion decisions.

PCA uses all 39 standardized numerical features. PC1 explains 25.45% of variance, PC1–PC2 35.28%, the first 5 components 58.28%, the first 8 73.53%, the first 10 79.54%, the first 12 84.12%, and the first 15 89.75%. The two-dimensional projection shows central class overlap plus structured attack-heavy extremes. PC5 has the strongest class effect (0.5397), ahead of PC1 (0.3747), proving that explained variance and classification usefulness differ. PCA is valuable for EDA but is not mandated for modeling.

The final rounded multivariate-distance cutoff flags 17,336 records (9.89%). Normal has a 16.99% outlier rate versus Attack at 6.56%; flagged records are 54.87% Normal and 45.13% Attack. The 20 most extreme inspected rows are all attacks—12 Exploits, 7 DoS, and 1 Generic—but that tail does not represent all outliers. Heavy tails make the chi-square cutoff heuristic, and no flagged row should be deleted solely for unusualness.

Leakage decisions are clear. `id` is a unique row identifier and is excluded. `attack_cat` perfectly discloses binary class membership and is excluded from binary inputs but retained for post-hoc error slicing. `label` is the target. The verified initial modeling data contains 42 predictors—39 numerical plus `proto`, `service`, and `state`—with shapes `(175341, 42)` for `X_model` and `(175341,)` for `y_model`.

The EDA is sufficient to begin baseline development after documenting the source-file role convention. The next step is to define the split strategy, fit categorical encoding and any scaling only on training partitions, train a baseline on all 42 legitimate candidates, evaluate class-specific errors, and use controlled validation experiments for later feature engineering or selection. The final comparison table should remain untouched until development decisions are frozen.
