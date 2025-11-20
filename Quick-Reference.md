# Quick Reference Guide - MobSF Android Security

## Quick Commands

### MobSF Setup (Docker)
```bash
# Pull and run MobSF
docker pull opensecurity/mobile-security-framework-mobsf:latest
docker run -it --rm -p 8000:8000 opensecurity/mobile-security-framework-mobsf:latest

# Access: http://localhost:8000
```

### Download Test APKs
```bash
# DIVA (Recommended for beginners)
wget https://github.com/payatu/diva-android/raw/master/diva-beta.apk

# InsecureBankv2
wget https://github.com/dineshshetty/Android-InsecureBankv2/releases/download/v1.0/InsecureBankv2.apk

# OWASP UnCrackable Level 1
wget https://github.com/OWASP/owasp-mstg/raw/master/Crackmes/Android/Level_01/UnCrackable-Level1.apk
```

## Security Checklist

### 1. Permissions Analysis
- [ ] Check for dangerous permissions
- [ ] Verify each permission is necessary
- [ ] Look for over-privileged access
- [ ] Document unused permissions

**High-Risk Permissions:**
- `READ_SMS` / `SEND_SMS` - SMS access
- `READ_CONTACTS` - Contact access
- `ACCESS_FINE_LOCATION` - GPS location
- `CAMERA` - Camera access
- `RECORD_AUDIO` - Microphone access
- `READ_CALL_LOG` - Call history
- `WRITE_EXTERNAL_STORAGE` - File access

### 2. Cryptography Analysis
- [ ] Check for ECB mode usage
- [ ] Look for hardcoded encryption keys
- [ ] Verify secure random number generation
- [ ] Check for weak algorithms (MD5, SHA1 for passwords)
- [ ] Verify proper IV usage

**Insecure Patterns:**
```java
// BAD
Cipher.getInstance("AES/ECB/PKCS5Padding")
String key = "hardcodedkey123"
MessageDigest.getInstance("MD5")
```

**Secure Patterns:**
```java
// GOOD
Cipher.getInstance("AES/GCM/NoPadding")
KeyStore.getInstance("AndroidKeyStore")
MessageDigest.getInstance("SHA-256")
```

### 3. Secrets Analysis
- [ ] Search for API keys
- [ ] Look for hardcoded passwords
- [ ] Check for OAuth secrets
- [ ] Find database credentials
- [ ] Identify cloud credentials (AWS, Azure, GCP)

**Search Patterns:**
- `api_key`, `apikey`, `API_KEY`
- `password`, `passwd`, `pwd`
- `secret`, `token`, `auth`
- `AKIA` (AWS Access Key)
- `AIza` (Google API Key)

### 4. Network Security
- [ ] Check for cleartext HTTP
- [ ] Verify certificate pinning
- [ ] Check TLS configuration
- [ ] Verify WebView security settings

### 5. Data Storage
- [ ] Check SharedPreferences encryption
- [ ] Verify SQLite database security
- [ ] Check for data in external storage
- [ ] Verify file permissions

### 6. Code Quality
- [ ] Check for SQL injection
- [ ] Look for path traversal
- [ ] Verify input validation
- [ ] Check for WebView vulnerabilities

### 7. Binary Protection
- [ ] Check for code obfuscation
- [ ] Verify root detection
- [ ] Check for anti-debugging
- [ ] Verify certificate validation

## Common Vulnerabilities Quick Lookup

### Insecure Permissions (HIGH)
**Issue:** Unnecessary dangerous permissions  
**CWE:** CWE-250  
**Fix:** Remove unused permissions from AndroidManifest.xml

### Weak Crypto - ECB Mode (CRITICAL)
**Issue:** AES/ECB mode usage  
**CWE:** CWE-327  
**Fix:** Use AES/GCM or AES/CBC with random IV

### Hardcoded Secrets (CRITICAL)
**Issue:** API keys in code/resources  
**CWE:** CWE-798  
**Fix:** Use Android Keystore, remote config

### Plaintext Storage (CRITICAL)
**Issue:** Sensitive data in SharedPreferences  
**CWE:** CWE-312  
**Fix:** Use EncryptedSharedPreferences

### SQL Injection (CRITICAL)
**Issue:** Unsanitized user input in SQL  
**CWE:** CWE-89  
**Fix:** Use parameterized queries or Room

### Insecure Logging (HIGH)
**Issue:** Sensitive data in logs  
**CWE:** CWE-532  
**Fix:** Remove logs or use ProGuard to strip

### Exported Components (HIGH)
**Issue:** Components exported without permission  
**CWE:** CWE-926  
**Fix:** Set exported="false" or add permissions

### Debug Mode (MEDIUM)
**Issue:** debuggable="true" in production  
**Fix:** Set debuggable="false"

### Backup Enabled (MEDIUM)
**Issue:** allowBackup="true"  
**Fix:** Set allowBackup="false"

## OWASP Mobile Top 10 Quick Reference

1. **M1: Improper Platform Usage**
   - Misuse of platform features
   - Insecure permissions

2. **M2: Insecure Data Storage**
   - Plaintext storage
   - Insecure SharedPreferences
   - Logs with sensitive data

3. **M3: Insecure Communication**
   - Cleartext HTTP
   - Weak TLS
   - No certificate pinning

4. **M4: Insecure Authentication**
   - Weak password policy
   - No MFA
   - Insecure session management

5. **M5: Insufficient Cryptography**
   - Weak algorithms
   - ECB mode
   - Hardcoded keys

6. **M6: Insecure Authorization**
   - Missing authorization checks
   - Insecure direct object references

7. **M7: Client Code Quality**
   - SQL injection
   - XSS in WebView
   - Buffer overflows

8. **M8: Code Tampering**
   - No integrity checks
   - Missing obfuscation
   - No root detection

9. **M9: Reverse Engineering**
   - No code obfuscation
   - Hardcoded secrets
   - Missing anti-tampering

10. **M10: Extraneous Functionality**
    - Debug code in production
    - Test backdoors

## MobSF Report Sections

### Application Info
- Package name, version
- Permissions, components
- SDK versions

### Security Analysis
- Certificate info
- Manifest analysis
- Code analysis

### Permissions
- Dangerous permissions
- Normal permissions
- Custom permissions

### Malware Analysis
- APK scans
- Domain malware check

### Code Analysis
- Security issues by severity
- File-by-file analysis

### Strings
- Hardcoded strings
- URLs
- Potential secrets

### Components
- Activities
- Services
- Receivers
- Providers

## Severity Levels

| Level | Score | Description |
|-------|-------|-------------|
| CRITICAL | 90-100 | Immediate exploitation possible |
| HIGH | 70-89 | Significant security impact |
| MEDIUM | 40-69 | Limited impact or specific conditions |
| LOW | 10-39 | Minimal security impact |
| INFO | 0-9 | Best practice violation |

## Report Writing Tips

### Executive Summary
- Overall security score
- Total findings by severity
- Top 3-5 critical risks
- Business impact

### Detailed Findings
- Clear title
- Severity level
- CWE/OWASP mapping
- Location (file, line)
- Evidence (code snippet)
- Impact analysis
- Remediation steps
- References

### Recommendations
- Immediate actions (Critical)
- Short-term (High)
- Long-term (Medium/Low)
- Process improvements

## Useful ADB Commands

```bash
# List packages
adb shell pm list packages | grep <app>

# Find APK path
adb shell pm path com.example.app

# Pull APK
adb pull /data/app/com.example.app/base.apk

# View app data (requires root)
adb shell
su
cd /data/data/com.example.app
ls -la

# View SharedPreferences
cat shared_prefs/*.xml

# View databases
sqlite3 databases/*.db
.tables
.schema
SELECT * FROM users;

# View logs
adb logcat | grep <package>
```

## Tools Ecosystem

### Static Analysis
- **MobSF** - Comprehensive analysis
- **APKTool** - APK decompilation
- **JADX** - Java decompilation
- **androguard** - Python-based analysis

### Dynamic Analysis
- **Frida** - Runtime instrumentation
- **Objection** - Runtime mobile exploration
- **Drozer** - Security audit framework

### Network Analysis
- **Burp Suite** - HTTP proxy
- **mitmproxy** - Python-based proxy
- **Wireshark** - Network protocol analyzer

### Reverse Engineering
- **JEB** - Professional decompiler
- **IDA Pro** - Disassembler
- **Ghidra** - NSA reverse engineering tool

## Learning Path

### Beginner (Week 1-2)
- [x] Install MobSF
- [x] Scan DIVA APK
- [x] Identify top 5 vulnerabilities
- [x] Write first security report
- [ ] Learn OWASP Mobile Top 10

### Intermediate (Week 3-4)
- [ ] Dynamic analysis with Frida
- [ ] Burp Suite mobile testing
- [ ] Analyze 5+ vulnerable apps
- [ ] Understand Android security architecture

### Advanced (Month 2-3)
- [ ] Custom Frida scripts
- [ ] SSL pinning bypass
- [ ] Root detection bypass
- [ ] Automated security testing
- [ ] CI/CD integration

### Expert (Month 4+)
- [ ] Write security tools
- [ ] Contribute to MobSF
- [ ] Bug bounty hunting
- [ ] Security research

## YouTube Tutorial Keywords

Search for:
- "MobSF tutorial beginner"
- "Android pentesting MobSF full tutorial"
- "DIVA Android walkthrough"
- "Mobile application security testing"
- "Android security analysis"
- "MobSF installation guide"
- "Android app hacking tutorial"

## Resources

### Documentation
- [MobSF Docs](https://mobsf.github.io/docs/)
- [OWASP MSTG](https://owasp.org/www-project-mobile-security-testing-guide/)
- [Android Security](https://source.android.com/security)

### Practice Apps
- DIVA Android
- InsecureBankv2
- OWASP UnCrackable (Level 1-3)
- AndroGoat
- InjuredAndroid
- DVHMA

### Communities
- OWASP Mobile Security Project
- r/androiddev
- r/netsec
- Stack Overflow

### Certifications
- OSCP (Offensive Security)
- CEH (Certified Ethical Hacker)
- GPEN (GIAC Penetration Tester)
- Mobile Security Certified Professional

## Troubleshooting

### MobSF Won't Start
```bash
# Check port
sudo lsof -i :8000
# Use different port
docker run -p 8080:8000 ...
```

### APK Upload Fails
```bash
# Verify APK
file myapp.apk
# Check permissions
chmod 644 myapp.apk
```

### Analysis Hangs
- Restart Docker container
- Check Docker logs
- Increase Docker memory (4GB+)

### Can't Access MobSF UI
- Check firewall
- Verify port mapping
- Try 127.0.0.1 instead of localhost

## Quick Checklist for Flipkart JD

✅ **Download a simple APK**
- DIVA, InsecureBankv2, or UnCrackable

✅ **Scan with MobSF**
- Install MobSF via Docker
- Upload and analyze APK

✅ **Identify insecure permissions**
- Review dangerous permissions
- Check necessity and justification

✅ **Identify weak crypto**
- Look for ECB mode
- Check for weak algorithms
- Find hardcoded keys

✅ **Identify secrets**
- API keys
- Passwords
- OAuth secrets
- Cloud credentials

✅ **Document findings**
- Write comprehensive report
- Include severity ratings
- Provide remediation steps
- Map to OWASP/CWE

## Time Estimates

- **Setup MobSF:** 10 minutes
- **Download APK:** 5 minutes
- **Scan APK:** 5 minutes
- **Analyze findings:** 30-60 minutes
- **Write report:** 1-2 hours
- **Total:** ~2-3 hours for first time

---

**Last Updated:** November 20, 2024  
**Version:** 1.0
