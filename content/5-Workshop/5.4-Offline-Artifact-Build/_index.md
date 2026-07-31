---
title: "Offline Artifact Build"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

### Offline Artifact Build

This step prepares the data and creates the necessary index files.

#### 1. CLI Steps: Notebook Environment
Run the `build_s3_offline_artifacts.ipynb` notebook located in the `backend/notebooks/` directory. Ensure you have the necessary libraries installed:

```bash
pip install jupyter pandas numpy scikit-learn
```

#### 2. Notebook UI Steps: Dataset Configuration
Open the Jupyter Notebook, locate the data configuration section, and adjust the dataset size as needed (e.g., 500 samples for the demo):

```python
# Configuration in the notebook
DATASET_SIZE = 500
```

#### 3. Notebook UI Steps: Execution
Execute all cells in the notebook. Once completed, the necessary files (`corpus.jsonl`, `index_manifest.json`, etc.) will be generated in the `output/` directory.