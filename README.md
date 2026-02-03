# Test Case: Poetry Tool-Versions Critical - VULNERABILITY DETECTION TEST

## Package Manager
Poetry

## Python Version Detection
- **Source**: .tool-versions (PRIORITY #2 - after .python-version)
- **Expected Version**: 3.11.0
- **Conflict**: pyproject.toml says ^3.8 (SHOULD BE IGNORED)

## Dependencies (CRITICAL - CONTAINS KNOWN VULNERABILITIES)
- `certifi==2022.12.7` - **CVE-2023-37920** (CRITICAL) with marker `python_version >= "3.11"`
- `cryptography==39.0.0` - **CVE-2023-23931** (HIGH) with marker `python_version >= "3.11"`
- `urllib3==1.26.15` - **CVE-2023-43804** (MEDIUM) with marker `python_version >= "3.11"`
- `requests>=2.28.0` - Depends on the vulnerable packages above

## Test Purpose
**CRITICAL VULNERABILITY DETECTION TEST WITH ENVIRONMENT MARKERS AND .tool-versions PRIORITY**

### Why This Test Matters
Multiple vulnerable dependencies have environment markers: `python_version >= "3.11"`

If tool uses **3.11.0** from .tool-versions:
- ✅ Environment markers evaluate to TRUE
- ✅ Vulnerable packages (certifi, cryptography, urllib3) are installed
- ✅ **Security scanner SHOULD detect 3 CVEs**
- ✅ Lock file was created with this version
- 🔴 **Expected CVEs**: CVE-2023-37920, CVE-2023-23931, CVE-2023-43804

If tool uses **3.8** from pyproject.toml:
- ❌ Environment markers evaluate to FALSE
- ❌ Vulnerable packages are SKIPPED during installation
- ❌ **Security scanner MISSES all vulnerabilities** (FALSE NEGATIVE)
- ❌ Lock file is inconsistent with resolved dependencies
- 🔴 **CRITICAL SECURITY ISSUE**: Silent omission of vulnerable code

### .tool-versions Priority Test
This specifically tests that tools properly detect and prioritize `.tool-versions` over `pyproject.toml`.
- .tool-versions is commonly used with asdf and mise version managers
- It should override the Python version in pyproject.toml
- The lock file was generated using Python 3.11.0 from .tool-versions

### Important Note on Environment Markers
The vulnerable packages (certifi, cryptography, urllib3) **technically support Python 3.8+**, but we use environment markers `python_version >= '3.11'` to create a test scenario where:
- The packages were added to the project **when using Python 3.11** (as specified in .tool-versions)
- They are conditionally included **only when Python >= 3.11 is detected**
- This simulates real-world scenarios where:
  - Certain dependencies are only needed for newer Python versions
  - Security scanners must respect the same Python version used during lock file generation
  - Wrong version detection leads to incomplete dependency resolution

## Expected Behavior
1. Tool detects .tool-versions (3.11.0) as higher priority than pyproject.toml
2. Tool ignores pyproject.toml python = "^3.8"
3. Installs ALL dependencies including tomli (based on Python 3.11)
4. Dependency resolution succeeds with lock file consistency

## Real-World Impact
Environment markers combined with version files are commonly used for:
- Python version-specific dependencies
- Conditional security patches
- Lock file generation with specific Python versions
- Team standardization on Python versions via .tool-versions

**Wrong version detection = silent vulnerability omission = false security reports**

This is a CRITICAL security issue:
1. Vulnerable packages are installed in production (Python 3.11 from .tool-versions)
2. Security scanner uses wrong Python version (3.8 from pyproject.toml)
3. Scanner skips vulnerable packages due to environment markers
4. **Security scan shows "NO VULNERABILITIES" when 3 CVEs are present**
5. Organization has false sense of security

### Known Vulnerabilities in This Test
- **CVE-2023-37920** (certifi 2022.12.7): Certifi removes TrustCor root certificates
- **CVE-2023-23931** (cryptography 39.0.0): Cipher.update_into can corrupt memory
- **CVE-2023-43804** (urllib3 1.26.15): Cookie request header leak via HTTP redirect

## Verification Test
```bash
# Verify Python version detection
cat .tool-versions  # Should show 3.11.0

# Install dependencies
poetry install

# Verify vulnerable packages were installed (should succeed with Python 3.11)
poetry run python -c "import certifi; print('✅ certifi version:', certifi.__version__)"
poetry run python -c "import cryptography; print('✅ cryptography version:', cryptography.__version__)"
poetry run python -c "import urllib3; print('✅ urllib3 version:', urllib3.__version__)"

# Run security scan
# Expected: 3 vulnerabilities detected (CVE-2023-37920, CVE-2023-23931, CVE-2023-43804)
```

### Security Scanner Test Results
**CORRECT (uses .tool-versions Python 3.11.0)**:
- ✅ Detects certifi 2022.12.7 - CVE-2023-37920
- ✅ Detects cryptography 39.0.0 - CVE-2023-23931
- ✅ Detects urllib3 1.26.15 - CVE-2023-43804
- **Result**: 3 vulnerabilities found ✅

**INCORRECT (uses pyproject.toml Python ^3.8)**:
- ❌ Skips certifi (marker not satisfied)
- ❌ Skips cryptography (marker not satisfied)
- ❌ Skips urllib3 (marker not satisfied)
- **Result**: 0 vulnerabilities found 🔴 FALSE NEGATIVE