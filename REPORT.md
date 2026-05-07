# MLOps Lab Report: Wine Quality Prediction System
**Student:** 2A202600429-Tran Ngoc Hung

---

## 1. Project Overview
This project demonstrates a complete MLOps lifecycle, from local experimentation to automated CI/CD and continuous training. We built a Wine Quality classification system using a RandomForest model, managed with DVC for data versioning and GitHub Actions for automation.

---

## 2. Step 1: Local Experimentation & MLflow Tracking
During the initial phase, multiple experiments were conducted to tune the `RandomForestClassifier`.

### Best Hyperparameters:
- **n_estimators:** 100
- **max_depth:** 5
- **min_samples_split:** 2

**Reasoning:** This configuration achieved a stable **Accuracy of ~0.7480** and an **F1-score of ~0.7460**. Increasing `max_depth` beyond 5 led to slight overfitting on the training set without significant gains on the evaluation set, while `n_estimators=100` provided a good balance between training speed and model robustness.

---

## 3. Step 2 & 3: Automation and Continuous Training
### CI/CD Pipeline:
We implemented a 4-stage pipeline in GitHub Actions:
1. **Unit Test:** Verified the training logic using synthetic data.
2. **Train:** Automatically pulled data via DVC, trained the model, and logged results to MLflow.
3. **Eval:** Acted as a quality gate, ensuring only models with `accuracy >= 0.70` proceed.
4. **Deploy:** Automatically restarted the FastAPI serving instance on a Google Compute Engine VM.

### Continuous Training:
By adding 5,996 new samples (Step 3) and updating the DVC pointer, the pipeline was triggered automatically. The model was retrained on a total of **8,994 samples**, successfully maintaining the performance threshold and deploying the updated model to production without manual intervention.

---

## 4. Challenges & Resolutions

| Challenge | Resolution |
| :--- | :--- |
| **ModuleNotFoundError: 'pkg_resources'** | Found that `mlflow` depends on `pkg_resources` which was removed in `setuptools >= 70`. Fixed by pinning `setuptools<70` in `requirements.txt`. |
| **MLflow Database Version Mismatch** | Encountered `ResolutionError` when switching between MLflow 2.x (project) and 3.x (global). Resolved by forcing the use of the `ai_infra` environment for both the UI and training. |
| **Invalid SQLite URI for Artifacts** | MLflow failed to log artifacts when using relative SQLite paths. Resolved by using **absolute paths** for the `tracking_uri` and defining an explicit experiment name. |
| **Deployment Unit Not Found** | The first deployment failed because the `systemd` service was not yet created on the VM. Manually configured the `.service` file on GCE to resolve. |

---

## 5. Final Results
- **GitHub Repository:** [hungnt0406/Day21-Track2-CI-CD-for-AI-Systems](https://github.com/hungnt0406/Day21-Track2-CI-CD-for-AI-Systems)
- **Deployment Status:** Active
- **Endpoints:**
  - `GET /health`: `{"status": "ok"}`
  - `POST /predict`: Successfully returns wine quality labels ("thap", "trung_binh", "cao").
- **Infrastructure:** GCS Bucket for data/models and GCE VM for inference.
