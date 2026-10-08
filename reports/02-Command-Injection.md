# Vulnerability Report: OS Command Injection (CWE-78)

### 📊 Vulnerability Overview
* **Vulnerability Type:** Improper Neutralization of Special Elements used in an OS Command (CWE-78)
* **Severity:** 🔴 Critical (CVSS v3.1 Score: 9.8)
* **Target Endpoint:** `/vulnerabilities/exec/`
* **Vulnerable Parameter:** `ip` (POST Parameter)
* **Vector:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H`

---

### 1. Technical Description
The target application endpoint features an utility meant to let internal teams ping server resources. However, it handles inputs unsafely by passing string objects straight down into system execute mechanisms. Attackers can leverage control parameters to run arbitrary system shell utilities under the privilege context of the active daemon operator service.

---

### 2. Multi-Level Exploitation Walkthrough (Proof of Concept)

#### 🔓 Low Security Level
* **Analysis:** The application applies no sanitization checks to input data strings.
* **Payload:** `127.0.0.1; cat /etc/passwd`
* **Execution:** Appending the semicolon operator successfully forces sequential tracking execution of our custom file dump request.

#### ⚠️ Medium Security Level
* **Analysis:** The application filters out explicit input sequences string blocks like `;` and `&&`.
* **Payload:** `127.0.0.1 | whoami`
* **Execution:** By switching token delimiters to a lone vertical pipe indicator, the application bypasses validation routines completely.

#### 🔒 High Security Level
* **Analysis:** A space character constraint flaw exists within the blacklist rule matrix configuration array (`'| '`).
* **Payload:** `127.0.0.1|uname -a`
* **Execution:** Removing the trailing white-space gaps bypasses string match validations, executing commands directly against target processing kernels.

*(Place screenshot images from your lab sessions under your workspace assets folder structure like: `![Evidence](../images/command-injection-proof.png)`)*

---

### 3. Impact Assessment
* **Remote Code Execution (RCE):** Complete control over system utility contexts.
* **Privilege Pivoting:** Potential execution vectors can allow local threat actors to pivot deeper across adjacent secure logical zones.

---

### 4. Code Remediation Matrix
To fix this vulnerability fundamentally, developers must avoid system shell execution wrappers entirely. Use built-in framework validation methods or apply strict white-list regex parameters that accept nothing but properly formatted IP characters.

```php
// Secure Remediation Fix Example
\(target_ip =\)_POST['ip'];

if (filter_var(\$target_ip, FILTER_VALIDATE_IP)) {
    \(output = shell_exec("ping -c 4 " . escapeshellarg(\)target_ip));
    echo "<pre>{\$output}</pre>";
} else {
    echo "Access Denied: Invalid Host Formatting Reference Provided.";
}
```
