## 1. Project Title and Overview

- **Paper Title**: Beyond Bug Fixes: An Empirical Investigation of Post-Merge Code Quality Issues in Agent-Generated Pull Requests
- **Authors**: Shamse Tasnim Cynthia, Al Muttakin, Banani Roy
- **Brief description (paper)**: The paper studies post-merge code quality problems introduced by agent-generated pull requests using SonarQube metrics and manual inspection.
- **Brief description (this replication)**: We reproduced the SonarQube-based analysis pipeline: (a) ran the original sequential notebook for an initial subset (123 PRs), (b) implemented a parallel, per-repository worker model to scale processing, and (c) extracted Sonar issues, security hotspots, and merged PR LOC stats to produce the final artifacts used in analysis.

## 2. Repository Structure (replication folder)

```
SoftwareAnalytics-Replication-Post-Merge-QualityIssues/
  README.md                         # This file (replication instructions + overview)
  .env.example                       # example env vars
  notebooks/
    SonarAnalysis.ipynb             # sonar analysis notebook
    result-analysis.ipynb           # downstream analysis + plotting
  datasets/
    All_PR_Sonar_Results.json
    All_PR_Sonar_Results.csv
    All_PR_Issues_Details_with_LOC.csv
    python_fix_prs_with_stats.csv
    python_fix_prs.csv
    Security_Hotspots.json
  artifacts/
   # CSVs and JSONs produced by sonar analysis
   # graphs, tables and other artifacts produced by result-analysis
  logs/
   # logs from the notebook runs and screenshots of the cells and errors encountered
```

## 3. Setup Instructions

### Prerequisites

- OS: Windows
- Python: 3.11.15 (Conda environment)
- Git
- Docker (used to run SonarQube Community LTS)
- SonarScanner CLI

### Environment variables (.env)

- `GITHUB_TOKEN` — GitHub API token (repo access)

### SonarQube setup

1. Run SonarQube (Docker example):

```powershell
docker run -d --name sonarqube -p 9000:9000 sonarqube:lts
```

Wait until SonarQube UI is reachable at `http://localhost:9000`, then create a user token and set `SONAR_TOKEN` in the notebook.

2. Ensure `sonar-scanner` is on `PATH` or set `SONAR_SCANNER_PATH` in `.env` and export it in the notebooks.

## 4. Additional info

- Firstly run the SonarAnalysis notebook, note that you might need to adjust some of the pathes for the data and artifacts output. Ensure your .env is placed in the same folder, where your notebooks are.
- During the Sonar analysis cells, we have executed first 123 PRs using original cell, and later opted out to our own parallelized implementation, which is placed below the original cell, up to 443 PRs. Parallel worker implementation utilizes per Repository grouping, which is why running these cells in different order might produce different results.
- Note, Sonar instance is using elastic search, so expect the RAM consumption to be extremely high, especially with parallel workers.

## 5. Modifications

- Per-repository worker model: PRs are grouped by repository and each worker processes all PRs for its assigned repos (avoids repeated cloning and reduces I/O).
- Parallel extraction: ThreadPoolExecutor is used for both Sonar issue fetches and security-hotspot extraction, which speeds up the pipeline severalfolds. Adjust the number of workers based on your CPU threads and SonarQube worker capacity (community has only one worker, and it's idle time is dependent on your parallel workers in the notebook).
- Small parsing/formatting fixes in extraction cells to prevent errors in the column names.

## 6. Analysis / Scripts

- `notebooks/SonarAnalysis.ipynb` — main pipeline (cloning, checkout, sonar analysis, issues extraction, hotspots, JSON/CSV exports).
- `notebooks/result-analysis.ipynb` — aggregation and plotting steps; produces the CSVs used for further statistical analysis.

## 7. GenAI Usage

- Copilot was used to assist with setting up the SonarQube container and describe the changes made based on the git diff of the original repo and our modifications.