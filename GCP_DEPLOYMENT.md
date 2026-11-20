# Deployment to Google Cloud Platform (GCP) Complete

The **CImEx Compliance** platform has been packaged into a production container and deployed to **Google Cloud Run** in the **Mumbai region (`asia-south1`)**.

---

## Live GCP Endpoints

| Resource | Live URL |
| :--- | :--- |
| **GCP Cloud Run Service** | [https://cimex-compliance-1023003229611.asia-south1.run.app](https://cimex-compliance-1023003229611.asia-south1.run.app) |
| **Interactive App on Cloud Run** | [https://cimex-compliance-1023003229611.asia-south1.run.app/app.html](https://cimex-compliance-1023003229611.asia-south1.run.app/app.html) |
| **Documentation on Cloud Run** | [https://cimex-compliance-1023003229611.asia-south1.run.app/docs.html](https://cimex-compliance-1023003229611.asia-south1.run.app/docs.html) |
| **Simulation on Cloud Run** | [https://cimex-compliance-1023003229611.asia-south1.run.app/simulation.html](https://cimex-compliance-1023003229611.asia-south1.run.app/simulation.html) |
| **Vercel Mirror** | [https://cimex-compliance.vercel.app](https://cimex-compliance.vercel.app) |
| **GitHub Repository** | [https://github.com/HemanthDadi1/CImEx-Compliance](https://github.com/HemanthDadi1/CImEx-Compliance) |

---

## GCP Infrastructure Configured

1. **Google Cloud Project:** `project-8fc4b3e5-5416-4eae-8e8` (Project Number: `1023003229611`).
2. **Container Registry:** `cloud-run-source-deploy` in Google Artifact Registry (`asia-south1`).
3. **Cloud Run Service:** `cimex-compliance` (100% traffic routed to revision `cimex-compliance-00001-kfv`).
4. **Vertex AI Integration:** Initialized with `VERTEX_PROJECT=project-8fc4b3e5-5416-4eae-8e8` in `asia-south1`, keeping all Indian industrial compliance data within India.
5. **Verified Health:** Tested `/api/state` and `/app.html` on Cloud Run—returning `HTTP 200 OK` with active Vertex mode enabled.
