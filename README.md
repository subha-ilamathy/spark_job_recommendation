# Personalized Job Recommendation System

A lightweight, end‑to‑end project that recommends relevant job roles to a user based on their skills, using classical similarity‑based matching (Collaborative Filtering over a skills×jobs matrix) and Association Rule Mining (Apriori).

---

## Table of Contents
- [Overview](#overview)
- [Key Features](#key-features)
- [System Architecture](#system-architecture)
- [Data & Preprocessing](#data--preprocessing)
- [Installation](#installation)
- [Quickstart](#quickstart)
- [Configuration](#configuration)
- [Evaluation Results](#evaluation-results)
- [Project Structure](#project-structure)
- [Reproducibility](#reproducibility)
- [References](#references)
- [License](#license)

---

## Overview
Traditional job search often overwhelms candidates with choices and inconsistent relevance. This project builds a **personalized job recommendation system** that matches a user’s **skill profile** to **job requirements** using:

1) **Collaborative Filtering (CF)** on a job–skills binary matrix with multiple **similarity metrics** (Cosine, Jaccard, Manhattan, Euclidean) to rank matches.
2) **Association Rule Mining (Apriori)** to learn frequent **skill ↔ job** patterns and generate interpretable recommendations.

The aim is to return (a) **Top‑K job recommendations** and (b) **Highest‑score matches**, along with interpretable rules learned from the data.

---

## Key Features
- **Skill‑aware matching** via CF over a jobs×skills presence matrix
- **Multiple similarity metrics**: Cosine, Jaccard, Manhattan, Euclidean
- **Apriori rules** to surface human‑readable skill→job associations
- **Two output modes**: Top‑K list and highest‑score suggestions
- **Visuals** (optional): skills word cloud, radar/plotted summaries

---

## System Architecture
**Core entities**
- **Users & Skills**: user profiles with skill ratings/tags
- **Jobs**: titles, descriptions, required skills, org metadata
- **Recommender**: computes job–user compatibility and ranks jobs

**Pipelines**
- **CF pipeline**: build jobs×skills matrix → choose similarity → score & rank → Top‑K
- **Apriori pipeline**: convert jobs to skill transactions → mine frequent itemsets → generate rules → (optionally) filter by support, confidence, lift

---

## Data & Preprocessing
**Expected fields (per job)**: country, city, location, title, description, sector, type, **required skills**, organization, posting dates, validity, salary (when available).

**Preprocessing steps**
1. **Join/clean** job offers with organization info.
2. **Extract** job titles and **parse skills** from descriptions.
3. **Deduplicate**: unique sets of jobs and skills.
4. **Build matrix (CF)**: rows=jobs, cols=skills; cell = 1 if skill appears in the job.
5. **Tokenize** skills where needed (e.g., bag‑of‑words for text fields).
6. **Transactions (Apriori)**: each job as a set of skills for rule mining.

> Tip: Keep skill vocabularies normalized (e.g., lowercasing, stemming/lemmatization, synonyms mapping like *javascript* ↔ *js*).

---

## Installation
**Prerequisites**
- Python 3.9+

**Recommended setup**
```bash
# create & activate a virtual environment
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate

# install core libraries
pip install -U pandas numpy scikit-learn mlxtend matplotlib jupyter ipykernel
```

(Optional) for visualizations:
```bash
pip install wordcloud plotly
```

---

## Quickstart
### 1) Prepare data
Place your raw job listings under `data/raw/` (CSV/JSON). Provide a column with **job title** and **required skills** (comma‑separated or list).

### 2) Build artifacts
- **Jobs×Skills matrix**: parse skills → build binary matrix.
- **Transactions**: `[[skillA, skillB, ...], ...]` per job.

### 3) Run the notebook (recommended)
Open `notebooks/JobRecommender.ipynb` and execute cells to:
- Clean data & build matrices
- Choose a similarity metric (default: **cosine**)
- Generate Top‑K recommendations for a sample user skill profile
- Run Apriori, inspect top rules, and refine thresholds (support/confidence/lift)

### 4) Example (minimal) CF usage in Python
```python
from recommender.cf import CFRecommender

user_skills = {"python", "sql", "pandas", "machine learning"}
rec = CFRecommender(metric="cosine", top_k=5)
rec.fit(jobs_skills_matrix, job_titles)
print(rec.recommend(user_skills))
```

### 5) Example (minimal) Apriori usage in Python
```python
from recommender.rules import mine_rules

rules = mine_rules(transactions, min_support=0.05, min_confidence=0.3)
# Filter rules whose antecedents are a subset of user skills
user_rules = [r for r in rules if set(r.antecedents) <= user_skills]
```

---

## Configuration
Create a `config.yaml` (optional):
```yaml
similarity_metric: cosine   # cosine|jaccard|manhattan|euclidean
k: 5                        # Top‑K recommendations
min_support: 0.05           # Apriori
min_confidence: 0.30        # Apriori
min_lift: 1.10              # optional filter
skills_normalization: true  # lower, strip, map synonyms
```

---

## Evaluation Results
**Similarity‑based CF (illustrative results):**
- **Cosine**: Accuracy ≈ **92%**, Recall ≈ **0.92**, F1 ≈ **92.98**
- **Jaccard**: Accuracy ≈ **90%**, Recall ≈ **0.90**, F1 ≈ **90.83**
- **Manhattan**: Accuracy ≈ **84%**, Recall ≈ **0.84**, F1 ≈ **85.83**
- **Euclidean**: Accuracy ≈ **82%**, Recall ≈ **0.82**, F1 ≈ **84.73**

**Association Rule Mining (Apriori):**
- Supports interpretable **skill→job** insights
- Assess rules by **support**, **confidence**, **lift**, **leverage**, **conviction**, **Zhang’s metric**

> Notes: Metrics depend on the dataset, preprocessing (e.g., skills parsing), and thresholds. Re‑run evaluation after changing normalization or filters.




### Changelog
- v1.0: Initial release (CF + Apriori, metrics, notebook)
