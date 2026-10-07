# Vulnerability Report: Brute Force Authentication Bypass

### 📊 Vulnerability Overview
* **Vulnerability Type:** Use of a Broken Authentication Mechanism (CWE-307)
* **Severity:** 🔴 High (CVSS v3.1 Score: 8.7)
* **Target Endpoint:** `/vulnerabilities/brute/`
* **Vulnerable Parameters:** `username`, `password` (GET Request)

---

### 1. Description
The web application includes a login interface that fails to implement any rate-limiting, account lockout mechanisms, or CAPTCHA challenges. Because of this missing control, an attacker can programmatically submit an unlimited number of username and password combinations. 

By automating the login requests against a predefined dictionary file (such as `rockyou.txt`), an attacker can eventually discover valid administrative credentials and completely compromise user accounts.

---

### 2. Proof of Concept (PoC)
1. Navigated to the "Brute Force" module inside DVWA.
2. Intercepted a dummy authentication request (`admin`/`password123`) using **Burp Suite Proxy**.
3. Sent the intercepted HTTP request to **Burp Intruder**.
4. Set the positions on the `username` and `password` parameters using a **Cluster Bomb** attack type.
5. Loaded a standard list of common usernames and passwords into the payload tabs.
6. Executed the attack and sorted the results by **Length** and **HTTP Status Code**. 

**Successful Request Payload Identified:**
```http
GET /vulnerabilities/brute/?username=admin&password=password&Login=Login HTTP/1.1
Host: localhost
```

* **Result Analysis:** While failed attempts returned a specific response length indicating an error message ("Username and/or password incorrect."), the valid combination (`admin` / `password`) triggered a distinct HTML response length and displayed a "Welcome to the password protected area admin" message.

**Evidence Screenshot:**
![Brute Force Success](../images/brute-force-proof.png) *(Note: Take a screenshot of your successful Burp Intruder results window showing the differing length/status and save it here)*

---

### 3. Business & Technical Impact
* **Account Takeover:** Unauthorized access to user profiles, potentially including administrative accounts.
* **Data Leakage:** Exposure of confidential business data, user records, or platform metadata accessible to that user tier.
* **Reputational Damage:** Corporate risk stemming from credential stuffing leaks and compromised user data integrity.

---

### 4. Remediation
To mitigate authentication brute-forcing, implement a combination of defensive controls:

#### Account Lockout Policy
Temporarily lock user accounts or IP addresses after a specified number of consecutive failed authentication attempts (e.g., lock account for 15 minutes after 5 failures).

#### Rate Limiting & CAPTCHA
Implement sliding-window rate limiting on the login endpoint using tools like Redis or Nginx configuration, and inject visual verification challenges (like Google reCAPTCHA) upon detecting automated spikes.

**Example Secure Logic Implementation (PHP):**
```php
// Check if login attempts from the tracking database exceed the threshold
\$login_attempts = get_failed_attempts(\(_POST['username'],\)_SERVER['REMOTE_ADDR']);

if (\$login_attempts >= 5) {
    echo "Account temporarily locked due to excessive login failures. Please try again in 15 minutes.";
    exit();
}

// Process authentication securely if under the limit...
```
