# Vulnerability Report: Presale Ecosystem Compromise via IDOR and Sensitive Data Exposure

**Target:** `hololaunch.ai` & `loud.hololaunch.ai` / Presale Infrastructure

**Severity:** High (P2)

**Vulnerability Types:** Insecure Direct Object Reference (IDOR), Sensitive Information Disclosure

**Author:** whaleyxbt

**Date:** 30.05.2025

## 1. Executive Summary

A security review of the Hololaunch AI presale platform revealed a critical vulnerability chain. Due to a lack of server-side authorization checks, an attacker can impersonate any project participant. By combining this with leaked whitelist data (CSV export), an attacker can gain full access to the private dashboards of high-value investors, view their allocations, and interact with authenticated UI elements.

## 2. Vulnerabilities Description

### 2.1. Improper Authorization (Global Identity Spoofing)

**Component:** `loud.hololaunch.ai` Backend / Privy Integration

**Description:** After a user connects their wallet via Privy, the application sends a POST request to the backend to retrieve account-specific data (allocations, contributions, status). The backend relies solely on the wallet address provided in the request body without verifying the session signature or ownership.

- **Vector:** By intercepting the POST request and replacing the `walletAddress` parameter with a target address, the server returns a `200 OK` response with the victim's private data.
    
- **Result:** The frontend renders the dashboard as if the attacker were the legitimate owner of the wallet.

### 2.2. Sensitive Information Disclosure (Whitelist Leak)

**Component:** Wallet Whitelist Checker

**Description:** The "Whitelist Checker" utility on the website leaks the entire database of eligible participants. By analyzing the network request when checking a single wallet, a static URL to a complete `.csv` table was discovered.

- **Leaked Data:** Full list of wallet addresses eligible for the presale, enabling targeted impersonation via the vulnerability described in section 2.1.

### 2.3. Information Exposure (Unclaimed Fees & Internal Metrics)

**Component:** Gitlist / Public Assets

**Description:** A publicly accessible configuration/log file on the developer's infrastructure reveals sensitive internal metrics, including:

- `unclaimedFees`: Real-time data on pair addresses, position addresses, and accrued fees.
    
- `tokenAMint` / `tokenBMint`: Internal contract addresses before official public disclosure.

## 3. Business Impact

- **Privileged Information Theft:** Attackers can monitor the exact SOL amounts contributed by every participant and track total allocation distribution.
    
- **UI/Logic Manipulation:** While the Smart Contract prevents unauthorized claiming, the ability to interact with the UI as another user allows for "Intent Spoofing." If the platform introduces off-chain features (e.g., social tasks, governance, or "burn" mechanisms), the attacker would have full control over these actions.
    
- **Strategic Front-Running:** Exposure of internal DEX metrics and early contract addresses allows for unfair market advantages (sniping) prior to official TGE.
    

## 4. Steps to Reproduce (Proof of Concept)

**Step 1. Data Collection:** Access the leaked whitelist via the static CSV link discovered in the frontend requests. Select a high-value target (whale address).

**Step 2. Request Interception:** Navigate to `[https://loud.hololaunch.ai/](https://loud.hololaunch.ai/)`, connect a burner wallet, and intercept the identity verification POST request using a proxy tool (e.g., Burp Suite).

**Step 3. Identity Spoofing:** Modify the request body by replacing your burner wallet address with the target whale address:

JSON

```
{
  "wallet": "TARGET_WHALE_ADDRESS_FROM_CSV"
}
```

**Step 4. Execution:** Forward the request. The web interface will now display the target's allocation, investment amount, and the "Claim Your Tokens" dashboard.

## 5. Remediation

1. **Enforce Signature Verification:** The backend must verify the cryptographic signature provided by the wallet (e.g., via Privy) for _every_ sensitive data request. Never trust a wallet address passed in a plaintext body.
    
2. **Secure Static Assets:** Remove public access to the full whitelist CSV. The checker should only return a boolean (True/False) for a specific address via a secure API.
    
3. **Infrastructure Hardening:** Remove sensitive project logs and "Gitlist" files from production-facing servers.
    
4. **Integrity Checks:** Ensure that any action initiated via the GUI (Claim, Contribute, Burn) is validated against the authenticated session on the server side.