# Day 83: Revert a Broken ML Release via the Gitea Revert Button

The xFusionCorp Industries ML platform team recently shipped a pull request (PR) titled Add speculative hashing scaffold; this PR was merged by an admin without prior review to facilitate the upcoming Wednesday release cut. Subsequently, this change has caused a lint regression, resulting in a persistent red status on the main branch. According to the team's rollback policy, reverts should not be executed from the command line, and force-pushes are strictly prohibited. Instead, reverts must be processed through a PR, following the same procedure used for the initial breakage. Your task is to utilize Gitea's Revert button on the merged PR to remove the change from the main branch, thereby restoring the CI status to green.


The Gitea UI is on port 3000 (Gitea button). Admin credentials: gitea-admin / gitea2026. The repo is at http://localhost:3000/gitea-admin/fraud-detector and a working clone is at /root/code/fraud-detector (on main). No local git commands are needed — the revert happens entirely in the Gitea UI.

The starting state: the merged PR Add speculative hashing scaffold (visible under Pull Requests → Closed) landed a regression on main, and the latest CI run on main (on the Actions tab) is red. The rollback path is Gitea's built-in revert: reverting the merged commit through a new revert branch and a second, reviewable revert PR — with its own lint + test checks — that merges the reverted state back onto main rather than force-pushing or hand-editing.

The end state must include:

The original Add speculative hashing scaffold PR is still merged (reverts do not re-open the original PR).
A second, separate PR with a title starting with Revert exists and is merged.
main's HEAD commit message contains Revert.
main's HEAD commit SHA reports combined CI status success.
The Revert button is the Gitea (and GitHub) equivalent of the incident-response pager → hotfix → revert-PR → merge playbook. It does two things the command line does not: it creates a human-reviewable PR so the rollback is audit-traceable, and it runs the full CI pipeline against the reverted state before the revert lands on main. That is the difference between fixing production and rolling production back safely.

## Objective

Safely roll back a broken ML release using Gitea's built-in **Revert** workflow.

The rollback must be performed through a reviewable revert Pull Request (PR). No local Git commands or force-pushes are required.

## Scenario

The xFusionCorp Industries ML platform team merged:

**PR #1 — Add speculative hashing scaffold**

The change introduced a CI regression on the `main` branch:

- `CI / lint` failed
- `CI / test` failed
- `main` became red

The rollback policy requires the change to be reverted through a new PR so the rollback is auditable and CI can validate the reverted state before it reaches `main`.

## Repository

- Gitea: `http://localhost:3000`
- Repository: `gitea-admin/fraud-detector`
- Main branch: `main`
- Original PR: `#1`
- Original merged commit: `2e9547bb665e4a604cffddd19132af51b5a6b31a`
- Original source branch: `break-train`

## Steps

### 1. Open the merged PR

In Gitea, go to:

**Pull Requests → Closed**

Open:

**Add speculative hashing scaffold #1**

Confirm that the PR is already merged into `main`.

### 2. Use the Revert button

Open the merged commit or PR and click:

**Revert**

Gitea will show:

**Revert 2e9547bb66 onto:**

Select:

**main**

Do not select `break-train`.

The revert must be based on the current `main` branch.

### 3. Create the revert PR

On the **Commit Changes** page, keep the generated revert commit message.

Select:

**Create a new branch for this commit and start a new pull request**

Gitea will create a revert branch, for example:

`gitea-admin-patch-1`

Click:

**Propose file change**

This creates the second PR.

### 4. Verify the revert PR

The new PR should have a title beginning with:

**Revert**

For example:

`revert 2e9547bb665e4a604cffddd19132af51b5a6b31a`

Confirm:

- Source branch is the generated revert branch
- Target branch is `main`
- `CI / lint` is successful
- `CI / test` is successful
- Gitea says the PR can be merged automatically

### 5. Merge the revert PR

Once both CI checks pass:

1. Click **Merge Pull Request**
2. Confirm the merge
3. Wait for the `main` CI workflow to finish
4. Do not force-push
5. Do not run local Git commands
6. Do not directly edit `main`

## Expected Final State

### Original PR

**PR #1 — Add speculative hashing scaffold**

Must remain:

**Merged**

The original PR should not be reopened.

### Revert PR

A second, separate PR must exist with a title starting with:

**Revert**

Its final state must be:

**Merged**

### Main branch

The latest commit on `main` must have a commit message containing:

**Revert**

The latest `main` CI status must be:

**success**

Required checks:

- `CI / lint` — Success
- `CI / test` — Success

## Incident-Response Flow

Broken PR  
↓  
PR #1 merged  
↓  
`main` becomes RED  
↓  
Open merged PR  
↓  
Click Gitea **Revert**  
↓  
Select `main`  
↓  
Create revert branch  
↓  
Create Revert PR  
↓  
Run CI  
↓  
`CI / lint` passes  
↓  
`CI / test` passes  
↓  
Merge Revert PR  
↓  
`main` contains Revert commit  
↓  
`main` CI becomes GREEN

## Verification Checklist

- [x] Original PR #1 is still merged
- [x] Gitea's **Revert** button was used
- [x] Revert targeted `main`
- [x] Revert was created on a separate branch
- [x] A second PR was created
- [x] Second PR title starts with `Revert`
- [x] `CI / lint` passes
- [x] `CI / test` passes
- [ ] Revert PR is merged
- [ ] `main` HEAD commit message contains `Revert`
- [ ] Latest `main` CI status is `success`

## Key Lesson

The correct rollback pattern is:

**Broken merged PR → Gitea Revert → Revert PR → CI validation → Merge → Green main**

Using the Gitea Revert workflow preserves the original PR, creates an auditable rollback, validates the reverted state with CI, and avoids force-pushing or rewriting `main`.


### Screenshots
