# Vulnerability Report: Insecure Session Management (Long-Lived Static Tokens)

**Target:** `api.tgmrkt.io` / API Infrastructure

**Severity:** Medium (P3)

**CVSS v4.0 Score:** 5.1 (Medium)

**Vulnerability Type:** Insecure Session Management

**Author:** whaleyxbt

**Date:** 27.04.2025

---

## 1. Executive Summary

An analysis of the `tgmrkt.io` API authorization mechanism revealed the use of static UUID tokens with indefinite lifespans. Unlike industry-standard protocols (such as JWT or OAuth2), the current implementation lacks session expiration (TTL) or user context binding (IP/Device). This creates a risk of long-term account compromise should a token be leaked via logs, browser cache, or social engineering.

## 2. Vulnerabilities Description

### 2.1. Static Session Tokens (Lack of Expiration)

**Endpoint:** `[https://api.tgmrkt.io/api/v1/](https://api.tgmrkt.io/api/v1/)*`

**Description:** Requests are authorized solely by passing a static UUID in the `Authorization` header. The primary weaknesses identified are:

- **Lack of TTL:** Tokens do not expire. Once generated, a token remains valid indefinitely, providing a permanent "backdoor" if intercepted.
    
- **No Rotation Mechanism:** The system does not utilize a Refresh Token pattern, meaning there is no way to cycle sessions without a manual re-login.
    
- **Lack of Context Binding:** Tokens are not tied to a browser fingerprint or IP address, making it trivial for an attacker to use a hijacked token across different devices and networks.
    

## 3. Business Impact

This vulnerability is classified as **Medium** because a successful attack requires a pre-existing token leak. However, the impact of such a leak is significant:

- **Persistent Unauthorized Access:** An attacker can monitor user balances and transaction history over an extended period without the user's knowledge.
    
- **Financial Risk:** Access to a valid token allows for operations to be performed on behalf of the user within the API's functional limits.
    
- **Difficulty of Revocation:** Users lack standard security tools to forcibly terminate all active sessions/tokens in the event of a suspected leak.
    

## 4. Steps to Reproduce (Proof of Concept)

**Step 1. Token Acquisition:**

Analyze network traffic (Network Tab) during a legitimate user session to capture the authorization header.

_Example:_ `Authorization: eff60fdb-26b3-4845-a29a-60c54fbc2144`

**Step 2. Emulating a Persistent Session:**

After a significant time delay (e.g., 24+ hours), use the same token in an isolated environment (different IP, different User-Agent):

Bash

```
curl -H "Authorization: eff60fdb-26b3-4845-a29a-60c54fbc2144" \
https://api.tgmrkt.io/api/v1/user/profile
```

**Step 3. Result:**

The server returns the profile data, confirming that the session has not been invalidated and lacks device context enforcement.

## 5. Remediation

1. **Implement JWT (JSON Web Tokens):** Transition to short-lived Access Tokens (15–60 minutes) coupled with secure Refresh Tokens.
    
2. **Context-Aware Validation:** Implement checks for drastic changes in IP address or User-Agent. If detected, require re-validation via Telegram `initData`.
    
3. **Server-Side Revocation:** Establish a mechanism to blacklist/revoke tokens on the backend to allow for "global logout" functionality.
