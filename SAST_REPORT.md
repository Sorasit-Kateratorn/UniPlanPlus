# Static Application Security Testing (SAST) Report — UniPlanPlus

## 1. Scan Overview

- **Tool:** Semgrep CLI `v1.186.0`
- **Configuration:** `--config=auto` (Semgrep Community Registry — 1,074 code rules loaded, 508 applicable rules executed)
- **Scan Date:** November 22, 2026
- **Scope:** Full repository tracked by Git (`105` files on initial scan; `106` files on post-remediation verification scan; ~99.9% parsed lines):
  - **Python (`backend/` & `scripts/`):** 39 files (243 rules)
  - **JavaScript / React (`frontend/`):** 23 files (153 rules)
  - **TypeScript:** 1 file (163 rules)
  - **YAML (`.github/workflows/`, `docker-compose.yml`):** 2 files (35 rules)
  - **Dockerfiles (`backend/Dockerfile`, `frontend/Dockerfile`):** 2 files (6 rules)
  - **HTML / JSON / Multilang:** 8+ files (65 rules)
- **Machine-Readable Post-Remediation Output:** [Semgrep_SAST_Report.json](Semgrep_SAST_Report.json)

---

## 2. Executive Summary (Before vs. After Remediation)

| Scan Phase | Files Scanned | Rules Run | Error (High) | Warning (Medium) | Info (Low) | Total Findings |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **Initial Scan (Pre-Remediation)** | 105 | 508 | 6 | 11 | 0 | **17** |
| **Verification Re-Scan (Post-Remediation)** | 106 | 508 | **0** | **0** | **0** | **0** |

### Key Application Code Result

- **0 findings** across all **39 Python backend files** (Django views, serializers, authentication, authorization, tokens, middleware, CSV import/export) and all **23 JavaScript/React frontend files**.
- No SQL injection, XSS, command injection, unsafe deserialization, path traversal, weak cryptography, or broken authentication patterns were detected in application source code.
- All **17 initial findings** were in infrastructure/CI configuration and a previously committed DAST HTML report, and all 17 were remediated and re-verified to **0 findings**.

---

## 3. Detailed Findings & Remediations

| # | Rule ID | Severity | CWE / OWASP | Target File & Lines | Instances | Finding Description | Remediation Applied & Verification |
| :-: | :--- | :---: | :--- | :--- | :---: | :--- | :--- |
| 1 | `dockerfile.security.missing-user.missing-user` | **ERROR** (High) | CWE-250 / A04:2021 | `backend/Dockerfile` (L14) | 1 | Container lacked a `USER` instruction, causing the Django backend process to run as `root` inside the container. | **Fixed & Verified:** Created dedicated non-root system user/group (`appuser:appgroup`), set ownership on `/app` and `/app/logs`, and added `USER appuser` before `CMD`. |
| 2 | `generic.secrets.security.detected-jwt-token.detected-jwt-token` | **ERROR** (High) | CWE-321 / A02:2021 | `Zap_Report_Default_Policy.html` (L744, L885, L1049, L1227, L1367) | 5 | A test session JWT captured during the authenticated OWASP ZAP scan was embedded in request headers inside the HTML report. | **Fixed & Verified:** Redacted all 5 occurrences to `Authorization: <REDACTED_JWT_TOKEN>` in `Zap_Report_Default_Policy.html`. |
| 3 | `yaml.github-actions.security.github-actions-mutable-action-tag` | **WARNING** (Medium) | CWE-1357, CWE-353 / A08:2021 | `.github/workflows/fullstack-ci-cd.yml` (L16, L19, L34, L78, L81, L120, L123, L156, L159, L165, L173) | 11 | GitHub Actions workflow referenced third-party actions using mutable major version tags (`@v4`, `@v5`, `@v3`, `@v6`), creating supply-chain tampering risk. | **Fixed & Verified:** Pinned all 11 references to verified 40-character commit SHAs (`actions/checkout`, `actions/setup-python`, `actions/setup-node`, `docker/login-action`, `docker/build-push-action`) with inline version comments. |

---

## 4. Verification Command & Result

```powershell
semgrep scan --config=auto --json-output=Semgrep_SAST_Report.json
```

**Post-Remediation Result:**

```text
Scan Summary:
  Scan completed successfully.
  - Findings: 0 (0 blocking)
  - Rules run: 508
  - Targets scanned: 106
  - Parsed lines: ~99.9%
Ran 508 rules on 106 files: 0 findings.
```
