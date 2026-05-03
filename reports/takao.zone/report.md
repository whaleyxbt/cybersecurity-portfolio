# Vulnerability Report: Sensitive Information Disclosure & Client-Side Admin Panel Bypass

**Target:** Web Platform / Frontend Infrastructure

**Severity:** Medium (P3)

**CVSS v4.0 Score:** 5.5 (Medium)

**CWE:** CWE-201: Insertion of Sensitive Information into Sent-out Data

**Author:** whaleyxbt

**Date:** 03.05.2026

## 1. Executive Summary

During a security audit, it was discovered that administrative logic and UI components (`adminPage`) are included in the production JavaScript build. This allows any authenticated user to force the activation of the management interface. While the backend correctly validates JWTs for API requests, the disclosure of the administrative module leaks the platform's internal structure, logic, and sensitive API endpoints to unauthorized parties.

## 2. Vulnerabilities Description

### 2.1. Administrative Module Leakage (Sensitive Information Disclosure)

The production bundle contains the entire code responsible for user management, balance viewing, and bot configuration. This code should ideally be isolated from the public-facing application.

### 2.2. Client-Side Access Control Bypass

The frontend relies on the local state of the `window.app` object to determine UI visibility. By manipulating this global object, an attacker can bypass client-side checks and render the administration panel without having the "admin" role in their JWT.

### 2.3. Endpoint Disclosure

The vulnerability reveals the following internal administrative endpoints (redacted):

- `/admin/set-fees
    
- `/admin/delete-refferal
    
- `/admin/set-refferal`
    
- `/admin/xxxx-xxxxxxxxxx?user_id=`
    
- `/admin/ban`
    
- `/admin/xxxxxx-settings`
    
- `/admin/stop`
    

## 3. Business Impact

- **Reduced Attack Barrier:** Exposure of the admin logic and endpoint structure significantly lowers the difficulty for performing more complex attacks, such as **BOLA (Broken Object Level Authorization)** or **IDOR**.
    
- **Infrastructure Mapping:** Attackers gain a full "map" of the administrative backend, allowing them to precisely target specific functions.
    
- **Data Intelligence:** If WebSocket channels are not properly secured, an attacker could attempt to subscribe to private administrative feeds discovered via the leaked code.
    

## 4. Steps to Reproduce (Proof of Concept)

**Step 1. Authentication:** Log in to the platform as a regular user.

**Step 2. State Manipulation:** Open the browser console (F12) and execute the following to spoof administrative privileges:

JavaScript

```
window.app.authManager.isAdmin = true;
window.app.authManager.userRole = "admin";
window.app.authManager.permissions = ["*"];
```

**Step 3. Accessing the UI:** Force the administrative page to load:

JavaScript

```
window.app.adminPage.load();
```

_The administration interface will now be visible to the unauthorized user._

## 5. Remediation

1. **Implement Dynamic Imports:** Use code-splitting to ensure that administrative components are only downloaded by users who have been verified as administrators on the server side.
    
2. **Sanitize Production Bundles:** Remove all references, methods, and endpoint strings related to the admin panel from public-facing JavaScript files.
    
3. **WebSocket Audit:** Ensure that private data is not transmitted over public channels and that WebSocket subscriptions strictly validate the user's role server-side.