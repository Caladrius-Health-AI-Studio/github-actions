# Caladrius Health: DevSecOps & Compliance Architecture

**Target Standards:** ABDM Health Data Management Policy (HDMP), CERT-In WASA, OWASP Top 10 (2025).

## 1. Executive Summary

This repository serves as the **Centralized Governance Hub** for Caladrius Health's engineering ecosystem. It enforces a "Compliance-as-Code" strategy, ensuring that all application code, AI models, and infrastructure configurations are scanned, audited, and validated before deployment to our on-premise Podman environment.

## 2. ABDM-WASA Control Mapping

The following table maps our automated security controls to specific regulatory requirements.

| ABDM / WASA Control | Requirement Description | Automated Control (GitHub Action) | Evidence Artifact |
| --- | --- | --- | --- |
| **Data Privacy (HDMP Ch. 3)** | Prevention of accidental leakage of PII (Aadhaar, PAN, Phone) in logs or code. | **`pii-check`** (Regex-based deep scan) | `violations_report.txt` |
| **Authentication Security** | Prohibition of hardcoded credentials, API keys, or tokens in source code. | **`scan-secrets`** (Gitleaks) | `gitleaks-report.json` |
| **App Security (OWASP Top 10)** | Protection against XSS, Injection, and Insecure Deserialization in Web Apps. | **`web-scan`** (Semgrep + ESLint Security) | `semgrep-report.json` |
| **Secure Logic (Server-Side)** | Validation of Python business logic (OCR) for insecure execution flows. | **`python-scan`** (Bandit SAST) | `bandit-report.json` |
| **AI Supply Chain Security** | Ensuring ML models (Hugging Face) are free from malicious pickle payloads. | **`python-scan`** (Picklescan) | `picklescan-report.json` |
| **Dependency Management** | Detection of known CVEs in 3rd-party libraries (NPM/Pip). | **`web-scan`** (Trivy) & **`python-scan`** (Pip-Audit) | `audit-report.json` |
| **System Hardening** | Prevention of dangerous file types (binaries, executables) in the repository. | **`nc-files`** (Extension Whitelisting) | `non_compliant_report.txt` |
| **Audit Trails (HDMP 26.3)** | Indefinite retention of security assessment logs for forensic analysis. | **Google Drive Archival** (Direct API) | `Caladrius_Security_Audits/` |

## 3. The "Safe-to-Host" Pipeline

Our CI/CD pipeline implements a "Shift-Left" blocking strategy. Vulnerabilities identified at any stage automatically block the promotion of code to the next environment.

### **Phase 1: Feature → Dev (Hygiene Gate)**

* **Trigger:** Pull Request
* **Action:** Scans for "low-hanging fruit" and blatant policy violations.
* **Blocking Controls:**
* Secrets Detected (Gitleaks)
* PII Patterns Found (Aadhaar/PAN)
* Non-Compliant Files (.exe, .dll, large archives)



### **Phase 2: Dev → QA (Deep Inspection)**

* **Trigger:** Weekly Merge / Scheduled
* **Action:** Intensive analysis of logic and dependencies.
* **Blocking Controls:**
* Semantic Code Analysis (Semgrep/Bandit)
* AI Model Safety (Picklescan)
* Container Vulnerability Scan (Trivy for Podman Images)


### **Phase 3: QA → Prod (Compliance Artifacts)**

* **Trigger:** Release Tag
* **Action:** Final verification and archival.
* **Outputs:**
* Software Bill of Materials (SBOM)
* WASA Compliance Report generation
* Push to Immutable Google Drive Archive


## 4. Infrastructure & Data Residency

To comply with **ABDM Data Localization** norms:

* **Runtime:** Applications run on **Podman (Rootless)** containers to prevent privilege escalation attacks.
* **Data Storage:** All Patient Health Information (PHI) resides strictly within on-premise servers in [Bengaluru, India].
* **Audit Storage:** Security logs are archived to a restricted Google Drive folder (Caladrius Corporate Workspace) via a direct Service Account integration, ensuring no data transits through unverified third-party automation tools.

## 5. Incident Response Protocol

Security failures in the pipeline trigger an automated GitHub Issue with the following standardized structure:

* **Title:** `[SEVERITY] Scan Name: Vulnerability Summary`
* **Status:** `🔴 Active`
* **Remediation:** Automated guidance on fixing the specific vulnerability (e.g., "Rotate Key", "Sanitize Input").
* **Resolution:** Issue is closed only after a subsequent scan passes on the same branch.
