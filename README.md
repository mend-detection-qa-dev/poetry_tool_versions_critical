# Test Case: Poetry Tool-Versions Critical - VULNERABILITY DETECTION TEST

## Package Manager
Poetry

## Python Version Detection
- **Source**: .tool-versions (PRIORITY #2 - after .python-version)
- **Expected Version**: 3.11.0
- **Conflict**: pyproject.toml says ^3.8 (SHOULD BE IGNORED)

## Dependencies (CRITICAL - CONTAINS KNOWN VULNERABILITIES)
- `numpy==1.26.0` - **Requires Python >=3.9** (INCOMPATIBLE with Python 3.8)
- `pillow==9.3.0` - **CVE-2023-50447** (Arbitrary code execution) with marker `python_version >= "3.11"`
- `GitPython==3.1.30` - **CVE-2023-40267** (Remote code execution) with marker `python_version >= "3.11"`
- `aiohttp==3.8.5` - **CVE-2024-23334** (Directory traversal) with marker `python_version >= "3.11"`

## Test Purpose
**CRITICAL VULNERABILITY DETECTION TEST WITH ENVIRONMENT MARKERS AND .tool-versions PRIORITY**

### Why This Test Matters
Dependencies have strict Python version requirements and environment markers.

If tool uses **3.11.0** from .tool-versions:
- ✅ numpy 1.26.0 satisfies Python >=3.9 requirement
- ✅ Environment markers evaluate to TRUE for pillow, GitPython, aiohttp
- ✅ Vulnerable packages are installed
- ✅ **Security scanner SHOULD detect 3 CVEs**
- ✅ Lock file was created with this version
- 🔴 **Expected CVEs**: CVE-2023-50447, CVE-2023-40267, CVE-2024-23334

If tool uses **3.8** from pyproject.toml:
- ❌ numpy 1.26.0 FAILS Python >=3.9 requirement (INCOMPATIBLE)
- ❌ Environment markers evaluate to FALSE for other packages
- ❌ Vulnerable packages are SKIPPED or installation FAILS
- ❌ **Security scanner MISSES all vulnerabilities** (FALSE NEGATIVE)
- ❌ Lock file is inconsistent with resolved dependencies
- 🔴 **CRITICAL SECURITY ISSUE**: Wrong Python version = invisible vulnerabilities

### .tool-versions Priority Test
This specifically tests that tools properly detect and prioritize `.tool-versions` over `pyproject.toml`.
- .tool-versions is commonly used with asdf and mise version managers
- It should override the Python version in pyproject.toml
- The lock file was generated using Python 3.11.0 from .tool-versions

### Important Note on Python Version Requirements
This test uses a combination of **actual Python version incompatibilities** and **environment markers**:

1. **numpy 1.26.0** genuinely requires Python >=3.9 and **will NOT work** with Python 3.8
2. **pillow, GitPython, aiohttp** use environment markers `python_version >= '3.11'` to simulate:
   - Packages added to the project when using Python 3.11 (as specified in .tool-versions)
   - Conditional dependencies based on Python version detection
   - Real-world scenarios where certain dependencies are only needed for newer Python versions

This creates a realistic test scenario where:
- Lock file was generated with Python 3.11 from .tool-versions
- Security scanners must use the **same Python version** to properly resolve dependencies
- Using the wrong Python version (3.8 from pyproject.toml) causes:
  - numpy installation to FAIL (incompatible)
  - Other packages to be SKIPPED (markers not satisfied)
  - All vulnerabilities to become INVISIBLE to security scans

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
- **CVE-2023-50447** (pillow 9.3.0): Arbitrary code execution via crafted image file
- **CVE-2023-40267** (GitPython 3.1.30): Remote code execution via malicious repository
- **CVE-2024-23334** (aiohttp 3.8.5): Directory traversal vulnerability in static file serving

### Python Version Incompatibility
- **numpy 1.26.0**: Requires Python >=3.9, drops support for Python 3.8
  - Using Python 3.8: Installation FAILS with compatibility error
  - Using Python 3.11: Installation succeeds

## Verification Test
```bash
# Verify Python version detection
cat .tool-versions  # Should show 3.11.0

# Install dependencies (requires Python 3.11)
poetry install

# Verify vulnerable packages were installed (should succeed with Python 3.11)
poetry run python -c "import numpy; print('✅ numpy version:', numpy.__version__)"
poetry run python -c "import PIL; print('✅ pillow version:', PIL.__version__)"
poetry run python -c "import git; print('✅ GitPython version:', git.__version__)"
poetry run python -c "import aiohttp; print('✅ aiohttp version:', aiohttp.__version__)"

# Run security scan
# Expected: 3 vulnerabilities detected (CVE-2023-50447, CVE-2023-40267, CVE-2024-23334)
```

### Security Scanner Test Results
**CORRECT (uses .tool-versions Python 3.11.0)**:
- ✅ Installs numpy 1.26.0 (satisfies Python >=3.9)
- ✅ Detects pillow 9.3.0 - CVE-2023-50447
- ✅ Detects GitPython 3.1.30 - CVE-2023-40267
- ✅ Detects aiohttp 3.8.5 - CVE-2024-23334
- **Result**: 3 vulnerabilities found ✅

**INCORRECT (uses pyproject.toml Python ^3.8)**:
- ❌ numpy 1.26.0 installation FAILS (incompatible with Python 3.8)
- ❌ Skips pillow (marker not satisfied)
- ❌ Skips GitPython (marker not satisfied)
- ❌ Skips aiohttp (marker not satisfied)
- **Result**: 0 vulnerabilities found 🔴 FALSE NEGATIVE