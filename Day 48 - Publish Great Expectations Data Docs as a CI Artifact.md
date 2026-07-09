# Day 48: Publish Great Expectations Data Docs as a CI Artifact
The xFusionCorp Industries ML platform team landed the drift_check Great Expectations checkpoint in CI on the fraud-detector repo—a data-quality workflow runs it on every PR. It stops a red-data PR from merging, but when the run is green reviewers still can't see the Data Docs output that GE generated in-job. A teammate has opened a PR titled Publish Data Docs as a CI artefact. Your task is to wire actions/upload-artifact into the workflow so every run leaves its Data Docs tree on the run page as a downloadable zip.


The Gitea UI is on port 3000 (Gitea button). Admin credentials: gitea-admin / gitea2026. The repo is at http://localhost:3000/gitea-admin/fraud-detector and a working clone is at /root/code/fraud-detector, already checked out on branch add-data-docs-artefact. The PR is pre-opened.

The current data-quality job in .gitea/workflows/data-quality.yml already:

Installs great_expectations, pandas, numpy.
Runs python3 -m src.gx_run – Which bootstraps the project in-workspace and executes the drift_check checkpoint. The run leaves the Data Docs site at gx/uncommitted/data_docs/local_site/ relative to the repo root.
Extend /root/code/fraud-detector/.gitea/workflows/data-quality.yml with an additional final step on the data-quality job that uploads the Data Docs tree as a workflow artefact, then commit and push to the add-data-docs-artefact branch to trigger a fresh run.

The end state must include:

The data-quality job in the workflow contains a step that uses actions/upload-artifact@* with a path referencing gx/uncommitted/data_docs/ (or any descendant of it).
The artefact name is data-docs.
The PR head commit's combined status reaches success.
GET /api/v1/repos/gitea-admin/fraud-detector/actions/artifacts returns at least one artefact.
The downloaded zip contains a Data Docs index.html AND at least one validations/*.html page.
A data-quality check inside CI is only actionable if the evidence survives the run—index.html + validations/*.html are what the reviewer opens in their browser to decide whether the drift is a real problem or a legitimate change that should update the expectation. Without the artefact the run becomes a binary pass/fail with no trail.
## Objective

The `fraud-detector` repository already had a **Great Expectations** data-quality workflow that executed on every Pull Request. The workflow generated **Data Docs**, but they were lost after the workflow completed because they were not uploaded as an artifact.

The objective was to modify the CI workflow to upload the generated **Data Docs** as a downloadable workflow artifact.

---

## Repository Details

- **Repository:** `fraud-detector`
- **Branch:** `add-data-docs-artefact`
- **Workflow:** `.gitea/workflows/data-quality.yml`

---

## Existing Workflow

The workflow already performed the following steps:

1. Checkout the repository.
2. Install dependencies:
   - `great_expectations`
   - `pandas`
   - `numpy`
3. Run the Great Expectations checkpoint:

```bash
python3 -m src.gx_run
```

The above command generates the Data Docs under:

```text
gx/uncommitted/data_docs/local_site/
```

Since the workflow workspace is deleted after completion, these files were not available to reviewers.

---

# Solution

Added a final workflow step to upload the generated Data Docs as a workflow artifact.

```yaml
- name: Upload Data Docs
  uses: actions/upload-artifact@v3
  if: always()
  with:
    name: data-docs
    path: gx/uncommitted/data_docs/
```

---

## Explanation

### `actions/upload-artifact@v3`

Uploads files generated during the workflow so they can be downloaded after the job completes.

### `if: always()`

Ensures the artifact is uploaded even if a previous step fails, allowing reviewers to inspect the generated reports.

### `name: data-docs`

The uploaded artifact appears with the name:

```text
data-docs
```

### `path`

```text
gx/uncommitted/data_docs/
```

Uploads the complete Great Expectations Data Docs directory.

---

# Git Configuration

Initially, Git commits failed because the author identity was not configured.

Configured Git using:

```bash
git config --global user.name "gitea-admin"
git config --global user.email "gitea-admin@example.com"
```

---

# Git Commands

Stage changes:

```bash
git add .gitea/workflows/data-quality.yml
```

Commit:

```bash
git commit -m "Publish Great Expectations Data Docs as CI artifact"
```

Push:

```bash
git push origin add-data-docs-artefact
```

---

# Workflow Verification

Verified the Pull Request:

```
Publish Data Docs as a CI artefact
```

Status:

```
All checks were successful

Data Quality / data-quality (pull_request)
Success
```

---

# Actions Verification

Verified the workflow run:

```
Publish Great Expectations Data Docs as CI artifact
Commit: 8419fef
Status: Success
```

Workflow steps:

```
✓ Set up job
✓ actions/checkout@v4
✓ Install Great Expectations
✓ Run drift_check checkpoint
✓ Upload Data Docs
✓ Complete job
```

---

# Artifact Verification

Confirmed the workflow generated the artifact:

```
Artifacts
└── data-docs
```

The downloaded artifact contains the Great Expectations Data Docs, including:

```
local_site/
├── index.html
└── validations/
    ├── *.html
```

---

# Final Workflow

```yaml
name: Data Quality

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  data-quality:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Install Great Expectations
        run: |
          pip install --break-system-packages \
            great_expectations pandas numpy

      - name: Run drift_check checkpoint
        run: python3 -m src.gx_run

      - name: Upload Data Docs
        uses: actions/upload-artifact@v3
        if: always()
        with:
          name: data-docs
          path: gx/uncommitted/data_docs/
```

---

# Outcome

- Added an artifact upload step to the CI workflow.
- Used `actions/upload-artifact@v3`.
- Uploaded the `gx/uncommitted/data_docs/` directory.
- Named the artifact **data-docs**.
- Configured Git author information.
- Committed and pushed changes to the `add-data-docs-artefact` branch.
- Verified the Pull Request passed all checks.
- Verified the workflow successfully uploaded the **data-docs** artifact.
- Ensured reviewers can download and inspect the generated Great Expectations Data Docs after every CI run.

### Screenshots
<img width="1012" height="474" alt="image" src="https://github.com/user-attachments/assets/dbec8d11-a074-42e9-9c9e-f3765de8ed2f" />
<img width="1010" height="436" alt="image" src="https://github.com/user-attachments/assets/f4acc167-0140-4825-9b88-562bc99ffd19" />
<img width="1026" height="452" alt="image" src="https://github.com/user-attachments/assets/6525fc6e-40ae-4393-9f18-3f1cd34f8106" />
<img width="1039" height="422" alt="image" src="https://github.com/user-attachments/assets/64f2421a-64c1-4fb0-8003-f1ca2b0d8f4e" />
<img width="1026" height="493" alt="image" src="https://github.com/user-attachments/assets/e407cee6-7341-4e3f-96ca-fa7af902916f" />
<img width="1026" height="471" alt="image" src="https://github.com/user-attachments/assets/ac9f06f9-fcd5-4068-bea0-48e35513bfec" />
<img width="1024" height="479" alt="image" src="https://github.com/user-attachments/assets/e8bd9fea-3ca8-45fb-8cae-ad637f8473e0" />
<img width="1000" height="452" alt="image" src="https://github.com/user-attachments/assets/e6c29267-7c35-40e2-931f-f440281c1809" />








