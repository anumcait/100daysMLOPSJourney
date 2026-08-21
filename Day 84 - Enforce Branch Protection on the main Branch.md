# Day 84 – Enforce Branch Protection on `main`
The xFusionCorp Industries ML platform team is in the process of promoting the fraud-detector repository to production governance. A recent incident, in which an administrator force-merged a red pull request, resulted in a one-hour disruption of the main branch. This incident highlighted two critical gaps: the Continuous Integration (CI) pipeline only performs linting and does not execute the test suite, and there are no restrictions in place to require contributors to wait for CI checks before merging. Your capstone task is to address both issues. First, enhance the CI pipeline to ensure that it executes the repository's test suite as a test check in addition to the existing lint check. Next, configure Gitea branch protection rules on the main branch to mandate that all future changes must be made through a pull request. This pull request must show successful completion of both the lint and test checks, as well as receive at least one approving review.


The Gitea UI is on port 3000 (Gitea button). Admin credentials: gitea-admin / gitea2026. The repo is at http://localhost:3000/gitea-admin/fraud-detector, with a working clone at /root/code/fraud-detector (on main).

The work has two parts. First, .gitea/workflows/ci.yml defines only a lint job while the test job is left as a # TODO; a test job that installs the test deps and runs the repo's suite needs to be added and run on main at least once, since a status check can only be required once it exists as a context. Second, the main branch needs a branch-protection rule (under the repo's Settings → Branches) that blocks direct pushes so every change arrives through a PR, requires both the lint and test status checks, and requires at least 1 approving review.

The end state must include:

.gitea/workflows/ci.yml on main defines a test job that runs pytest, alongside lint.
GET /api/v1/repos/gitea-admin/fraud-detector/branch_protections returns a rule for main with enable_status_check: true and status_check_contexts including both a lint and a test entry.
required_approvals is at least 1.
Direct push is blocked — either enable_push: false, or enable_push_whitelist: true with every allow-list empty.
Branch protection is the guardrail production governance depends on: required status checks stop the 'reviewer overrides red CI' pattern, required reviews stop the 'admin merges pre-review on a Friday afternoon' incident, and blocking direct pushes means the only path onto main is a reviewed, green pull request—so every change is auditable, revertable, and release-taggable. And you can only require a check that exists, which is why you build the test job before you lock the branch behind it.

## Objective

Add a `test` job to CI and protect the `main` branch so changes require:

- Passing `lint` and `test` checks
- At least 1 approval
- A Pull Request
- No direct pushes

## 1. Update CI

File: `.gitea/workflows/ci.yml`

Added the `test` job:

```yaml
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install test dependencies
        run: pip install --break-system-packages pytest pandas numpy scikit-learn joblib
      - name: Run tests
        run: python3 -m pytest tests -v

Committed and pushed:

git add .gitea/workflows/ci.yml
git commit -m "Add test job to CI pipeline"
git push origin main

Commit:

1826a3f Add test job to CI pipeline

Verified Gitea Actions:

CI / lint (push)  ✅
CI / test (push)  ✅

2. Configure Branch Protection
Gitea:

Settings → Branches → Add Rule

Configuration:

Protected branch: main

Push:
No pushing will be allowed to this branch.

Required approvals:
1

Require status checks:
Enabled

Status check patterns:
*lint*
*test*

3. Verify
curl -s -u 'gitea-admin:gitea2026' \
  http://localhost:3000/api/v1/repos/gitea-admin/fraud-detector/branch_protections | jq .

Final important values:

{
  "branch_name": "main",
  "enable_push": false,
  "enable_status_check": true,
  "status_check_contexts": [
    "*lint*",
    "*test*"
  ],
  "required_approvals": 1
}

Result
CI lint check ✅
CI test check ✅
Test job executed successfully ✅
Branch protection enabled ✅
lint and test required ✅
1 approval required ✅
Direct pushes to main blocked ✅


### Screenshots
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/6dae49eb-a7f8-4b0f-b0aa-bcf8a14494a6" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/14384458-b269-46c1-953f-b07a2e8adb30" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/ae6d68e3-b251-494e-b0bb-d4e8940a3e1e" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/95d9e390-e2e7-49ed-a360-7a57904b8eb1" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/a37b7247-b664-4dbd-acec-c01a5cb69b2f" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/5bdeb202-8586-4e46-9041-a6275b2506ab" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/c0a1f799-053f-4317-a430-d81345ed7de7" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/3c51fb7e-9df4-4fe6-8936-194c554e98e8" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/be9e49f0-4ca7-4963-8c69-9e4d7664b39c" />
