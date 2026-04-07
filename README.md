# Retrieval-Augmented Generation vs Fine-Tuning for Low-Resource Question Answering

## Overview
This repository contains the code and data used in the study **"Retrieval-Augmented Generation vs Fine-Tuning for Low-Resource Question Answering"**. The project provides a controlled comparison between:

1. **Fine-tuning** an extractive question answering (QA) model on Turkish QA data, and
2. **Retrieval-augmented QA** pipelines that first retrieve candidate contexts and then apply the same reader model.

The experiments are designed for a **low-resource Turkish question answering setting** and compare three retrieval strategies:

- **BM25** (sparse lexical retrieval)
- **Dense retrieval** using multilingual sentence embeddings
- **Hybrid retrieval** combining BM25 and dense similarity scores

The repository is intended to support **reproducibility** of the experiments reported in the manuscript.

---

## Repository Contents

- `Retrieval-Augmented Generation vs Fine-Tuning for Low-Resource Question Answering.ipynb`  
  Main notebook containing data loading, train/validation/test splitting, fine-tuning, retrieval construction, evaluation, and result export.

- `combined_data.json`  
  SQuAD-style Turkish question answering dataset file used by the notebook.

- `LICENSE`  
  MIT License.

> Recommended addition: keep this `README.md` file at the repository root so users and reviewers can immediately understand how to run the project.

---

## Dataset Information
The experiments use a **Turkish question answering dataset in SQuAD-like JSON format**. The local file included in this repository is:

- `combined_data.json`

### Original third-party data source
The dataset used in this study is based on the Turkish reading comprehension / question answering resource commonly referred to as **THQuAD**:

- GitHub source: <https://github.com/okanvk/Turkish-Reading-Comprehension-Question-Answering-Dataset>
- Associated publication:  
  Soygazi, F., Çiftçi, O., Kök, U., & Cengiz, S. (2021). *THQuAD: Turkish Historic Question Answering Dataset for Reading Comprehension*. 2021 6th International Conference on Computer Science and Engineering (UBMK), 215–220.  
  DOI: <https://doi.org/10.1109/UBMK52708.2021.9559013>

### Dataset format
The notebook expects a SQuAD-style structure similar to the following:

```json
{
  "data": [
    {
      "title": "...",
      "paragraphs": [
        {
          "context": "...",
          "qas": [
            {
              "question": "...",
              "answers": [
                {
                  "text": "...",
                  "answer_start": 123
                }
              ]
            }
          ]
        }
      ]
    }
  ]
}
```

### Notes on the included data file
- The notebook reads data from `combined_data.json`.
- The file should be UTF-8 encoded.
- For journal submission and reviewer accessibility, all comments, metadata notes, and descriptive text inside shared data files should be in **English**.

---

## Code Information
The main implementation is in the Jupyter notebook:

- `Retrieval-Augmented Generation vs Fine-Tuning for Low-Resource Question Answering.ipynb`

### What the notebook does
The notebook performs the following steps:

1. Mounts Google Drive in Google Colab.
2. Loads the Turkish QA dataset from `combined_data.json`.
3. Splits the data into **train / validation / test** subsets.
4. Fine-tunes a Turkish BERT extractive QA reader.
5. Builds three retrieval settings:
   - BM25
   - Dense retrieval
   - Hybrid retrieval
6. Evaluates all systems using:
   - **Exact Match (EM)**
   - **Token-level F1**
   - **Recall@1** and **Recall@5** for retrieval-based pipelines
7. Exports result files such as:
   - `final_results.csv`
   - `qualitative_examples.csv`

### Core models used
- **Reader model:** `dbmdz/bert-base-turkish-cased`
- **Dense retriever:** `sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2`

### Main experimental settings currently used in the notebook
- Random seed: `42`
- Max sequence length: `384`
- Document stride: `128`
- Top-k retrieved contexts: `5`
- Learning rate: `3e-5`
- Batch size: `8`
- Number of training epochs: `2`
- Hybrid retrieval weight: `alpha = 0.5`

---

## Data Preprocessing
The preprocessing steps applied in the notebook are intentionally lightweight:

1. The SQuAD-style JSON file is parsed into flat QA examples.
2. Examples without answers are skipped.
3. Context, question, answer text, and answer start positions are extracted.
4. Text normalization is applied during evaluation:
   - lowercasing
   - stripping leading/trailing whitespace
   - collapsing repeated whitespace
5. Contexts from the **training partition only** are used as the retrieval corpus in order to avoid test-set leakage.

No additional stemming, lemmatization, or external linguistic preprocessing is applied in the current implementation.

---

## Methodology Summary
This repository implements a **controlled comparison** between fine-tuning and retrieval-augmented question answering.

### Fine-tuning setup
A Turkish BERT model is fine-tuned for extractive QA using supervised question-answer pairs. In this setting, the model receives the **gold context** associated with each question.

### Retrieval-augmented setup
For retrieval-augmented QA, the system first retrieves candidate contexts from the training corpus and then applies the same QA reader to the top-ranked contexts.

Three retrieval methods are compared:

- **BM25:** lexical retrieval baseline
- **Dense retrieval:** semantic retrieval using sentence embeddings
- **Hybrid retrieval:** weighted combination of BM25 and dense retrieval scores

### Evaluation procedure
The study is evaluated using a held-out test set and reports:

- **Exact Match (EM)**
- **F1 score**
- **Recall@1**
- **Recall@5**

This design allows the project to measure both:
- end-to-end answer quality, and
- the effect of retrieval quality on downstream QA performance.

---

## Requirements
The notebook was written for a **Python / Google Colab** workflow.

### Required Python packages
The notebook installs missing dependencies automatically, but the core packages are:

- `python>=3.9`
- `torch`
- `transformers`
- `datasets`
- `sentence-transformers`
- `rank-bm25`
- `scikit-learn`
- `pandas`
- `numpy`

### Recommended environment
- Google Colab or a local Python environment with notebook support
- CUDA-enabled GPU is recommended for faster fine-tuning, but CPU execution is possible for smaller-scale testing

### Optional installation command
If you want to install dependencies manually, you can use:

```bash
pip install torch transformers datasets sentence-transformers rank-bm25 scikit-learn pandas numpy
```

---

## Usage Instructions

### Option A — Run in Google Colab (recommended)
1. Clone or download this repository.
2. Upload `combined_data.json` to your Google Drive.
3. Open the notebook in Google Colab.
4. Update the dataset path if needed. The notebook currently expects:

```python
DATA_PATH = r"/content/drive/MyDrive/Murat_Calisma/lokalize_hibrit/combined_data.json"
```

5. Run all notebook cells in order.
6. Review the output metrics and exported result files.

### Option B — Run locally
1. Clone the repository:

```bash
git clone https://github.com/muratsimsek003/Retrieval-Augmented-Generation-vs-Fine-Tuning-for-Low-Resource-Question-Answering.git
cd Retrieval-Augmented-Generation-vs-Fine-Tuning-for-Low-Resource-Question-Answering
```

2. Install the required packages.
3. Edit the `DATA_PATH` variable in the notebook so it points to your local `combined_data.json` file.
4. Run the notebook using Jupyter Notebook or JupyterLab.

---

## Expected Outputs
After a successful run, the workflow generates summary outputs such as:

- `final_results.csv` — main quantitative results
- `qualitative_examples.csv` — example predictions for qualitative analysis

You may also save additional figures, tables, or trained checkpoints depending on your execution environment.

---

## Reproducibility Notes
To improve reproducibility:

- Keep the random seed fixed (`SEED = 42`).
- Use the same dataset file version for all runs.
- Keep train/validation/test splitting consistent.
- Do not retrieve from validation or test contexts.
- Document the execution environment used for the final reported results (OS, CPU/GPU, RAM, package versions).

For journal submission, it is also recommended to add:

- a `requirements.txt` file with pinned package versions,
- a release tag or archived version of the repository,
- and, if available, supplementary output files used in the manuscript.

---

## Limitations
This repository reflects the implementation used for the study, but several limitations should be noted:

- Package versions are not pinned in a separate `requirements.txt` file yet.
- The notebook is currently optimized for a Colab-based workflow.
- The data path is hard-coded and should be edited before reuse.
- Retrieval is evaluated on the available corpus only; performance may differ with alternative corpora or broader document collections.
- The repository currently contains one notebook rather than a modular Python package.

---

## Citation
If you use this repository, please cite both the dataset source and the corresponding study.

### Dataset citation
```bibtex
@inproceedings{soygazi2021thquad,
  author    = {Soygazi, Fatih and Çiftçi, Okan and Kök, Uğur and Cengiz, Soner},
  title     = {THQuAD: Turkish Historic Question Answering Dataset for Reading Comprehension},
  booktitle = {2021 6th International Conference on Computer Science and Engineering (UBMK)},
  pages     = {215--220},
  year      = {2021},
  doi       = {10.1109/UBMK52708.2021.9559013}
}
```

### Repository / manuscript citation
Please add the final article citation here after publication.

```bibtex
@article{simsek2026ragft,
  title   = {Retrieval-Augmented Generation vs Fine-Tuning for Low-Resource Question Answering},
  author  = {Simsek, Murat},
  journal = {PeerJ Computer Science},
  year    = {2026},
  note    = {Under review at the time of repository preparation}
}
```

---

## License
This repository is distributed under the **MIT License**. See the `LICENSE` file for details.

---

## Contribution Guidelines
Contributions that improve reproducibility and clarity are welcome. Useful contributions include:

- improving documentation,
- translating remaining non-English comments into English,
- adding `requirements.txt`,
- modularizing the notebook into reusable scripts,
- and reporting issues related to reproducibility.

If you contribute, please:

1. fork the repository,
2. create a feature branch,
3. commit clear changes,
4. and submit a pull request with a concise explanation.

---

## Contact
For questions about the repository or the study, please contact:

**Murat Simsek**  
Department of Artificial Intelligence Engineering  
Ostim Technical University, Ankara, Türkiye  
Email: `murat.simsek@ostimteknik.edu.tr`
