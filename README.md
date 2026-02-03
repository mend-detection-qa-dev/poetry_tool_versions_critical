# Test Case: Poetry Tool-Versions Critical - DEPENDENCY FAILURE TEST

## Package Manager
Poetry

## Python Version Detection
- **Source**: .tool-versions (PRIORITY #2 - after .python-version)
- **Expected Version**: 3.11.0
- **Conflict**: pyproject.toml says ^3.8 (SHOULD BE IGNORED)

## Dependencies (CRITICAL)
- `tomli>=2.0.0` with marker `python_version >= "3.11"`
- `typing-extensions>=4.8.0`

## Test Purpose
**CRITICAL DEPENDENCY TEST WITH ENVIRONMENT MARKERS AND .tool-versions PRIORITY**

### Why This Test Matters
The `tomli` dependency has an environment marker: `python_version >= "3.11"`

If tool uses **3.11.0** from .tool-versions:
- ✅ Environment marker evaluates to TRUE
- ✅ tomli is installed
- ✅ All dependencies present
- ✅ Lock file was created with this version

If tool uses **3.8** from pyproject.toml:
- ❌ Environment marker evaluates to FALSE
- ❌ tomli is SKIPPED during installation
- ❌ **Missing dependency** - application will fail at runtime
- ❌ Lock file is inconsistent with resolved dependencies

### .tool-versions Priority Test
This specifically tests that tools properly detect and prioritize `.tool-versions` over `pyproject.toml`.
- .tool-versions is commonly used with asdf and mise version managers
- It should override the Python version in pyproject.toml
- The lock file was generated using Python 3.11.0 from .tool-versions

## Expected Behavior
1. Tool detects .tool-versions (3.11.0) as higher priority than pyproject.toml
2. Tool ignores pyproject.toml python = "^3.8"
3. Installs ALL dependencies including tomli (based on Python 3.11)
4. Dependency resolution succeeds with lock file consistency

## Real-World Impact
Environment markers combined with version files are commonly used for:
- Python version-specific dependencies
- Conditional feature dependencies
- Lock file generation with specific Python versions
- Team standardization on Python versions via .tool-versions

**Wrong version detection = silent dependency omission = runtime failures**

This is worse than installation failure - the installation "succeeds" but dependencies are missing!

## Installation Test
```bash
# Verify Python version detection
cat .tool-versions  # Should show 3.11.0

# Install dependencies
poetry install

# Verify tomli was installed (should succeed with Python 3.11)
poetry run python -c "import tomli; print('✅ tomli imported successfully')"
```

If the tool incorrectly uses Python 3.8 from pyproject.toml, the tomli import will fail.