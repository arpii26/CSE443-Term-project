# Domain-Aware ESM-2 Based Missense Mutation Pathogenicity Prediction

Predicting whether a missense mutation is disease-causing remains one of the most important challenges in computational genomics. This project presents a lightweight and reproducible workflow that combines protein language model representations from ESM-2 with biologically interpretable features from ClinVar and UniProt to classify missense variants as benign or pathogenic.

Instead of fine-tuning a large protein language model, ESM-2 is used as a frozen feature extractor. These learned representations are then combined with domain-aware biological features and fed into classical machine learning models for pathogenicity prediction.

## Overview

Missense mutations alter a single amino acid in a protein sequence. While some mutations are harmless, others can disrupt protein function and contribute to disease.

The goal of this project is to build a practical mutation-effect prediction pipeline that:

* Uses real-world clinical variant data from ClinVar
* Incorporates protein annotations from UniProt
* Extracts mutation-aware embeddings using ESM-2
* Combines deep protein representations with interpretable biological features
* Predicts whether a variant is benign or pathogenic

## Dataset

The dataset was constructed from ClinVar missense variants belonging to nine disease-associated human genes:

* TP53
* PTEN
* LDLR
* CFTR
* BRCA1
* BRCA2
* PAH
* HBB
* G6PD

After filtering, balancing, and sequence verification:

| Dataset Component                       | Count |
| --------------------------------------- | ----- |
| Labeled variants                        | 1,441 |
| Benign / Likely Benign                  | 445   |
| Pathogenic / Likely Pathogenic          | 996   |
| Genes                                   | 9     |
| VUS / Conflicting variants (case study) | 60    |

Variants of Uncertain Significance (VUS) were excluded from training and used separately for model evaluation and prioritization.

## Methodology

### 1. Variant Collection

* ClinVar was used to obtain missense variants and clinical significance labels.
* Only benign/likely benign and pathogenic/likely pathogenic variants were used for supervised training.

### 2. Protein Annotation

For each target gene:

* Canonical protein sequences were downloaded from UniProt.
* Functional regions, domains, binding sites, and other protein annotations were extracted.

### 3. Sequence Window Generation

To keep ESM-2 inference computationally feasible, local protein windows centered around each mutation were generated.

For every variant:

* Wild-type sequence window
* Mutant sequence window

were created and used as model inputs.

### 4. Feature Engineering

#### ESM-2 Features

Using the ESM-2 (35M parameter) protein language model:

* Mean embedding differences
* Mutation-site embedding differences
* Cosine distances
* L2 distances

between wild-type and mutant proteins were extracted.

#### Biological & Domain-Aware Features

Additional interpretable features include:

* BLOSUM62 substitution score
* Hydropathy change
* Molecular mass change
* Charge change
* Mutation position
* Protein length
* UniProt domain indicators
* Binding-site indicators
* Functional-region counts

### 5. Classification Models

The following models were evaluated:

* Logistic Regression
* Random Forest

Across three feature configurations:

1. ESM-only
2. Bio/Domain-only
3. ESM + Bio/Domain (proposed model)

## Results

### Model Comparison

| Model                          | Accuracy   | Precision  | Recall     | F1 Score   | ROC-AUC   |
| ------------------------------ | ---------- | ---------- | ---------- | ---------- | --------- |
| ESM + Bio/Domain Random Forest | **84.49%** | **84.15%** | **95.60%** | **89.51%** | 0.887     |
| Bio/Domain Random Forest       | 83.10%     | 84.62%     | 92.40%     | 88.34%     | **0.900** |
| ESM-only Random Forest         | 74.52%     | 77.24%     | 89.60%     | 82.96%     | 0.745     |

The combined ESM + Bio/Domain Random Forest model achieved the best practical performance and was selected as the final model.

## Key Findings

* ESM-2 embeddings alone contain useful mutation information.
* Biological and domain-aware features remain highly informative.
* Combining ESM-derived features with interpretable biological features consistently improves prediction performance.
* Frozen protein language model embeddings can be leveraged effectively without expensive fine-tuning.

## Feature Importance

The most influential features included:

* Protein length
* Window length
* Normalized mutation position
* ESM site cosine distance
* ESM mean cosine distance
* BLOSUM62 score
* UniProt feature count

This indicates that both learned protein representations and biological context contribute to prediction quality.

## Technologies Used

* Python
* PyTorch
* Transformers (Hugging Face)
* ESM-2
* Scikit-learn
* Pandas
* NumPy
* Biopython
* Matplotlib

## Project Structure

```text
.
├── Missense_Mutation_Pathogenicity_Prediction.py
├── CSE443_Project_Report.pdf
├── model_comparison_results.csv
├── hyperparameter_sensitivity_results.csv
└── README.md
```

## Limitations

* Dataset imbalance between benign and pathogenic variants.
* Only nine genes were included.
* Local sequence windows may miss long-range protein interactions.
* Evaluation was performed on genes that were also present in training.
* Predictions should be treated as computational prioritization rather than clinical diagnoses.

## Future Work

* Evaluate on completely unseen genes.
* Incorporate structural information from AlphaFold.
* Explore larger ESM models.
* Investigate graph-based and multimodal protein representations.
* Expand training data using additional variant databases.

## Authors

**Anindya Dasgupta**
**Arpita Sarkar**
