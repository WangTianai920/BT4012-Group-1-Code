# BT4012 Group 1 · Phishing website detection

[BT4012_Group_1.ipynb](BT4012_Group_1.ipynb) reproduces Group 1’s phishing classification project from the raw dataset. The notebook contains the full workflow, from sampling and preprocessing through feature analysis, classifier training and test evaluation.

## 1. Clone the repository and install dependencies

Use Python 3.12. Run the following from a terminal with access to this repository.

```sh
git clone https://github.com/WangTianai920/BT4012-Group-1-Code.git
cd BT4012-Group-1-Code
python3.12 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
```

On Windows, create the environment with `py -3.12 -m venv .venv` and activate it in PowerShell with `.venv\Scripts\Activate.ps1`. On macOS, XGBoost and LightGBM need the OpenMP runtime. If it is missing, install it with `brew install libomp`. [LightGBM installation guide](https://lightgbm.readthedocs.io/en/v4.6.0/Installation-Guide.html#macos)

## 2. Run the notebook

```sh
python -m jupyterlab BT4012_Group_1.ipynb
```

In JupyterLab, use the Python 3 kernel from the new environment and select **Restart Kernel and Run All Cells**. You can also run the notebook from the terminal:

```sh
python -m nbconvert --execute --to notebook --inplace BT4012_Group_1.ipynb --ExecutePreprocessor.timeout=7200
```

The notebook downloads the pinned `dataset_full.csv` into `data/` on the first run and verifies its SHA-256 checksum. For offline use after dependencies are installed, save the [exact source CSV](https://raw.githubusercontent.com/GregaVrbancic/Phishing-Dataset/19674806813c1205d8c12bae99c15c5a735a4ac9/dataset_full.csv) as `data/dataset_full.csv` before running. The verified checksum is `87da36a20c8c1a9d13db5899514071907179c0fe790dc2ea4f0811957d68745d`.

### Replay feature ranking and hyperparameter search

The default run computes correlations and the 72-vs-77-feature comparison, fits all five models from scratch with the originally selected hyperparameters, and scores the weighted ensemble. To also regenerate both leave-one-feature-out rankings and rerun all four randomized searches, run:

```sh
BT4012_RUN_FEATURE_RANKING=1 BT4012_RUN_HYPERPARAMETER_SEARCH=1 \
  python -m nbconvert --execute --to notebook --inplace BT4012_Group_1.ipynb --ExecutePreprocessor.timeout=7200
```

You can enable either step independently. In JupyterLab, set the corresponding `RUN_FEATURE_RANKING` or `RUN_HYPERPARAMETER_SEARCH` flag in the first code cell to `True`. The full replay adds 745 feature-ranking fits and 300 cross-validation fits for tuning. Allow substantially more CPU time than for the default run.

## 3. Check the results

| Generated in `outputs/` | Contents |
| --- | --- |
| `{train,test}_processed.csv` | 75 processed predictors |
| `{train,test}_processed_72_features.csv` | 72 predictors after removing three duplicate columns |
| `{train,test}_engineered_77_features.csv` | 77 predictors after adding five engineered features |
| `correlation_pairs_*.csv`, `pearson_matrix_*.csv`, `spearman_matrix_*.csv` | Training-set correlation diagnostics |
| `feature_comparison_by_fold.csv` | Five-fold comparison of 72 and 77 predictors |
| `feature_ranking_*.csv`, `ranking_fold_results_*.csv`, `integrated_feature_ranking_72.csv` | Leave-one-feature-out results when feature ranking is enabled |
| `validation_metrics.csv`, `test_metrics.csv`, `test_precision_recall.png` | Individual and ensemble metrics and holdout precision–recall curves |
| `model_parameters.json`, `reproduction_receipt.json` | Model settings, run mode, dataset checksum, sizes, package versions and completion status |

The notebook should produce **47,820 training rows** and **11,956 test rows**, with **2,451** and **613** phishing observations respectively. Each final split should have **77 predictors** plus the `phishing` label. Check that `complete` is `true` in the receipt.

The protocol preserves the original full-sample constant and sparsity screening before the holdout split. Imputation, power transforms and scaling are fitted on the outer training split. Internal cross-validation and validation reuse that preprocessing, so those scores are exploratory. The random holdout has an imposed phishing prevalence of 5.13%. Its metrics describe that evaluation setting and may vary slightly across platforms and library builds.

## Data source

Grega Vrbančič (2020), *Phishing Websites Dataset*, Version 1, Mendeley Data, CC BY 4.0. The notebook uses the full variant with 88,647 observations and 111 input features. [Dataset record](https://doi.org/10.17632/72ptz43s9v.1)
