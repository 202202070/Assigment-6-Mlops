# MLOps Assignment 6 — Gatekeeper CI/CD Pipeline

## Overview

This project implements a **Gatekeeper** CI/CD workflow using GitHub Actions that enforces strict conditional execution to prevent wasteful compute resource allocation on broken or unauthorized code.

---

## 🛡️ Gatekeeper Logic

The expensive GPU training job **only runs** when **ALL THREE** gates pass:

| Gate | Check | Condition |
|------|-------|-----------|
| **Gate 1** | Lint passes | `flake8` + `pylint` must succeed |
| **Gate 2** | Branch protection | Push must be on `main` branch |
| **Gate 3** | Manual intent | Commit message must contain `[run-train]` |

---

## 📂 Project Structure

```
.
├── .github/
│   └── workflows/
│       └── pipeline.yaml     ← Main CI/CD workflow (Gatekeeper)
├── train.py                  ← Simulated GPU training script
├── requirements.txt          ← Python dependencies
└── README.md                 ← This file
```

---

## 🚀 How to Trigger Training

To trigger the training job, push to `main` with `[run-train]` in your commit message:

```bash
git add .
git commit -m "feat: update model architecture [run-train]"
git push origin main
```

To run **without** training (lint only):

```bash
git add .
git commit -m "fix: update README"
git push origin main
# OR push to any non-main branch
git push origin feature/my-branch
```

---

## 🔥 Failure Handling

If the training job fails:

1. **`error_logs.txt`** is generated with diagnostic metadata (timestamp, branch, commit SHA, etc.)
2. The file is **uploaded as a GitHub Actions Artifact** — downloadable from the Actions UI for 14 days
3. A **cleanup step** (`always()`) always runs last, printing:
   > `Cleaning up temporary cloud volumes...`

---

## 📸 Pipeline Behavior Summary

| Scenario | Lint | Train | Notes |
|----------|------|-------|-------|
| Push to feature branch | ✅ Runs | ⏭️ Skipped | Branch protection gate fails |
| Push to main, no `[run-train]` | ✅ Runs | ⏭️ Skipped | Intent gate fails |
| Push to main + `[run-train]` | ✅ Runs | ✅ Runs | All gates pass |
| Training fails | ✅ Ran | ❌ Failed | `error_logs.txt` artifact uploaded |
