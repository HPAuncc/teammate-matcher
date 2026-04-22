# Teammate Matcher

> ML-powered student team formation using clustering and assignment optimization.

A data science final project for DTSC 2302 at UNC Charlotte. The system takes anonymous survey responses from students and produces balanced team configurations — optimizing for either scheduling compatibility or complementary skill coverage, depending on what the instructor needs.

---

## Motivation

Random group assignment produces uneven workloads, schedule conflicts, and mismatched work styles. Existing tools rely on indirect behavioral proxies. This project uses primary survey data — actual self-reported availability, skills, and work style — to form teams more thoughtfully.

The system outputs a **ranked list of team configurations with interpretable metrics**, not a final assignment. The instructor stays in the loop.

---

## How It Works

Two distinct matching objectives are treated separately:

- **Similarity** — group students with compatible schedules, work styles, and communication preferences to minimize friction
- **Complementarity** — distribute technical skills across teams so no group is uniformly weak in any area

Four models are implemented and compared:

| Model | Objective | Key feature |
|---|---|---|
| K-Means | Similarity | Baseline clustering; elbow + silhouette for k selection |
| Agglomerative (Ward) | Similarity | Dendrogram for visual interpretation |
| Hungarian Assignment | Similarity + size constraint | Guarantees equal team sizes via `scipy.optimize.linear_sum_assignment` |
| Gaussian Mixture Model | Complementarity | Soft assignments surface students who span multiple skill profiles |

The Hungarian Algorithm is the primary deployment model. K-Means provides centroids; the assignment problem enforces `⌊N/k⌋` members per team.

---

## Dataset

Original anonymous student survey (Google Forms), collected within the course. No Kaggle, no UCI Repository.

**~35 features per student record across 5 categories:**

| Category | Features |
|---|---|
| Schedule & availability | `avail_mon` – `avail_sun`, `avail_morning/afternoon/evening/latenight` |
| Time commitment & meeting mode | `weekly_hours`, `meeting_mode` |
| Technical skills (self-rated 1–5) | `skill_python`, `skill_data_analysis`, `skill_statistics`, `skill_visualization`, `skill_ml`, `skill_writing`, `skill_research`, `skill_presenting` |
| Work style | `role_pref`, `deadline_style`, `comm_pref`, `checkin_freq`, `collab_style`, `detail_orientation`, `conflict_style` |
| Self-assessment | `gpa_band` (optional), `contrib_*`, `pain_point` |

Survey instrument: [`survey/survey_questions.md`](survey/survey_questions.md)

---

## Project Structure

```
teammate-matcher/
├── data/
│   ├── raw_survey_data.csv          # Google Forms export (not committed)
│   └── processed_survey_data.csv    # Output of preprocessing pipeline
├── survey/
│   └── survey_questions.md          # Full survey instrument with feature map
├── notebooks/
│   ├── 01_eda.ipynb                 # Exploratory data analysis
│   ├── 02_preprocessing.ipynb       # Full preprocessing pipeline
│   ├── 03_models_1_2.ipynb          # K-Means & Agglomerative
│   ├── 04_models_3_4.ipynb          # Hungarian Algorithm & GMM
│   └── 05_evaluation.ipynb          # Comparison table, PCA biplot
├── src/
│   ├── preprocess.py                # Reusable preprocessing functions
│   ├── models.py                    # Model wrappers
│   └── evaluate.py                  # Metric calculations
├── outputs/
│   └── team_assignments/            # Generated team configurations
├── CLAUDE.md                        # Developer context for AI-assisted work
└── README.md
```

---

## Setup

```bash
git clone https://github.com/HPAuncc/teammate-matcher.git
cd teammate-matcher
pip install -r requirements.txt
```

Run notebooks in order (`01` → `05`). Place your Google Forms CSV export at `data/raw_survey_data.csv` before running `02_preprocessing.ipynb`.

**Requirements:** Python 3.x, pandas, NumPy, scikit-learn, scipy, matplotlib, seaborn, jupyter

---

## Evaluation

Six metrics are reported across all four models:

| Metric | What it measures |
|---|---|
| Silhouette Score | Cluster cohesion vs. separation (higher = better) |
| Davies-Bouldin Index | Avg. similarity of each cluster to its nearest neighbor (lower = better) |
| Calinski-Harabasz Index | Ratio of between- to within-cluster dispersion (higher = better) |
| Intra-team Skill Variance | Std dev of skill ratings within each team |
| Schedule Overlap Score | Mean Jaccard similarity of availability vectors within teams |
| Skill Coverage Score | Skill dimensions where ≥1 member scores ≥4 per team |

Results are compared in a single table so the "best" model depends explicitly on what the instructor is optimizing for.

---

## Ethics & Limitations

- **Anonymous by design** — no names, emails, or student IDs are collected
- **Self-report bias** — skill ratings are used relatively (for comparison) not absolutely
- **Human-in-the-loop** — output is a recommendation, not a final assignment; instructors can account for context the model cannot (conflicts, accommodations, etc.)
- **Equity constraint** — the complementarity model enforces minimum skill diversity so no team is composed entirely of students who self-rate below threshold on any one dimension
- **Static snapshot** — the model cannot account for how team dynamics evolve after assignment
- **Sample size** — single class section (~25–50 responses) limits generalizability

---

## Course Context

**DTSC 2302** — Data Science II, UNC Charlotte  
Final group project | 100 points | Due May 5, 2026  
Poster presentation: May 1, 2026 (8–10:30 AM)

---

## References

1. Kyprianidou et al., "Group formation based on learning styles," *Educational Technology Research and Development*, 2012.
2. Kuhn, H.W., "The Hungarian method for the assignment problem," *Naval Research Logistics Quarterly*, 1955.
3. Akgun, S., "Artificial intelligence in education: Addressing ethical challenges," *PMC*, 2021.
4. Bishop, C.M., *Pattern Recognition and Machine Learning*, Springer, 2006. (GMM, Ch. 9)
5. Pedregosa et al., "Scikit-learn: Machine Learning in Python," *JMLR*, 2011.
