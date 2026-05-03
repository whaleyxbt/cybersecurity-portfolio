# Vulnerability Report: GCS Bucket Compromise via Broken Access Control & Unrestricted File Upload

**Target:** getmoni.io / API & Storage Infrastructure
**Severity:** High (P2)

**CVSS (Common Vulnerability Scoring System Version 4.0):** 7.9 (High)

**Author:** whaleyxbt

**Date:** 10.04.2026

## 1. Executive Summary

During a security assessment of the project's infrastructure, a chain of critical vulnerabilities was identified, related to incorrect access control settings for cloud storage (Google Cloud Storage) and a lack of validation when uploading files via API. These vulnerabilities allow an unauthorized attacker to view the project's confidential files (including private wallet lists and employee PII), as well as use the company's infrastructure as free hosting for arbitrary (including malicious) content.

## 2. Vulnerabilities Description

### 2.1. Broken Access Control (Unauthenticated Bucket Listing)

The base vulnerability lies in the open access to the directory listing of a project-owned GCS bucket (`<REDACTED_BUCKET>`).

**URL:** `<REDACTED_GCS_BUCKET_URL>`

**Description:** Any unauthorized user can obtain a complete XML list of all objects in the storage. Access permissions are incorrectly configured: it is likely that the `storage.objects.list` permission was granted to the `allUsers` group. This completely compromises the company's file structure.

### 2.2. Unrestricted File Upload (Bypassing Intended Use)

**Endpoint:** `POST <REDACTED_UPLOAD_ENDPOINT>`

**Description:** The endpoint allows uploading images to the bucket without any authorization (the API is publicly available via Swagger). Although the server limits file types to images and adds a random suffix (preventing direct overwriting), the lack of rate limits and permission checks allows an attacker to automatically upload an unlimited number of large media files. This leads to resource exhaustion (Storage DoS) and financial losses for the company due to cloud storage billing.

### 2.3. Sensitive Data Exposure & Improper Assets Management

Due to the open listing (section 2.1), a leak of critical data was discovered:

- **Business Logic Data:** In a private project-data directory, there is a whitelist file containing EVM addresses. The disclosure of this data compromises the privacy of project users and creates vectors for targeted phishing.
    
- **PII (Personally Identifiable Information):** In a non-production personal-data directory, employee-related personal files were exposed. The exact paths, names, dates, and locations are intentionally redacted in this public version.
    

Using corporate storage for personal purposes (Improper Assets Management) leads to serious reputational risks and risks of disclosing employee privacy.

## 3. Business Impact

The combination of these vulnerabilities poses the following risks to the project:

- **Phishing & XSS:** It is likely possible to upload arbitrary files (.html), allowing for Stored XSS in the context of the trusted storage domain and hosting phishing pages using moni team resources.
    
- **Resource Exhaustion (Storage DoS):** The lack of authorization and limits on the upload endpoint allows an attacker to automatically fill the bucket with garbage data. This will lead to significant financial losses due to Google Cloud bills (storage and traffic) and denial of service (DoS).
    
- **Reputational & Privacy Damage:** Public leakage of PII and private whitelists.
    

## 4. Steps to Reproduce (Proof of Concept)

**Step 1. Exploiting Unauthenticated Listing:** Open the redacted GCS bucket URL. The response returns an XML listing with the structure of the storage.

**Step 2. Accessing Confidential Assets:** Using the keys from the listing, internal files can be accessed directly:

- Wallet lists: `<REDACTED_WALLET_LIST_OBJECT>`
    
- Personal files: `<REDACTED_PERSONAL_DATA_OBJECTS>`
    
- `<REDACTED_PERSONAL_DATA_INDEX_OBJECT>`
    

**Step 3. Exploiting Unauthenticated Image Upload:** Use the redacted public API documentation and the affected upload endpoint.

## 5. Remediation

1. **GCS IAM Policy Configuration:** Remove `storage.objects.list` permissions for `allUsers` at the level of the affected bucket. Access should be granular.
    
2. **Protecting Upload Endpoints:** Implement mandatory authorization token verification for the affected upload endpoint.
    
3. **Storage Audit:** Conduct a bucket review, delete personal employee data, and move business logic files to private storage.
    

**Contact with me:** 

t.me/whaleyxbt
[x.com/whaleyxbt](https://x.com/whaleyxbt)
