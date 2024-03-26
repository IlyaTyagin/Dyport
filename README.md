# Dyport: Dynamic Importance-based Biomedical Hypothesis Generation Benchmarking Technique

## Overview:
Dyport is a novel benchmarking framework for evaluating biomedical hypothesis generation systems. Utilizing curated datasets, our approach tests these systems under realistic conditions, enhancing the relevance of our evaluations. We integrate knowledge from the curated databases into a dynamic graph, accompanied by a method to quantify discovery importance. This not only assesses hypothesis accuracy but also their potential impact in biomedical research which significantly extends traditional link prediction benchmarks. Applicability of our benchmarking process is demonstrated on several link prediction systems applied on biomedical semantic knowledge graphs. Being flexible, our benchmarking system is designed for broad application in hypothesis generation quality verification, aiming to expand the scope of scientific discovery within the biomedical research community.

Arxiv preprint link: https://arxiv.org/abs/2312.03303 

# Data
Full benchmark data is available here: [Google Drive](https://drive.google.com/drive/folders/1tngJ2BU5MmIyHCoyyIkPLLCUEb-WHjI1)

## Data description

The data is represented as `.csv` tables with pairwise concept associations stratified by the year of their first occurrence in literature.
For each positive connection $e$ we calculate a range of features describing its _importance_ using the network and literature-based data.
After that the final importance score is calculated as the average percentile rank of the underlying features.

Each `.csv` file has the following structure:
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>source</th>
      <th>target</th>
      <th>source_st</th>
      <th>target_st</th>
      <th>origin_db</th>
      <th>mentions_diff</th>
      <th>sem_scholar_cit</th>
      <th>ig</th>
      <th>jac_2nd_train</th>
      <th>eig_cent_diff</th>
      <th>betw_cent</th>
      <th>IMP_manh</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>42241</th>
      <td>C4078312__acalabrutinib</td>
      <td>C4079830__venetoclax</td>
      <td>Pharmacologic Substance; Organic Chemical</td>
      <td>Pharmacologic Substance; Organic Chemical</td>
      <td>rxnav</td>
      <td>62</td>
      <td>975.0</td>
      <td>0.074084</td>
      <td>0.077561</td>
      <td>0.000203</td>
      <td>0.000019</td>
      <td>0.956588</td>
    </tr>
    <tr>
      <th>22993</th>
      <td>C2353951__dapagliflozin</td>
      <td>C4524093__Euglycaemic diabetic ketoacidosis</td>
      <td>Pharmacologic Substance; Organic Chemical</td>
      <td>Disease or Syndrome</td>
      <td>drugcentral</td>
      <td>30</td>
      <td>427.0</td>
      <td>0.103463</td>
      <td>0.041605</td>
      <td>0.000261</td>
      <td>0.000050</td>
      <td>0.965584</td>
    </tr>
    <tr>
      <th>8080</th>
      <td>C1335212__PIK3CA gene</td>
      <td>C4049141__copanlisib</td>
      <td>Gene or Genome</td>
      <td>Pharmacologic Substance; Organic Chemical</td>
      <td>drugcentral; kegg</td>
      <td>17</td>
      <td>1066.0</td>
      <td>0.067049</td>
      <td>0.025928</td>
      <td>0.000923</td>
      <td>0.000018</td>
      <td>0.974659</td>
    </tr>
  </tbody>
</table>

where:
- `source` and `target` columns represent UMLS CUI terms and their corresponding preferred names (separated with `__`);
- `source_st` and `target_st` columns contain their semantic types;
- `origin_db` shows what biomedical database a particular connection came from;

Next columns represent the importance components:
- `mentions_diff`: number of times a particular association was mentioned in the literature;
- `sem_scholar_cit`: number of Semantic Scholar citations;
- `ig`: Integrated Gradients attribution score;
- `jac_2nd_train`: 2nd order Jaccard similarity;
- `eig_cent_diff`: eigenvector centrality difference;
- `betw_cent`: betweenness centrality;

The last column `IMP_manh` is a final combined importance score (a number between 0 and 1).

Note: the importance components described above are the ones we reported in our [paper](https://arxiv.org/abs/2312.03303). If you want to use your own custom component(s), you can calculate them for every connection (alongside already existing components) and simply add them to the list. `IMP_manh` should be recalculated accordingly at the end to get the final combined importance score for every connection. Please refer to [this illustrated notebook](https://github.com/IlyaTyagin/Dyport/blob/main/04_models_evaluation_code/e09_imp_vs_perf.ipynb), part - `Calculating merged importance metric`.

# Proposed Use Case

We shared the output numbers for every model we tested (for both positive and negative samples) and then we computed ROC AUC scores based on different stratification strategies. 

If you would like to use the benchmark, this might be a course of actions:
1. Train your model. The model should accept two UMLS terms as input and output a number indicating the likelihood of two terms being connected. The training data should come from scientific publications (or relevant literature)  prior to the testing timestamp `<YEAR>`.
2. Download a table `all_model_scores_test_<YEAR>.csv` from [Google Drive](https://drive.google.com/drive/folders/1tngJ2BU5MmIyHCoyyIkPLLCUEb-WHjI1), where `<YEAR>` represents the year of interest and indicates the timestamp when the associations first appeared in the literature.
3. Compute your model output for associations from step 2 and calculate ROC AUC score with, for example, [this scikit-learn implementation](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.roc_auc_score.html).

# Current leaderboard 
Dataset time split:
- train: 2015
- test: 2016
- importance: 2022

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>AGATHA</th>
      <th>DistMult</th>
      <th>Node2Vec</th>
      <th>HolE</th>
      <th>ComplEx</th>
      <th>TransE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Low importance ( < 0.41)</th>
      <td>0.774</td>
      <td>0.760</td>
      <td>0.755</td>
      <td>0.623</td>
      <td>0.577</td>
      <td>0.555</td>
    </tr>
    <tr>
      <th>Medium importance (0.41 $\leq$ $I_t$ $\leq$ 0.53)</th>
      <td>0.698</td>
      <td>0.674</td>
      <td>0.666</td>
      <td>0.588</td>
      <td>0.559</td>
      <td>0.541</td>
    </tr>
    <tr>
      <th>High importance ( > 0.53)</th>
      <td>0.638</td>
      <td>0.623</td>
      <td>0.603</td>
      <td>0.566</td>
      <td>0.551</td>
      <td>0.533</td>
    </tr>
  </tbody>
</table>

Want to upload your results? Open an issue on GitHub! Or reach out to me via email: tyagin at udel dot edu

# Acknowledgements

This research was supported by NIH award #R01DA054992. The computational experiments were supported in part through the use of DARWIN computing system: DARWIN - A Resource for Computational and Data-intensive Research at the University of Delaware and in the Delaware Region, which is supported by NSF Grant #1919839.

# Citing Dyport

I. Tyagin, I. Safro. Dyport: Dynamic Importance-based Hypothesis Generation Benchmarking Technique. 2023

```
@article{tyagin2023dyport,
      title={Dyport: Dynamic Importance-based Hypothesis Generation Benchmarking Technique}, 
      author={Ilya Tyagin and Ilya Safro},
      year={2023},
      eprint={2312.03303},
      archivePrefix={arXiv},
      primaryClass={cs.AI}
}
```
