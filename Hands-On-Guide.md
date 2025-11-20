# Hands-On Guide: Android Security Analysis with MobSF

## Quick Start Tutorial

This is a practical, step-by-step guide to get you started with Android security testing using MobSF within 30 minutes.

## Part 1: Setup (10 minutes)

### Step 1: Install Docker (if not already installed)

**Linux/Ubuntu:**
```bash
# Update package list
sudo apt-get update

# Install Docker
sudo apt-get install -y docker.io

# Start Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Add your user to docker group (to run without sudo)
sudo usermod -aG docker $USER

# Log out and log back in for group changes to take effect
```

**Mac:**
```bash
# Install using Homebrew
brew install --cask docker

# Or download Docker Desktop from:
# https://www.docker.com/products/docker-desktop
```

**Windows:**
- Download Docker Desktop from https://www.docker.com/products/docker-desktop
- Run the installer
- Restart your computer

### Step 2: Launch MobSF

```bash
# Pull MobSF Docker image
docker pull opensecurity/mobile-security-framework-mobsf:latest

# Run MobSF (this will start the server)
docker run -it --rm \
  -p 8000:8000 \
  -v /tmp/mobsf:/home/mobsf/.MobSF \
  opensecurity/mobile-security-framework-mobsf:latest
```

Wait for the message: `Starting MobSF Server...`

### Step 3: Access MobSF

Open your browser and navigate to:
```
http://localhost:8000
```

You should see the MobSF dashboard with an upload interface.

## Part 2: Download Test APK (5 minutes)

### Option A: Using DIVA (Recommended for Beginners)

```bash
# Create a directory for test APKs
mkdir -p ~/mobile-security/apks
cd ~/mobile-security/apks

# Download DIVA APK
wget https://github.com/payatu/diva-android/raw/master/diva-beta.apk

# Verify download
ls -lh diva-beta.apk
```

### Option B: Using InsecureBankv2

```bash
cd ~/mobile-security/apks

# Download pre-built APK
wget https://github.com/dineshshetty/Android-InsecureBankv2/releases/download/v1.0/InsecureBankv2.apk
```

### Option C: Using OWASP UnCrackable Level 1

```bash
cd ~/mobile-security/apks

# Download UnCrackable Level 1
wget https://github.com/OWASP/owasp-mstg/raw/master/Crackmes/Android/Level_01/UnCrackable-Level1.apk
```

## Part 3: Scan APK (5 minutes)

### Upload and Analyze

1. **Go to MobSF Dashboard**
   - Navigate to `http://localhost:8000`

2. **Upload APK**
   - Click "Upload & Analyze" or drag-and-drop
   - Select your downloaded APK (e.g., `diva-beta.apk`)
   - Click "Analyze"

3. **Wait for Analysis**
   - Analysis progress will be shown
   - Typically takes 2-5 minutes
   - Coffee time! ☕

4. **View Results**
   - Once complete, you'll see the security report
   - Note the security score (0-100)

## Part 4: Analyze Findings (10 minutes)

### 4.1 Check Security Score

At the top of the report, you'll see:
```
Security Score: XX/100
```

- 80-100: Good security
- 60-79: Moderate security
- 40-59: Poor security
- 0-39: Critical security issues

### 4.2 Review Application Info

**What to note:**
- Package name
- App name
- Version
- Minimum SDK version
- Target SDK version
- Permissions count

### 4.3 Analyze Insecure Permissions

**Navigate to:** "Permissions" section

**Look for:**

1. **Dangerous Permissions:**
   - READ_SMS
   - SEND_SMS
   - READ_CONTACTS
   - ACCESS_FINE_LOCATION
   - CAMERA
   - RECORD_AUDIO
   - READ_CALL_LOG
   - WRITE_EXTERNAL_STORAGE

2. **For each permission, ask:**
   - Is this necessary for the app's core functionality?
   - Can this be exploited for data theft?
   - Is this documented in the privacy policy?

**Example finding:**
```
Finding: Unnecessary SMS Permission
Severity: HIGH
Permission: android.permission.READ_SMS
Justification: Banking app does not need to read SMS messages
Risk: Can be used to steal 2FA codes or intercept sensitive messages
```

### 4.4 Find Weak Cryptography

**Navigate to:** "Code Analysis" section

**Search for:**

1. **ECB Mode Usage**
   ```
   Pattern: AES/ECB
   Risk: Not semantically secure, reveals patterns
   ```

2. **Weak Hash Algorithms**
   ```
   Pattern: MD5, SHA1 (for passwords)
   Risk: Cryptographically broken
   ```

3. **Hardcoded Keys**
   ```
   Pattern: Static encryption keys in code
   Risk: All data encrypted with same key
   ```

**Example finding:**
```
Finding: Weak Cryptographic Algorithm
Severity: CRITICAL
Location: com.android.diva.Crypto.java
Issue: AES/ECB/PKCS5Padding detected
Evidence:
  Line 45: Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
Risk: ECB mode can leak patterns in encrypted data
CWE: CWE-327
```

### 4.5 Discover Hardcoded Secrets

**Navigate to:** "Strings" section or "Code Analysis"

**Look for:**

1. **API Keys**
   ```
   Pattern: "api_key", "apikey", "API_KEY"
   Example: api_key = "sk_live_abc123xyz"
   ```

2. **Passwords**
   ```
   Pattern: "password", "passwd", "pwd"
   Example: String password = "admin123"
   ```

3. **URLs with Credentials**
   ```
   Pattern: http://user:pass@server.com
   Example: jdbc:mysql://root:secret@db.local/mydb
   ```

4. **Cloud Credentials**
   ```
   Pattern: AWS, Azure, GCP keys
   Example: aws_access_key_id = "AKIA..."
   ```

**Example finding:**
```
Finding: Hardcoded API Key
Severity: CRITICAL
Location: res/values/strings.xml
Line: 23
Evidence:
  <string name="api_key">AIzaSyBxxxxxxxxxxxxxxxxxxxxxxxxxxx</string>
Risk: API key exposure can lead to:
  - Unauthorized API usage
  - Data breaches
  - Financial loss from API abuse
Impact: HIGH - Immediate key rotation required
```

### 4.6 Additional Security Issues

**Check these sections:**

1. **Network Security**
   - Cleartext HTTP traffic
   - Certificate validation issues
   - Insecure TLS configuration

2. **Manifest Analysis**
   - Exported components without permission
   - Debug mode enabled
   - Backup enabled
   - Intent filter issues

3. **Code Quality**
   - SQL injection risks
   - Path traversal vulnerabilities
   - WebView security issues
   - Log sensitive data

## Part 5: Document Findings

### Create Your Report

Use this template:

```markdown
# Security Analysis Report: [App Name]

**Date:** [Current Date]
**Analyzed By:** [Your Name]
**Tool:** MobSF v3.x
**APK:** [APK Name]

## Executive Summary

Overall Security Score: XX/100
Risk Level: [Critical/High/Medium/Low]

## Key Findings

### Critical Issues: X
1. [Issue 1]
2. [Issue 2]

### High Issues: X
1. [Issue 1]
2. [Issue 2]

### Medium Issues: X
1. [Issue 1]

---

## Detailed Findings

### Finding #1: [Title]

**Severity:** Critical
**Category:** [Permissions/Crypto/Secrets/Other]
**CWE:** CWE-XXX

**Description:**
[What is the vulnerability?]

**Location:**
- File: [filename.java]
- Line: [line number]

**Evidence:**
```
[Code snippet or screenshot]
```

**Impact:**
[What can an attacker do?]

**Recommendation:**
[How to fix it?]

---

[Repeat for each finding]

## Summary of Recommendations

### Immediate Actions
1. [Action 1]
2. [Action 2]

### Short-term
1. [Action 1]

### Long-term
1. [Action 1]

## References
- MobSF Report: [Link]
- OWASP MSTG: https://owasp.org/www-project-mobile-security-testing-guide/
```

## Practical Example: DIVA APK Analysis

### Expected Findings for DIVA

When you scan DIVA APK, you should find:

#### 1. Insecure Data Storage
```
Location: SharedPreferences
Issue: Credentials stored in plaintext
File: com.android.diva.InsecureDataStorage1Activity.java
```

#### 2. Insecure Logging
```
Issue: Sensitive data logged via android.util.Log
Risk: Logs can be read by other apps
```

#### 3. Hardcoded Secrets
```
Location: res/values/strings.xml
Issue: Vendor key hardcoded
Value: vendorsecretkey
```

#### 4. Weak Cryptography
```
Location: Crypto activities
Issue: ECB mode, hardcoded keys
Files: Crypto*.java
```

#### 5. SQL Injection
```
Location: Database activities
Issue: Unsanitized user input in SQL queries
Files: SQLInjection*.java
```

#### 6. Insecure Permissions
```
Permissions: READ_PHONE_STATE, WRITE_EXTERNAL_STORAGE
Justification: Not required for app functionality
```

## Common Issues and Troubleshooting

### MobSF Won't Start

```bash
# Check if port 8000 is already in use
sudo lsof -i :8000

# Kill the process using the port
sudo kill -9 [PID]

# Or use a different port
docker run -it --rm -p 8080:8000 opensecurity/mobile-security-framework-mobsf
```

### APK Upload Fails

1. **Check APK is valid:**
   ```bash
   file myapp.apk
   # Should show: Android application package file
   ```

2. **Check file size:**
   - Large APKs (>100MB) may take longer
   - Increase Docker memory if needed

3. **Check file permissions:**
   ```bash
   chmod 644 myapp.apk
   ```

### Analysis Stuck

1. **Check Docker logs:**
   ```bash
   docker logs [container-id]
   ```

2. **Restart MobSF:**
   ```bash
   # Stop container
   docker stop [container-id]
   
   # Start fresh
   docker run -it --rm -p 8000:8000 opensecurity/mobile-security-framework-mobsf
   ```

## Next Steps

### Beginner Level
- [x] Complete this hands-on guide
- [ ] Analyze 3 vulnerable APKs (DIVA, InsecureBank, UnCrackable)
- [ ] Document 10+ security findings
- [ ] Learn OWASP Mobile Top 10

### Intermediate Level
- [ ] Set up dynamic analysis with MobSF
- [ ] Learn to use Frida for runtime analysis
- [ ] Practice with real-world APKs (with permission)
- [ ] Understand Android Security Architecture

### Advanced Level
- [ ] Automate MobSF in CI/CD pipeline
- [ ] Develop custom security rules
- [ ] Contribute to MobSF project
- [ ] Write security blog posts

## Learning Resources

### YouTube Tutorials
Search for:
- "MobSF tutorial beginner"
- "Android pentesting MobSF full tutorial"
- "DIVA Android walkthrough"
- "Mobile application security testing"

### Recommended Videos
1. MobSF Tutorial by Payatu
2. Android Security Testing by OWASP
3. Mobile Pentesting Course

### Reading Materials
- OWASP Mobile Security Testing Guide (MSTG)
- Android Security Internals
- The Mobile Application Hacker's Handbook

### Practice Platforms
- DIVA Android
- InsecureBankv2
- OWASP UnCrackable Apps (Level 1-3)
- AndroGoat
- InjuredAndroid

## Checklist for Flipkart JD Requirement

- [ ] Downloaded and analyzed a simple APK ✓
- [ ] Scanned with MobSF ✓
- [ ] Identified insecure permissions ✓
- [ ] Identified weak cryptography ✓
- [ ] Identified hardcoded secrets ✓
- [ ] Documented findings in a report ✓
- [ ] Understood remediation strategies ✓
- [ ] Learned about OWASP Mobile Top 10 ✓

## Tips for Success

### 1. Start Simple
- Use vulnerable training apps first
- Don't jump to complex apps immediately
- Focus on understanding one vulnerability type at a time

### 2. Take Notes
- Document everything you find
- Screenshot important findings
- Create a finding database

### 3. Understand Impact
- Don't just list vulnerabilities
- Explain the real-world impact
- Think like an attacker

### 4. Learn Remediation
- Know how to fix each vulnerability
- Understand secure coding practices
- Study Android security best practices

### 5. Stay Updated
- Follow security researchers
- Read security blogs
- Join security communities
- Attend conferences/webinars

## Conclusion

You now have:
1. ✓ Working MobSF setup
2. ✓ Test APKs to analyze
3. ✓ Knowledge of what to look for
4. ✓ Documentation templates
5. ✓ Next steps for learning

**Time to practice!** 🚀

Start with DIVA APK and try to find all the vulnerabilities mentioned in this guide. Good luck!
