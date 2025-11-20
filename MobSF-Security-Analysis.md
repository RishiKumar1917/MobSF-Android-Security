# Android Application Security Analysis with MobSF

## Overview
This document provides a comprehensive guide for performing Android application security analysis using Mobile Security Framework (MobSF). This aligns with Flipkart's security requirements for mobile application pentesting.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [APK Download and Preparation](#apk-download-and-preparation)
3. [MobSF Installation](#mobsf-installation)
4. [Scanning Process](#scanning-process)
5. [Security Findings Analysis](#security-findings-analysis)
6. [Reporting Template](#reporting-template)

## Prerequisites

### Required Tools
- Docker (recommended) or Python 3.8+
- Git
- Web browser
- At least 4GB RAM
- 10GB free disk space

### Knowledge Requirements
- Basic understanding of Android security
- Familiarity with OWASP Mobile Top 10
- Basic command line usage

## APK Download and Preparation

### Option 1: Using a Vulnerable APK for Testing

For learning purposes, we recommend using intentionally vulnerable applications:

1. **DIVA (Damn Insecure and Vulnerable App)**
   ```bash
   wget https://github.com/payatu/diva-android/raw/master/diva-beta.apk
   ```

2. **InsecureBankv2**
   ```bash
   git clone https://github.com/dineshshetty/Android-InsecureBankv2.git
   cd Android-InsecureBankv2/AndroLabServer
   # Follow build instructions in the repository
   ```

3. **OWASP MSTG UnCrackable Apps**
   ```bash
   # Download from OWASP MSTG repository
   wget https://github.com/OWASP/owasp-mstg/raw/master/Crackmes/Android/Level_01/UnCrackable-Level1.apk
   ```

### Option 2: Extract APK from Android Device

```bash
# List installed packages
adb shell pm list packages

# Find the APK path
adb shell pm path com.example.app

# Pull the APK
adb pull /data/app/com.example.app/base.apk myapp.apk
```

### APK Verification

Before scanning, verify the APK:

```bash
# Check APK signature
apksigner verify --verbose myapp.apk

# Get basic APK info
aapt dump badging myapp.apk | head -20
```

## MobSF Installation

### Method 1: Docker (Recommended)

```bash
# Pull the latest MobSF Docker image
docker pull opensecurity/mobile-security-framework-mobsf:latest

# Run MobSF container
docker run -it --rm -p 8000:8000 opensecurity/mobile-security-framework-mobsf:latest
```

Access MobSF at: `http://localhost:8000`

### Method 2: Manual Installation

```bash
# Clone MobSF repository
git clone https://github.com/MobSF/Mobile-Security-Framework-MobSF.git
cd Mobile-Security-Framework-MobSF

# Install dependencies (Ubuntu/Debian)
sudo apt-get update
sudo apt-get install -y python3 python3-pip openjdk-11-jdk

# Install Python dependencies
pip3 install -r requirements.txt

# Run setup
./setup.sh

# Start MobSF
./run.sh
```

Access MobSF at: `http://127.0.0.1:8000`

### Verify Installation

1. Open browser to `http://localhost:8000`
2. You should see the MobSF dashboard
3. Check that the upload interface is available

## Scanning Process

### Step 1: Upload APK to MobSF

1. Navigate to `http://localhost:8000`
2. Click on "Upload & Analyze"
3. Select your APK file
4. Click "Analyze"

### Step 2: Wait for Analysis

The analysis process includes:
- Static analysis of the APK
- Decompilation of code
- Manifest analysis
- Permission analysis
- Code security analysis
- Binary analysis

Analysis typically takes 2-5 minutes depending on APK size.

### Step 3: Review Initial Report

Once complete, you'll see:
- Security Score
- Application Info
- Security Analysis Summary

## Security Findings Analysis

### 1. Insecure Permissions Analysis

#### What to Look For:

**High-Risk Permissions:**
- `android.permission.READ_SMS` - Can read SMS messages
- `android.permission.SEND_SMS` - Can send SMS messages
- `android.permission.READ_CONTACTS` - Access to contacts
- `android.permission.ACCESS_FINE_LOCATION` - Precise location
- `android.permission.CAMERA` - Camera access
- `android.permission.RECORD_AUDIO` - Microphone access
- `android.permission.READ_CALL_LOG` - Call history
- `android.permission.WRITE_EXTERNAL_STORAGE` - File system access

#### Analysis Steps:

1. Navigate to "Permissions" section in MobSF report
2. Review each permission and assess:
   - Is it necessary for app functionality?
   - Is it documented in privacy policy?
   - Can it be misused for data exfiltration?

#### Example Findings:

```
CRITICAL: Unnecessary Dangerous Permissions
- READ_SMS: App requests SMS read permission without clear justification
- ACCESS_FINE_LOCATION: Location permission not aligned with app purpose
- CAMERA: No camera functionality evident in the app

RECOMMENDATION: Remove unnecessary permissions or provide clear justification
```

### 2. Weak Cryptography Analysis

#### What to Look For:

**Insecure Cryptographic Practices:**
- Use of ECB mode in AES encryption
- Hardcoded encryption keys
- Use of weak algorithms (DES, MD5, SHA1 for passwords)
- Insecure random number generation
- Static initialization vectors (IVs)

#### MobSF Detection Areas:

1. **Code Analysis Section** - Review:
   - "Cryptography" findings
   - "Code Analysis" for crypto patterns

2. **Common Vulnerabilities:**
   ```java
   // INSECURE: ECB mode
   Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
   
   // INSECURE: Hardcoded key
   String key = "ThisIsMySecretKey123";
   
   // INSECURE: MD5 for passwords
   MessageDigest.getInstance("MD5");
   ```

3. **Secure Alternatives:**
   ```java
   // SECURE: CBC/GCM mode with random IV
   Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
   
   // SECURE: Key from Android Keystore
   KeyStore keyStore = KeyStore.getInstance("AndroidKeyStore");
   
   // SECURE: SHA-256 or better
   MessageDigest.getInstance("SHA-256");
   ```

#### Example Findings:

```
HIGH: Weak Cryptographic Implementation
- Location: com.example.crypto.CryptoUtils.java
- Issue: AES encryption using ECB mode detected
- Risk: ECB mode is not semantically secure and can leak patterns
- CVE Reference: CWE-327

MEDIUM: Insecure Hash Algorithm
- Location: com.example.auth.LoginActivity.java
- Issue: MD5 hash used for password storage
- Risk: MD5 is cryptographically broken and vulnerable to collisions
- CVE Reference: CWE-328

RECOMMENDATION: 
1. Use AES/GCM or AES/CBC mode with random IV
2. Implement PBKDF2, bcrypt, or Argon2 for password hashing
3. Store keys in Android Keystore
```

### 3. Secrets and Hardcoded Data Analysis

#### What to Look For:

**Sensitive Data Exposure:**
- API keys and tokens
- Passwords and credentials
- Private keys and certificates
- OAuth secrets
- Database connection strings
- AWS/Cloud credentials
- Encryption keys
- URLs to internal systems

#### MobSF Detection:

1. **Strings Section**: Review for:
   - API_KEY patterns
   - Password patterns
   - URLs with credentials
   - Base64 encoded secrets

2. **Code Analysis**: Check for:
   - Hardcoded credentials in Java/Kotlin code
   - Secrets in resources (strings.xml)
   - Keys in shared preferences
   - Credentials in SQLite databases

#### Common Secret Patterns:

```java
// INSECURE: Hardcoded API Key
public static final String API_KEY = "sk_live_abc123xyz789";

// INSECURE: Hardcoded Password
String password = "admin123";

// INSECURE: AWS Credentials
String awsAccessKey = "AKIAIOSFODNN7EXAMPLE";

// INSECURE: Database Password
String dbUrl = "jdbc:mysql://db.example.com:3306/mydb?user=admin&password=secret123";
```

#### Secure Alternatives:

```java
// SECURE: Use BuildConfig for API keys (still needs obfuscation)
String apiKey = BuildConfig.API_KEY;

// SECURE: Use Android Keystore
KeyStore keyStore = KeyStore.getInstance("AndroidKeyStore");

// SECURE: Use encrypted shared preferences
EncryptedSharedPreferences.create(...)

// SECURE: Fetch credentials from secure backend
authenticateAndGetToken(username, password);
```

#### Example Findings:

```
CRITICAL: Hardcoded API Keys
- Location: com.example.config.AppConfig.java
- Issue: Google Maps API key hardcoded in source
- Value: AIzaSyD*********************
- Risk: API key exposure can lead to unauthorized usage and billing

CRITICAL: Hardcoded AWS Credentials
- Location: res/values/strings.xml
- Issue: AWS access key and secret key in resources
- Risk: Complete AWS account compromise possible

HIGH: Database Credentials
- Location: com.example.db.DatabaseHelper.java
- Issue: Database password hardcoded in connection string
- Risk: Unauthorized database access

MEDIUM: Hardcoded OAuth Secret
- Location: com.example.oauth.OAuthManager.java
- Issue: OAuth client secret in code
- Risk: OAuth token theft and impersonation

RECOMMENDATION:
1. Remove all hardcoded secrets from code and resources
2. Use Android Keystore for sensitive data
3. Implement remote configuration for API keys
4. Use environment variables or secure backend for credentials
5. Rotate all exposed credentials immediately
```

### 4. Additional Security Checks

#### Network Security:

- Check for cleartext HTTP usage
- Review certificate pinning implementation
- Analyze TLS/SSL configuration
- Check for insecure WebView settings

#### Code Security:

- SQL injection vulnerabilities
- Path traversal issues
- Insecure data storage
- Debug mode enabled in production
- Exported components without proper protection

#### Binary Protection:

- Code obfuscation status
- Root detection implementation
- Anti-debugging measures
- Certificate validation

## Reporting Template

### Executive Summary

```markdown
# Security Assessment Report
**Application Name:** [App Name]
**Package Name:** com.example.app
**Version:** 1.0.0
**Assessment Date:** [Date]
**Assessed By:** [Your Name]
**MobSF Version:** 3.x

## Overall Security Score: X/100

## Summary of Findings:
- Critical: X
- High: X
- Medium: X
- Low: X
- Info: X
```

### Detailed Findings

```markdown
## Finding #1: [Title]

**Severity:** Critical/High/Medium/Low
**Category:** Insecure Permissions / Weak Cryptography / Hardcoded Secrets / Other
**CWE ID:** CWE-XXX
**OWASP Mobile Top 10:** M1/M2/M3...

### Description:
[Detailed description of the vulnerability]

### Location:
- File: path/to/file.java
- Line: XX
- Component: ComponentName

### Impact:
[What could an attacker do with this vulnerability?]

### Evidence:
```code
[Code snippet showing the vulnerability]
```

### Proof of Concept:
[Steps to reproduce if applicable]

### Recommendation:
[Specific steps to fix the vulnerability]

### References:
- [OWASP Mobile Security Testing Guide](https://owasp.org/www-project-mobile-security-testing-guide/)
- [CWE-XXX](https://cwe.mitre.org/data/definitions/XXX.html)
```

### Full Report Example

```markdown
# Mobile Application Security Assessment

## Application Details
- **Name:** InsecureBank
- **Package:** com.android.insecurebankv2
- **Version:** 1.0
- **Platform:** Android 5.0+
- **Assessment Date:** 2024-11-20

## Executive Summary

This report presents findings from a static security analysis performed using MobSF on the InsecureBank Android application. The assessment identified multiple critical security vulnerabilities that could lead to unauthorized access, data theft, and privacy violations.

**Overall Risk: HIGH**

### Findings Summary
| Severity | Count |
|----------|-------|
| Critical | 5     |
| High     | 8     |
| Medium   | 12    |
| Low      | 6     |
| Info     | 10    |

---

## Critical Findings

### 1. Hardcoded Server Credentials

**Severity:** Critical  
**CWE:** CWE-798 (Use of Hard-coded Credentials)  
**OWASP Mobile:** M9 - Reverse Engineering

**Description:**  
Server authentication credentials are hardcoded in the application source code, allowing anyone who decompiles the APK to access backend systems.

**Location:**  
- File: `com/android/insecurebankv2/ServerAuthenticate.java`
- Lines: 45-47

**Evidence:**
```java
private static final String SERVER_URL = "http://insecurebank.com/api";
private static final String API_KEY = "sk_live_1234567890abcdef";
private static final String SECRET_KEY = "secret_key_hardcoded_123";
```

**Impact:**  
- Complete compromise of backend API
- Unauthorized access to all user data
- Potential for data manipulation and deletion

**Recommendation:**
1. Remove all hardcoded credentials immediately
2. Implement secure credential storage using Android Keystore
3. Use OAuth 2.0 for authentication
4. Rotate compromised API keys
5. Implement certificate pinning

---

### 2. Insecure Cryptographic Storage

**Severity:** Critical  
**CWE:** CWE-327 (Use of a Broken or Risky Cryptographic Algorithm)  
**OWASP Mobile:** M5 - Insufficient Cryptography

**Description:**  
Application uses ECB mode for AES encryption, which is not semantically secure and can reveal patterns in encrypted data.

**Location:**  
- File: `com/android/insecurebankv2/CryptoClass.java`
- Lines: 89-92

**Evidence:**
```java
public String encrypt(String plainText) {
    Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
    cipher.init(Cipher.ENCRYPT_MODE, getKey());
    // ...
}
```

**Impact:**  
- Pattern analysis of encrypted data
- Potential plaintext recovery
- Violation of data confidentiality

**Recommendation:**
1. Use AES/GCM or AES/CBC mode with random IV
2. Implement proper key management using Android Keystore
3. Use authenticated encryption (GCM mode preferred)
4. Example secure implementation:
```java
Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
GCMParameterSpec spec = new GCMParameterSpec(128, iv);
cipher.init(Cipher.ENCRYPT_MODE, key, spec);
```

---

### 3. Dangerous Permissions Without Justification

**Severity:** High  
**CWE:** CWE-250 (Execution with Unnecessary Privileges)  
**OWASP Mobile:** M1 - Improper Platform Usage

**Description:**  
Application requests dangerous permissions (READ_SMS, READ_CONTACTS) that are not required for core functionality.

**Location:**  
- File: `AndroidManifest.xml`
- Lines: 12-15

**Evidence:**
```xml
<uses-permission android:name="android.permission.READ_SMS"/>
<uses-permission android:name="android.permission.SEND_SMS"/>
<uses-permission android:name="android.permission.READ_CONTACTS"/>
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
```

**Impact:**  
- Potential privacy violation
- Unauthorized data collection
- User data exfiltration risk

**Recommendation:**
1. Remove unnecessary permissions
2. Request permissions at runtime only when needed
3. Provide clear justification in privacy policy
4. Implement principle of least privilege

---

## High Findings

### 4. Insecure Data Storage

**Severity:** High  
**CWE:** CWE-312 (Cleartext Storage of Sensitive Information)  
**OWASP Mobile:** M2 - Insecure Data Storage

**Description:**  
User credentials stored in SharedPreferences without encryption.

**Location:**  
- File: `com/android/insecurebankv2/LoginActivity.java`

**Evidence:**
```java
SharedPreferences prefs = getSharedPreferences("user_data", MODE_PRIVATE);
prefs.edit().putString("username", username).apply();
prefs.edit().putString("password", password).apply();
```

**Recommendation:**
Use EncryptedSharedPreferences from AndroidX Security library.

---

### 5. Weak Hash Algorithm for Passwords

**Severity:** High  
**CWE:** CWE-328 (Use of Weak Hash)  
**OWASP Mobile:** M5 - Insufficient Cryptography

**Description:**  
MD5 hash used for password storage, which is cryptographically broken.

**Recommendation:**
Implement PBKDF2, bcrypt, or Argon2 for password hashing.

---

## Recommendations Summary

### Immediate Actions (Critical)
1. Remove and rotate all hardcoded credentials
2. Fix cryptographic implementations (no ECB mode)
3. Implement Android Keystore for sensitive data
4. Remove unnecessary dangerous permissions

### Short-term Actions (High)
1. Implement encrypted storage for sensitive data
2. Update hash algorithms for passwords
3. Add certificate pinning
4. Implement root detection

### Long-term Actions (Medium)
1. Security code review process
2. Implement ProGuard/R8 obfuscation
3. Regular security testing cycle
4. Security training for developers

---

## References

- [OWASP Mobile Security Testing Guide](https://owasp.org/www-project-mobile-security-testing-guide/)
- [Android Security Best Practices](https://developer.android.com/topic/security/best-practices)
- [CWE Database](https://cwe.mitre.org/)
- [MobSF Documentation](https://github.com/MobSF/Mobile-Security-Framework-MobSF)

---

**Report Generated:** [Date]  
**Analyst:** [Name]  
**Tool Used:** MobSF v3.x
```

## Best Practices

### Before Scanning
1. Always get proper authorization before testing
2. Use vulnerable test apps for learning
3. Verify APK integrity
4. Document the scope of testing

### During Analysis
1. Take detailed notes of findings
2. Screenshot important vulnerabilities
3. Validate findings manually when possible
4. Prioritize by severity and exploitability

### After Scanning
1. Create comprehensive reports
2. Provide actionable recommendations
3. Include remediation timelines
4. Track vulnerability resolution

## Learning Resources

### Video Tutorials
- "MobSF tutorial beginner" - YouTube
- "Android pentesting MobSF full tutorial" - YouTube
- OWASP Mobile Security Testing Guide videos

### Documentation
- [MobSF GitHub](https://github.com/MobSF/Mobile-Security-Framework-MobSF)
- [OWASP MSTG](https://owasp.org/www-project-mobile-security-testing-guide/)
- [Android Security Documentation](https://source.android.com/security)

### Practice Applications
- DIVA Android
- InsecureBankv2
- OWASP UnCrackable Apps
- AndroGoat

## Conclusion

This guide provides a comprehensive framework for conducting Android application security analysis using MobSF. Regular security testing should be integrated into the development lifecycle to identify and remediate vulnerabilities before production deployment.

For Flipkart and similar enterprise environments, this process should be:
- Automated in CI/CD pipeline
- Conducted before each release
- Documented thoroughly
- Followed by remediation tracking
