# Vulnerability Report: Sensitive Information Disclosure via Leaked Web3 API Keys

**Target:** petals.farm / Frontend & Static Assets

**Severity:** Medium/High (P2)

**Vulnerability Type:** Information Disclosure

**Author:** whaleyxbt

**Date:** 05.05.2026

## 1. Executive Summary

A security audit of the `petals.farm` client-side assets has identified a critical exposure of infrastructure credentials. Hardcoded API keys for major Web3 providers (Alchemy and Infura) were found within the production JavaScript bundles. These keys facilitate Remote Procedure Calls (RPC) to the blockchain; their exposure allows any third party to utilize the project’s paid resources, leading to potential service disruptions and financial leakage.

## 2. Vulnerabilities Description

### 2.1. Information Disclosure (Hardcoded Alchemy & Infura Keys)

**Component:** Frontend / Static Assets

**Description:** The application’s frontend JavaScript files contain hardcoded API keys used for blockchain interaction. Because these assets are delivered to the user's browser, the keys are essentially public. Unauthorized actors can extract these keys to make arbitrary requests to Ethereum/Polygon nodes on behalf of the project.

**Leaked Credentials (redacted):**

- **Alchemy Key:** `WMxxxxZBgoxxxxxxx
    
- **Infura Key:** `5d2326xxxxx62c91b1xxxxxxxx`
    

## 3. Business Impact

The exposure of these credentials presents several high-level risks:

- **Denial of Service (DoS):** An attacker can exhaust the project’s Compute Units (CU) or daily rate limits by flooding the RPC providers. This would break the connection between the legitimate frontend and the blockchain, making the app non-functional for users.
    
- **Financial Loss:** If the accounts are on "Pay-as-you-go" billing models, unauthorized usage will result in direct, uncontrolled charges to the company.
    
- **Data Scraping & Monitoring:** Exposed keys allow attackers to bypass frontend restrictions to scrape on-chain data or monitor project-specific event logs at scale using the project's own high-speed infrastructure.
    

## 4. Steps to Reproduce (Proof of Concept)

**Step 1. Discovery:** Navigate to the production site and open the browser's **Developer Tools (F12)**.

**Step 2. Asset Inspection:** Go to the **Network** or **Sources** tab and locate the needed JavaScript bundle (censored). Search for the strings "Alchemy" or "Infura".

**Step 3. Verification (POC):** Verify the validity of the leaked key using a simple terminal command:

Bash

```
curl https://eth-mainnet.g.alchemy.com/v2/xxxxxxxxxxxxxx \
-X POST \
-H "Content-Type: application/json" \
-d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'
```

_The response returns the current block height, confirming the key is active and unauthorized access is possible._

## 5. Remediation

1. **Rotate the Keys:** Immediately revoke/delete the leaked keys in the Alchemy and Infura dashboards and generate new ones.
    
2. **Backend Proxy (Recommended):** Implement a backend relay. The frontend should call your own server API, which then attaches the secret key and forwards the request to the provider. This keeps the key entirely hidden from the client.
    
3. **Strict Allowlisting:** If a proxy is not feasible, configure the provider's security settings to restrict requests only to specific **HTTP Referrers** (your domain) and specific **Smart Contract addresses**.