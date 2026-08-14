# Day 79: Publish CI Training Artefacts via upload-artifact
The xFusionCorp Industries ML platform team's Continuous Integration (CI) system currently trains the model and generates a confusion matrix in PNG format. However, these artifacts are discarded when the runner tears down the workspace, preventing reviewers on a Pull Request (PR) from accessing them. A teammate has submitted a PR titled Publish Training Artifacts from CI in the fraud-detector repository. Your task is to integrate actions/upload-artifact into the existing report job, ensuring that metrics.json and confusion_matrix.png are available as a downloadable zip file on the run page.


The Gitea UI is running on port 3000 (the Gitea button opens the login page). Admin credentials: gitea-admin / gitea2026. The repo lives at http://localhost:3000/gitea-admin/fraud-detector and a working clone is at /root/code/fraud-detector, already checked out on branch add-artifact-upload. The PR is pre-opened.

The current report job in .gitea/workflows/ci.yml already installs numpy scikit-learn joblib matplotlib, runs python3 -m src.train (writes artifacts/model.joblib and artifacts/metrics.json), runs python3 -m src.plot (writes artifacts/confusion_matrix.png), and ends with ls -la artifacts/ — but the produced files are discarded when the run finishes. The job needs a final step that uploads the artifacts/ directory as a named artefact (model-report) using actions/upload-artifact. Use @v3 — Gitea's runner rejects @v4.

The end state must include:

The report job contains a step that uses actions/upload-artifact@* with a non-empty path and the artefact name model-report.
The run-level artefact download at /gitea-admin/fraud-detector/actions/runs/<id>/artifacts/model-report returns a zip containing both metrics.json and confusion_matrix.png by basename.
The PR head commit's combined status is success.
Artefacts are CI's answer to the question 'what did that run actually produce?'. A green check-mark tells you the code compiled; an uploaded artefact tells you what shipped out the other end. For ML runs, this is where the metrics JSON, the confusion matrix plot, and the pickled model live before anyone promotes them to a registry.

## Objective

Update the `report` job in `.gitea/workflows/ci.yml` so that the training artefacts produced by CI are uploaded and available for download from the Gitea Actions run page.

## Required Change

Add the upload step after the existing `List produced artefacts` step.

```yaml
      - name: Upload training artifacts
        uses: actions/upload-artifact@v3
        with:
          name: model-report
          path: artifacts/
```

> **Important:** Use `actions/upload-artifact@v3`. The Gitea runner rejects `@v4`.

## Final `report` Job

```yaml
  report:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install report deps
        run: pip install --break-system-packages numpy scikit-learn joblib matplotlib
      - name: Train
        run: python3 -m src.train
      - name: Plot confusion matrix
        run: python3 -m src.plot
      - name: List produced artefacts
        run: ls -la artifacts/
      - name: Upload training artifacts
        uses: actions/upload-artifact@v3
        with:
          name: model-report
          path: artifacts/
```

## Commit and Push

```bash
cd /root/code/fraud-detector

git add .gitea/workflows/ci.yml
git commit -m "Publish training artifacts from CI"
git push origin add-artifact-upload
```

## Verification

After the CI run completes, verify:

- The `report` job succeeds.
- The PR head commit's combined status is `success`.
- An artefact named `model-report` appears on the Actions run page.
- Downloading `model-report` produces a ZIP.
- The ZIP contains both `metrics.json` and `confusion_matrix.png` by basename.
- The ZIP also contains `model.joblib`.

## Expected Result

```text
model-report.zip
├── metrics.json
├── confusion_matrix.png
└── model.joblib
```

The CI run now preserves the ML training outputs instead of losing them when the runner workspace is destroyed. Reviewers can download `model-report` directly from the Gitea Actions run page.

### Screenshots
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/a0d5e274-6da5-4652-bf2b-b05a00021209" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/decf6b31-8766-4007-bd99-c1ba0c68c932" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/5cb1c673-f044-4562-8cf2-3b6bd13f46ab" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/f6d6d49b-56fc-42c4-880c-e41dffce3420" />

