# Final Project Proposal: Group Project Teammate Matcher

> **Note:** This is the original project proposal, written before the
> survey was deployed and the analysis run. Findings, methods, and final
> claims may have evolved during execution — see
> [`technical_report.md`](technical_report.md) for the canonical write-up
> of the completed project, and
> [`portfolio_post.md`](portfolio_post.md) for the public-facing version.
> This file is preserved as project history.

## Problem, Motivation & Research Questions

The success of group projects in academic settings is heavily dependent on team
composition. Random assignment or self-selection often leads to suboptimal outcomes,
including uneven distribution of work, schedule conflicts, and mismatched work styles [1].
These issues not only hinder the educational objectives of collaborative assignments but also
cause significant frustration among students.

The proposed **Group Project Teammate Matcher** addresses this problem by developing an
automated, data-driven application that optimizes team formation based on directly measured
student attributes: self-reported technical skills, weekly availability, communication
preferences, and work style. Unlike prior approaches that rely on indirect behavioral proxies,
this system uses primary survey data collected from real students, making it directly
applicable to the context it aims to improve.

This project is motivated by the need for more equitable and productive collaborative
learning environments. By leveraging machine learning to group students thoughtfully, we can
maximize team synergy and minimize the friction that typically plagues group assignments.

The primary research questions guiding this project are:

1. How can we quantify and represent student skills, availability, and work styles using
   direct self-report to facilitate effective team matching?
2. Which clustering or optimization algorithms produce the most balanced and compatible
   student teams — and does the answer differ when optimizing for *similar* teams versus
   *complementary* teams?
3. What student attributes are most predictive of perceived team compatibility, and which
   features contribute most to cluster separation?

> **Note on RQ framing:** Prior work often conflates two distinct goals — grouping *similar*
> students (maximizing intra-team consistency in schedule and work style) and grouping
> *complementary* students (maximizing skill diversity within teams). This project explicitly
> treats these as separate objectives and evaluates models against both.

---

## Dataset & Data Collection

### Primary Data Source: Original Student Survey

To ensure a real-world, directly relevant dataset free of proxy assumptions, we collected
primary data via an anonymous student survey distributed within our course. The survey was
designed specifically to capture the attributes most relevant to team formation:

| Feature Category | Fields | Encoding |
|---|---|---|
| Availability | 7 day-of-week flags, 4 time-of-day flags | Binary (11 columns) |
| Time commitment | Hours/week available | Ordinal (1–4) |
| Meeting preference | In-person vs. remote | One-hot |
| Technical skills | 8 self-rated dimensions (1–5 scale) | Continuous |
| Work style | Role preference, deadline style, collaboration style, detail orientation | Ordinal/one-hot |
| Communication | Preferred channel, check-in frequency | One-hot/ordinal |
| Self-assessment | GPA band, top contributions, biggest challenge | Ordinal/binary |

**Total features:** ~35 per student record.

This approach has several advantages over secondary datasets:

- **Schedule data is direct** — students report actual availability, not inferred from
  historical log timestamps.
- **Skills are self-assessed** — directly capturing the 8 dimensions most relevant to data
  science coursework, rather than proxied from click counts on learning management
  systems.
- **Work style is measured explicitly** — no secondary dataset contains information about
  deadline behavior, conflict resolution style, or collaboration preferences.
- **Sample is directly relevant** — these are the exact students who will be assigned to teams.

### Survey Design

The survey was administered anonymously via Google Forms and contains five sections:

1. **Context** — course code, year in school
2. **Schedule & Availability** — days, time slots, weekly hours, meeting mode preference
3. **Technical Skills** — 8 items, Likert 1–5
4. **Work Style** — role preference, deadline style, communication preference, check-in
   frequency, collaboration style, detail orientation, conflict approach
5. **Self-Assessment** — GPA band (optional), top contributions, biggest pain point in
   group work

Full survey instrument is available in `survey/survey_questions.md`.

### Data Limitations & Assumptions

- **Self-report bias:** Students may over- or under-rate their own skills. We treat ratings as
  relative rather than absolute, which is appropriate for comparison-based clustering.
- **Sample size:** With a single class section, we expect 25–50 responses. This is sufficient
  for clustering into teams of 3–4 but limits generalizability.
- **GPA is optional:** Missing GPA values are imputed with the median band. Sensitivity
  analysis is run with and without GPA to assess its influence.
- **Static snapshot:** The survey captures preferences at one point in time. Team dynamics
  evolve, which the model cannot account for.

---

## Data Preprocessing & Feature Engineering

The raw survey export (CSV from Google Forms) is processed through the following pipeline:

### 1. Column Cleaning
Strip whitespace and standardize column names. Drop timestamp column (not analytically
relevant).

### 2. Availability Encoding
The 7 day-of-week and 4 time-slot checkbox columns are split into individual binary features
(one per option). Two students' availability vectors can then be compared directly via
dot product or Jaccard similarity — a direct measure of scheduling overlap.

### 3. Ordinal Encoding
Ordered categorical variables are mapped to integers:
- `year`: Freshman=1, Sophomore=2, Junior=3, Senior=4, Graduate=5
- `weekly_hours`: <3hrs=1, 3–5=2, 6–9=3, 10+=4
- `deadline_style`: Early=3, Steady=2, Last-minute=1
- `checkin_freq`: Daily=4, Few/week=3, Weekly=2, As needed=1
- `role_pref`: Follower=1, Contributor=2, Flexible=3, Leader=4
- `gpa_band`: <2.5=1, 2.5–3.0=2, 3.0–3.5=3, 3.5–4.0=4

### 4. One-Hot Encoding
Nominal variables with no natural order (meeting preference, communication preference,
conflict style, collaboration style) are one-hot encoded.

### 5. Normalization
All continuous and ordinal features are scaled to [0, 1] using Min-Max normalization before
clustering. This prevents high-variance features (e.g., skill scores 1–5) from dominating
distance calculations over binary features.

### 6. Missing Value Handling
- GPA: Imputed with median band (ordinal imputation preserves scale).
- Any other missing fields: Row flagged; if >20% of features are missing, row is excluded.

### 7. Feature Sets
We construct two distinct feature sets to evaluate the two matching philosophies:

- **Compatibility features** (used in similarity-based models): availability overlap,
  meeting preference, work style dimensions, communication style. Goal: group students
  who will work well *together*.
- **Complementarity features** (used in diversity-focused models): technical skill
  dimensions only. Goal: ensure each team has a range of skills, not a team of five
  Python experts.

The final processed dataset is saved to `data/processed_survey_data.csv`.

---

## Methodology & Experimentation Plan

The core challenge of team formation sits at the intersection of two distinct objectives that
require different algorithmic approaches:

- **Objective A (Similarity):** Group students who share compatible schedules, work styles,
  and communication preferences → minimize intra-team friction.
- **Objective B (Complementarity):** Ensure each team has a diverse skill distribution →
  maximize intra-team capability coverage.

No single clustering algorithm naturally handles both simultaneously. Our experimentation
plan tests models across both objectives.

---

### Model 1: K-Means Clustering (Baseline — Similarity)

K-Means partitions students into *k* clusters by minimizing within-cluster sum of squared
distances. Applied to the **compatibility feature set**, it produces teams of students with
similar schedules and work styles.

**Why K-Means first:** It is the most interpretable baseline and directly tests whether
pure similarity grouping produces cohesive teams. K-Means minimizes:

$$J = \sum_{j=1}^{k} \sum_{x_i \in C_j} \| x_i - \mu_j \|^2$$

where $\mu_j$ is the centroid of cluster $j$. The optimal *k* is selected using the elbow
method on inertia and confirmed with the Silhouette Score.

**Limitation:** K-Means assumes spherical clusters and requires *k* to be specified.
It also does not enforce equal team sizes, which is a practical requirement.

---

### Model 2: Agglomerative Hierarchical Clustering (Alternative Baseline — Similarity)

Agglomerative clustering builds a hierarchy of clusters bottom-up by repeatedly merging the
two most similar students/groups. Applied to the **compatibility feature set** with Ward
linkage (minimizes variance within merged clusters).

**Why agglomerative:** It makes no assumptions about cluster shape and produces a
dendrogram that visually communicates how student groupings form — useful for explaining
results to non-technical audiences (portfolio post). Ward linkage is equivalent to minimizing
the same objective as K-Means at each merge step, making comparison fair.

**Limitation:** Computationally expensive at scale; with 25–50 students this is not a
concern. Does not enforce team size constraints.

---

### Model 3: Size-Constrained Assignment via Hungarian Algorithm *(Beyond Class Content)*

**This is the primary deployment model.** The Hungarian Algorithm (also called the Linear
Sum Assignment algorithm) solves the assignment problem: given a cost matrix of
student-to-team assignments, find the assignment that minimizes total cost while satisfying
the constraint that each team has exactly *n* members [2].

**Algorithm:**

1. Run K-Means on compatibility features to obtain *k* cluster centroids.
2. Construct a cost matrix $C \in \mathbb{R}^{N \times k}$ where $C_{ij}$ is the
   Euclidean distance from student $i$ to centroid $j$.
3. Solve: $\min_{\text{assignment}} \sum_i C_{i, \sigma(i)}$ subject to each centroid
   being assigned exactly $\lfloor N/k \rfloor$ students.
4. Use `scipy.optimize.linear_sum_assignment` to solve in $O(N^3)$ time.

This directly addresses the practical limitation of standard clustering: a K-Means cluster of
8 students cannot be split into two teams of 4 without re-optimization. The Hungarian
Algorithm guarantees balanced teams while minimizing total distance from centroids.

**Why this is beyond class content:** Assignment optimization is covered in operations
research, not introductory data science. It is distinct from clustering — it is a
combinatorial optimization problem solved via dynamic programming.

---

### Model 4: Gaussian Mixture Models — GMM *(Beyond Class Content)*

GMMs extend K-Means by modeling each cluster as a multivariate Gaussian distribution
rather than a hard centroid. This produces **soft cluster assignments** — a probability
vector for each student indicating how strongly they belong to each cluster.

**Why GMM:** For team formation, a soft assignment is more informative than a hard one.
A student with $P(\text{cluster}_A) = 0.51$ and $P(\text{cluster}_B) = 0.49$ is genuinely
ambiguous — they could fit well in either team. GMM surfaces this uncertainty, allowing a
"Human-in-the-Loop" instructor to make more nuanced decisions.

Applied to the **complementarity feature set** (skills only), GMM can identify natural
skill-profile archetypes (e.g., "strong coder / weak writer," "strong communicator / weak
stats") and ensure each team contains a mix of archetypes.

---

### Experiment Summary

| Model | Feature Set | Objective | Team Size Constraint | Beyond Class? |
|---|---|---|---|---|
| K-Means | Compatibility | Similarity | No | No |
| Agglomerative | Compatibility | Similarity | No | No |
| Hungarian Assignment | Compatibility | Similarity | Yes | Yes |
| GMM | Complementarity | Skill diversity | No | Yes |

---

## Evaluation Plan

### Algorithmic Metrics

| Metric | Models | What it measures |
|---|---|---|
| Silhouette Score | K-Means, Agglomerative, GMM | Cluster cohesion vs. separation (higher = better) |
| Davies-Bouldin Index | K-Means, Agglomerative | Avg. similarity of each cluster to its most similar cluster (lower = better) |
| Calinski-Harabasz Index | K-Means, Agglomerative | Ratio of between-cluster to within-cluster dispersion (higher = better) |
| Intra-team Score Variance | All models | Std dev of skill ratings within each team (lower = more homogeneous; higher = more complementary — depends on objective) |
| Schedule Overlap Score | All models | Mean Jaccard similarity of availability vectors within each team (higher = fewer scheduling conflicts) |
| Skill Coverage Score | All models | For each team: number of skill dimensions where at least one member scores ≥ 4 (higher = more complementary) |

### Comparative Analysis

Results are reported in a single comparison table across all four models. For each model we
report all six metrics and explicitly note which metrics improve under which objective
(similarity vs. complementarity). This demonstrates that the "best" model depends on what
you're optimizing for — a finding with genuine pedagogical implications.

### Feature Importance

Principal Component Analysis (PCA) is run on the full feature set to identify which features
drive the most variance in student profiles. The top PCA components are visualized as a
biplot to answer RQ3.

---

## Team Plan & Timeline

| Week | Milestone |
|---|---|
| Week 1 | Survey deployed; responses collected; EDA on raw survey data |
| Week 2 | Preprocessing pipeline complete; Models 1 & 2 implemented and evaluated |
| Week 3 | Models 3 & 4 implemented; full comparison table; PCA analysis |
| Week 4 | Technical Report written; Public Portfolio Post drafted; Poster designed |
| May 1 | Poster presentation (8–10:30 AM) |
| May 5 | All deliverables submitted |

---

## Ethics, Impact & Reflection

### Fairness & Bias

The use of self-reported data introduces the possibility of social desirability bias — students
may rate themselves higher in skills they perceive as prestigious (e.g., machine learning)
and lower in skills that feel less technical (e.g., writing). To mitigate this, survey questions
are framed as confidence assessments rather than ability claims, and results are used
relatively (for comparison) rather than absolutely.

Demographic information is not collected beyond year in school. GPA is optional and is
treated as one of ~35 features, not a primary sorting variable. The algorithm does not have
access to race, gender, nationality, or socioeconomic indicators.

### Human-in-the-Loop Design

The system is designed as a **recommendation tool for instructors**, not an autonomous
assignment engine. The output is a ranked list of suggested team configurations with
interpretable metrics. Final decisions remain with the instructor, who can incorporate context
the algorithm does not have (e.g., known interpersonal conflicts, accommodation needs).

### Equity Considerations

Optimizing for skill complementarity may inadvertently isolate students with lower skill
self-ratings. To counter this, the system can be configured to enforce a minimum skill
diversity constraint — ensuring no team is composed entirely of students who self-rate below
a threshold on any single dimension.

Future work should include collecting post-project satisfaction scores from students to
validate whether algorithmically formed teams actually experience less friction — a
ground-truth validation not possible with secondary datasets.

### Data Privacy

Survey responses are anonymous. No names, emails, or student IDs are collected. The CSV
export is stored locally and is not published.

---

## References

[1] M. Kyprianidou, S. Demetriadis, T. Tsiatsos, and A. Pombortsis, "Group formation based
on learning styles: can it improve students' teamwork?" *Educational Technology Research
and Development*, vol. 60, pp. 83–110, 2012.

[2] H. W. Kuhn, "The Hungarian method for the assignment problem," *Naval Research
Logistics Quarterly*, vol. 2, no. 1–2, pp. 83–97, 1955.

[3] S. Akgun, "Artificial intelligence in education: Addressing ethical challenges," *PMC*,
2021. Available: https://pmc.ncbi.nlm.nih.gov/articles/PMC8455229/

[4] C. M. Bishop, *Pattern Recognition and Machine Learning*. Springer, 2006. (GMM, Ch. 9)

[5] F. Pedregosa et al., "Scikit-learn: Machine Learning in Python," *Journal of Machine
Learning Research*, vol. 12, pp. 2825–2830, 2011.
