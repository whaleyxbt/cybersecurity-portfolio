# Session Isolation Failure & Public Exposure of Account Data

**Target:** nado.xyz / Infrastructure & API

**Severity:** Low/Medium (P3)

**Author:** whaleyxbt

**Date:** 03.05.2026

## 1. Executive Summary

A security review of the `nado.xyz` infrastructure revealed two significant flaws in session management and data access control. The first issue involves weak session isolation across subdomains, allowing for potential lateral movement. The second issue involves an information exposure vulnerability within the Archive Indexer API, which allows unauthenticated access to granular portfolio data and private account snapshots for any user.

## 2. Vulnerabilities Description

### 2.1. Weak Session Isolation (Cross-Subdomain Cookie Sharing)

**Component:** `*.nado.xyz`

**Description:** The application utilizes a single session identifier (including `__cf_bm` and potential authentication cookies) across all functional subdomains (`app`, `gateway`, `archive`, `trigger`). There is no strict isolation between read-only services (Archive) and execution-critical services (Gateway).

**Impact:**

- **Lateral Movement:** If a vulnerability (such as XSS) is discovered on a less protected subdomain (e.g., `docs` or `stats`), an attacker can hijack the session to interact with the Gateway or Trigger APIs.
    
- **Increased Attack Surface:** Compromising a session on a public-facing service grants immediate access to the entire trading infrastructure.
    

### 2.2. Information Exposure (Public Access to Account Snapshots)

**Component:** `archive.prod.nado.xyz/v1`

**Description:** The Archive Indexer API allows any unauthenticated user—or any user with a generic session cookie—to query detailed `account_snapshots` for arbitrary `subaccount_id` values. Although subaccount IDs are derived from public wallet addresses, the API returns granular private-sector data typically abstracted in the UI.

**Impact:**

- **Privacy Leak:** Exposure of exact spot/perp balances, unrealized PnL, and liquidation-relevant metrics (maintenance margin, funding).
    
- **Targeted Attacks:** Facilitates "whale watching" and predatory trading strategies (front-running, hunting liquidation prices) by simplifying data collection that would otherwise require complex on-chain indexing.
    

## 3. Business Impact

The combination of these vulnerabilities poses the following risks:

- **Infrastructure Compromise:** Lack of subdomain isolation increases the risk that a minor flaw in a peripheral service leads to a total account takeover.
    
- **Competitive Disadvantage:** The public exposure of detailed portfolio snapshots allows competitors and predatory traders to track user positions and health in real-time, undermining user privacy and platform integrity.
    

## 4. Steps to Reproduce (Proof of Concept)

**Step 1. Identify Target:** Obtain any active `subaccount_id` (e.g., from the public matches endpoint).

**Step 2. Query Archive API:** Send a POST request to `[https://archive.prod.nado.xyz/v1](https://archive.prod.nado.xyz/v1)` with the following payload:

JSON

```
{
  "account_snapshots": {
    "subaccounts": ["TARGET_SUBACCOUNT_ID"],
    "timestamps": [1714694400] 
  }
}
```

**Step 3. Observe Response:** The API returns the full portfolio breakdown, including balances and margins, without requiring specific ownership of the account.

## 5. Remediation

1. **Implement Scoped Sessions:** Use unique, scoped tokens for execution-critical subdomains to prevent cross-service session hijacking.
    
2. **Strict Ownership Checks:** Ensure the backend validates that the `subaccount_id` in the request body matches the authenticated user's wallet identifier stored in the session or JWT.
    
3. **Access Control:** Restrict the Archive Indexer API so that users can only query snapshots for accounts they own.