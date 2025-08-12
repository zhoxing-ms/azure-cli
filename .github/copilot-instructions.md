# Azure CLI Copilot Development Instructions

## Repository Overview

**Microsoft Azure CLI** is the official command-line interface for Microsoft Azure cloud services. This is a large Python-based repository (~4 core modules + extensive tooling) that provides a comprehensive multi-platform CLI experience.

- **Primary Language**: Python 3.9-3.12 (3.12 recommended)
- **Repository Type**: Command-line interface with command modules
- **Target Platforms**: Windows, Linux, macOS
- **Package Formats**: MSI, DEB, RPM, Docker, Homebrew, PyPI wheels
- **Development Tool**: `azdev` (Azure Developer Tool) - **DO NOT use the deprecated `dev_setup.py`**

## Essential Development Setup

### Prerequisites
- Python 3.9-3.12 (3.12 preferred)
- Git
- Virtual environment support

### Quick Start (ALWAYS follow this order)
```bash
# 1. Create and activate virtual environment
python -m venv env
source env/bin/activate  # Linux/Mac
# env\Scripts\activate   # Windows

# 2. Upgrade pip first
python -m pip install --upgrade pip

# 3. Install azdev (may take 2-3 minutes)
pip install azdev

# 4. Setup development environment (may take 5-10 minutes)
azdev setup -c

# 5. Verify installation
az --version
```

**Network Issues**: If pip install fails with timeout errors, retry with `--timeout 300` or `--retries 10`. This is common in this repository.

## Build and Validation Commands

### Core Development Commands
```bash
# Install all modules from source (alternative to azdev setup)
./scripts/install_full.sh

# Style checking (fast, ~30 seconds)
azdev style

# Comprehensive linting (medium speed, 2-3 minutes)
azdev linter --ci-exclusions --min-severity medium

# Quick linter for specific changes
azdev linter --ci-exclusions --min-severity medium --repo ./ --src HEAD --tgt origin/dev

# Check command coverage for a module
azdev cmdcov {module_name}
azdev cmdcov {module_name} --level argument  # argument-level coverage
```

### Testing Commands
```bash
# Run tests for specific module (recommended for development)
azdev test {module_name}

# Run tests with verbose output and timing
azdev test {module_name} --verbose --pytest-args "--durations=10"

# Run tests in serial mode (for modules like appservice, botservice, cloud, network)
azdev test --series {module_name}

# Full test suite (WARNING: very time-consuming, 30+ minutes)
azdev test --profile latest

# Quick self-test to verify CLI functionality
az self-test
```

### Package Building
```bash
# Build Python wheels
./scripts/release/pypi/build.sh

# Build for specific platforms (Windows)
build_scripts/windows/scripts/build.cmd

# Install and test from packages
pip install --find-links ./dist azure-cli
```

## Project Layout and Architecture

### Core Module Structure
```
src/
├── azure-cli/           # Main CLI package
├── azure-cli-core/      # Core functionality and framework
├── azure-cli-telemetry/ # Telemetry collection
└── azure-cli-testsdk/   # Testing framework and utilities
```

### Key Directories
- `scripts/`: Build and utility scripts (install_full.sh, CI scripts)
- `build_scripts/`: Platform-specific build scripts (Windows, RPM, DEB)
- `doc/`: Development documentation and authoring guides
- `.azure-pipelines/`: Azure DevOps CI/CD configuration
- `.github/`: GitHub Actions and policies

### Configuration Files
- `.flake8`: Style checking configuration (max-line-length: 120)
- `pylintrc`: Linting rules and exclusions
- `linter_exclusions.yml`: Module-specific linter exemptions
- `requirements.txt`: Basic dependencies (setuptools, pip)
- `nose.cfg`: Test runner configuration

## Validation Pipeline

### Pre-commit Checks
The repository enforces several checks before code integration:
1. **Pull Request Format**: Title and content validation
2. **Style Check**: `azdev style` (enforced via GitHub Actions)
3. **Linting**: `azdev linter --ci-exclusions --min-severity medium`
4. **Unit Tests**: Module-specific test suites
5. **Command Coverage**: Ensure all commands have tests
6. **Secret Scanning**: `azdev scan` for credential detection

### CI/CD Pipeline Structure
- **Azure DevOps**: Primary build system with multi-platform matrix
- **GitHub Actions**: Style and linting validation for PRs
- **Package Validation**: MSI, DEB, RPM, Docker, Homebrew testing
- **Live Testing**: Nightly runs against actual Azure services

### Build Timing Expectations
- Style check: ~30 seconds
- Linting: 2-3 minutes
- Module tests: 5-15 minutes per module
- Full test suite: 30+ minutes
- Package builds: 10-20 minutes per platform

## Common Patterns and Conventions

### Command Module Development
- Each command module lives in `src/azure-cli/azure/cli/command_modules/`
- Use `azdev cmdcov` to ensure 100% command coverage
- Follow naming: `test_<module>_<feature>` for test methods
- Always include tests for new commands (PRs rejected without tests)

### Testing Best Practices
- Use `ScenarioTest` for integration tests with VCR.py recording
- Serial modules: `appservice`, `botservice`, `cloud`, `network`, `azure-cli-core`, `azure-cli-telemetry`
- 100% command coverage required (use `azdev cmdcov` to verify)
- Test boundary values: empty strings, null, 0, False

### Code Quality Standards
- Maximum line length: 120 characters
- Follow existing linter exclusions in `linter_exclusions.yml`
- Use `azdev style` before committing
- Address linter warnings with `--min-severity medium`

## Troubleshooting Common Issues

### Network/Installation Issues
- Pip timeouts: Use `pip install --timeout 300 --retries 10`
- Dependency conflicts: Create fresh virtual environment
- azdev setup failures: Ensure virtual environment is activated

### Build Issues
- Windows: Ensure proper Visual Studio components installed
- macOS: May need to relink Python in Homebrew
- Linux: Check for required development packages

### Test Issues
- Live test failures: Check Azure service availability
- Recording issues: Ensure proper test isolation
- Coverage issues: Use `azdev cmdcov` to identify gaps

## Specific Coding Guidelines for PR review

### 1. Rule of help messages
Firstly, find the help messages, their format is `...help='{help_message}'...` or `....help="{help_message}"...`, {help_message} is the placeholder
Next, you need to review the format of the found help messages according to the following rules:
1. If the help message does not start with a verb, skip this check. Otherwise, check the verb at the beginning of help message
2. Confirm if the verb is in the first person voice, such as the third person voice of 'Enables......' is not compliant
3. Confirm if the verb is in the active voice, such as the passive voice 'Enabled......' is not compliant
This rule only applys to file path: "src/azure-cli/azure/cli/command_modules/**/_params.py"

### 2. Confirmation for dangerous commands
Firstly, find the command signature, their format is `... g.custom_command('{command_signature}', ...)...` or `... g.command('{command_signature}', ...)...`, {command_signature} is the command signature
Next, you need to determine whether the command signature is a dangerous operation, such as `delete`, `remove`, `stop` operation which will break or remove resources. If so, You need to suggest adding the flag `confirmation=True`, such as `g.custom_command('{command_signature}', ..., )...`
This rule only applys to file path: "src/azure-cli/azure/cli/command_modules/**/commands.py"

### 3. Common code best practices
- ​​Use 4 spaces for indentation​​, never mix tabs and spaces
​​- Naming:
    (1) Variables/functions: snake_case
    (2) Classes: CamelCase
    (3) Constants: UPPER_CASE_WITH_UNDERSCORES
- ​​Surround operators with spaces​​ (x = y + 1), except in function parameters
- ​​Avoid extraneous whitespace​​ in brackets, before commas/colons
- ​​Handle exceptions explicitly​​ using try-except blocks
- ​​Break long expressions​​ using parentheses alignment or hanging indents
​​- Use blank lines​​: two between top-level definitions, one between methods

## Important Notes for Agents

1. **ALWAYS use azdev**: Never use the deprecated `dev_setup.py` script
2. **Virtual environment required**: All development must be in a virtual environment
3. **Trust these instructions**: Only search for additional information if these instructions are incomplete or incorrect
4. **Style before commit**: Always run `azdev style` before making changes
5. **Test coverage mandatory**: All new commands require tests
6. **Incremental testing**: Use module-specific tests during development, not full suite
7. **Network resilience**: Be prepared for pip timeout issues and retry with longer timeouts

This repository has complex build requirements but following these instructions will minimize exploration time and avoid common pitfalls. The development workflow is well-established through azdev tooling.
