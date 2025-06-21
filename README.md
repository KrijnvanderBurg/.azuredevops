# Azure DevOps Pipeline Templates for Flint

This repository contains reusable Azure DevOps pipeline templates for the Flint PySpark ETL framework.

## Table of Contents

1. [Structure Overview](#structure-overview)
2. [Getting Started](#getting-started)
3. [Template Hierarchy](#template-hierarchy)
4. [Python Template Examples](#python-template-examples)
5. [Complete Template Reference](#complete-template-reference)
6. [Best Practices](#best-practices)
7. [Versioning Strategy](#versioning-strategy)
8. [Integration with Flint](#integration-with-flint)
9. [Contributing Guidelines](#contributing-guidelines)

## Structure Overview

```
.azuredevops/
├── v1/                           # Current version of pipeline templates
│   ├── containers/               # Container definitions for build/test environments
│   ├── pipelines/                # Complete pipeline definitions
│   ├── pools/                    # Agent pool configurations
│   ├── scripts/                  # Helper scripts used by pipelines
│   ├── templates/                # Reusable pipeline templates
│   │   ├── atomic/               # Atomic templates (single-purpose)
│   │   │   ├── azuredevops/      # Azure DevOps specific templates
│   │   │   ├── git/              # Git operation templates
│   │   │   ├── github/           # GitHub integration templates
│   │   │   ├── opentofu/         # OpenTofu (Terraform) templates
│   │   │   └── python/           # Python tooling templates
│   │   └── composite/               # Composite templates combining atomic templates
│   │       ├── opentofu/         # Composite OpenTofu templates
│   │       └── python/           # Composite Python templates
│   └── variables/                # Pipeline variables and variable groups
│       ├── environments/         # Environment-specific variables
│       ├── groups/               # Variable groups
│       └── shared.yaml            # Variables shared across pipelines
└── v2/                           # Next generation templates (in development)
```

## Getting Started

### Setting Up a New Pipeline

#### 1. Add the Template Repository as a Git Submodule

If you're starting a new project, add this templates repository as a Git submodule:

```bash
git submodule add https://github.com/your-org/dp-azuredevops.git .azuredevops
```

#### 2. Create a Basic Pipeline File

Create an `azure-pipelines.yaml` file in your project root with the following contents:

```yaml
trigger:
  - main

extends:
  template: .azuredevops/v1/pipelines/python.yaml
  parameters:
    projectName: 'your-project-name'
    pythonVersion: '3.11'
```

#### 3. Customize the Pipeline

You can customize the pipeline by adding parameters:

```yaml
trigger:
  - main

extends:
  template: .azuredevops/v1/pipelines/python.yaml
  parameters:
    projectName: 'flint'
    pythonVersion: '3.11'
```

## Template Hierarchy

- **Level 0 Templates**: Atomic, single-purpose templates that perform one specific task
- **Level 1 Templates**: Composite templates that combine multiple Level 0 templates for common workflows

### Using Level 0 Templates

Level 0 templates are atomic, single-purpose templates that perform one specific task. Use these when you need fine-grained control over your pipeline.

Example: Installing dependencies with Poetry

```yaml
- template: .azuredevops/v1/templates/atomic/python/poetry_install.yaml
  parameters:
    pythonSrcDirectory: $(Build.SourcesDirectory)
    pythonPyprojectFilepath: $(Build.SourcesDirectory)/pyproject.toml
```

### Using Level 1 Templates

Level 1 templates combine multiple Level 0 templates to perform a composite operation. Use these to simplify your pipeline by grouping related tasks.

Example: Running all code formatters

```yaml
- template: .azuredevops/v1/templates/composite/python/formatter.yaml
  parameters:
    pythonSrcDirectory: $(Build.SourcesDirectory)
    pythonSourceDirectories: 'src tests'
    enabled: true
    continueOnError: false
```

## Python Template Examples

**Basic Python Build Pipeline:**

```yaml
# azure-pipelines.yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

extends:
  template: .azuredevops/v1/pipelines/python.yaml
  parameters:
    projectName: 'flint'
    pythonVersion: '3.11'
```

**Creating Custom Pipelines:**

```yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

stages:
- stage: Build
  jobs:
  - job: Format
    steps:
    - checkout: self
    
    - task: UsePythonVersion@0
      inputs:
        versionSpec: '3.11'
        addToPath: true
    
    - template: .azuredevops/v1/templates/atomic/python/poetry_install.yaml
      parameters:
        pythonSrcDirectory: $(Build.SourcesDirectory)
        pythonPyprojectFilepath: $(Build.SourcesDirectory)/pyproject.toml
    
    - template: .azuredevops/v1/templates/composite/python/formatter.yaml
      parameters:
        pythonSrcDirectory: $(Build.SourcesDirectory)
        pythonSourceDirectories: 'src tests'
```

## Complete Template Reference

### Common Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| pythonSrcDirectory | Root directory of the Python project | $(Build.SourcesDirectory) |
| pythonPyprojectFilepath | Path to pyproject.toml file | $(Build.SourcesDirectory)/pyproject.toml |
| pythonSourceDirectories | Space-separated list of source directories | src tests |
| enabled | Whether to run the template | true |
| continueOnError | Whether to continue on error | false |

### Level 0 Python Templates

| Template | Description | Required Parameters | Optional Parameters |
|----------|-------------|---------------------|---------------------|
| `poetry_install.yaml` | Installs Python dependencies using Poetry | `pythonSrcDirectory`, `pythonPyprojectFilepath` | `poetryHome`, `poetryCacheDirectory` |
| `pytest_test.yaml` | Runs pytest tests with coverage | `pythonTestsDirectory` | `publishCoverage`, `testResultsFormat` |
| `black_formatter.yaml` | Runs Black code formatter | `pythonSourceDirectories` | `continueOnError` |
| `ruff_formatter.yaml` | Runs Ruff code formatter | `pythonSourceDirectories` | `continueOnError` |
| `isort_formatter.yaml` | Runs isort import sorter | `pythonSourceDirectories` | `continueOnError` |
| `ruff_linter.yaml` | Runs Ruff linter | `pythonSourceDirectories` | `continueOnError` |
| `pylint_linter.yaml` | Runs Pylint linter | `pythonSourceDirectories` | `continueOnError` |
| `flake8_linter.yaml` | Runs Flake8 linter | `pythonSourceDirectories` | `continueOnError` |
| `mypy_typechecker.yaml` | Runs MyPy type checker | `pythonSourceDirectories` | `continueOnError` |
| `pyre_typechecker.yaml` | Runs Pyre type checker | `pythonSourceDirectories` | `continueOnError` |
| `pyright_typechecker.yaml` | Runs Pyright type checker | `pythonSourceDirectories` | `continueOnError` |
| `bandit_scanner.yaml` | Runs Bandit security scanner | `pythonSourceDirectories` | `continueOnError` |
| `semgrep_vuln_scanner.yaml` | Runs Semgrep vulnerability scanner | `pythonSourceDirectories` | `continueOnError` |
| `trufflehog_secret_scanner.yaml` | Runs TruffleHog secret scanner | `pythonSourceDirectories` | `continueOnError` |
| `vulture_deadcode_scanner.yaml` | Runs Vulture dead code scanner | `pythonSourceDirectories` | `continueOnError` |
| `ossaudit_scanner.yaml` | Runs OSSAudit dependency scanner | `pythonSrcDirectory` | `continueOnError` |
| `build_wheel_bdist.yaml` | Builds Python wheel | `pythonSrcDirectory`, `projectName` | `publishArtifacts` |
| `twine_upload.yaml` | Uploads package to PyPI | `pythonSrcDirectory` | `repository` |

### Level 1 Python Templates

| Template | Description | Included Level 0 Templates | Required Parameters |
|----------|-------------|---------------------------|---------------------|
| `formatter.yaml` | Runs all code formatters | `black_formatter.yaml`, `isort_formatter.yaml`, `ruff_formatter.yaml` | `pythonSrcDirectory`, `pythonSourceDirectories` |
| `linter.yaml` | Runs all code linters | `ruff_linter.yaml`, `pylint_linter.yaml`, `flake8_linter.yaml` | `pythonSrcDirectory`, `pythonSourceDirectories` |
| `typechecking.yaml` | Runs all type checkers | `mypy_typechecker.yaml`, `pyre_typechecker.yaml`, `pyright_typechecker.yaml` | `pythonSrcDirectory`, `pythonSourceDirectories` |
| `scanning_1st_vulnerabilities.yaml` | Scans first-party code for vulnerabilities | `bandit_scanner.yaml`, `semgrep_vuln_scanner.yaml`, `trufflehog_secret_scanner.yaml`, `vulture_deadcode_scanner.yaml`, `devskim_scanner.yaml`, `graudit_scanner.yaml` | `pythonSrcDirectory` |
| `scanning_3rd_vulnerabilities.yaml` | Scans third-party dependencies for vulnerabilities | `ossaudit_scanner.yaml`, `owasp_scanner.yaml` | `pythonSrcDirectory` |

## Best Practices

### General Pipeline Best Practices

#### 1. Use the Appropriate Template Level

- **Level 0 templates** are best for fine-grained control
- **Level 1 templates** are best for common workflows
- **Complete pipelines** are best for standard projects

#### 2. Parameter Naming Conventions

- Use consistent parameter names across your pipelines
- Follow the established parameter naming patterns:
  - `pythonSrcDirectory` for the source directory
  - `pythonTestsDirectory` for the test directory
  - `pythonSourceDirectories` for specifying source directories to analyze

#### 3. Resource Efficiency

- Use conditional execution to skip unnecessary steps
- Group related tasks to minimize pipeline overhead
- Consider using parallel jobs for independent tasks

### Python Project Best Practices

#### 1. Dependency Management

- Use Poetry for dependency management
- Keep your dependencies up-to-date
- Pin versions for production dependencies

```yaml
- template: .azuredevops/v1/templates/atomic/python/poetry_install.yaml
  parameters:
    pythonSrcDirectory: $(Build.SourcesDirectory)
    pythonPyprojectFilepath: $(Build.SourcesDirectory)/pyproject.toml
```

#### 2. Code Quality

- Run formatters before linters
- Run linters before tests
- Keep formatting and linting steps separate from testing

```yaml
- template: .azuredevops/v1/templates/composite/python/formatter.yaml
  parameters:
    pythonSrcDirectory: $(Build.SourcesDirectory)
    pythonSourceDirectories: 'src tests'

- template: .azuredevops/v1/templates/composite/python/linter.yaml
  parameters:
    pythonSrcDirectory: $(Build.SourcesDirectory)
    pythonSourceDirectories: 'src tests'
```

#### 3. Testing

- Run tests in a separate job from linting
- Publish test results and coverage reports
- Use the appropriate test runner for your project

```yaml
- template: .azuredevops/v1/templates/atomic/python/pytest_test.yaml
  parameters:
    pythonTestsDirectory: $(Build.SourcesDirectory)/tests
    publishCoverage: true
    enabled: true
```

#### 4. Security Scanning

- Run first-party and third-party vulnerability scans
- Consider setting `continueOnError: true` for security scans to avoid blocking the pipeline
- Review security findings as part of your regular process

```yaml
- template: .azuredevops/v1/templates/composite/python/scanning_1st_vulnerabilities.yaml
  parameters:
    pythonSrcDirectory: $(Build.SourcesDirectory)
    continueOnError: true
    enabled: true

- template: .azuredevops/v1/templates/composite/python/scanning_3rd_vulnerabilities.yaml
  parameters:
    pythonSrcDirectory: $(Build.SourcesDirectory)
    continueOnError: true
    enabled: true
```

### Pipeline Organization

#### 1. Stage Organization

Organize your pipeline into logical stages:

```yaml
stages:
- stage: Build
  displayName: 'Build and Validate'
  jobs:
  - job: Lint
    # Linting tasks
  
- stage: Test
  displayName: 'Test'
  jobs:
  - job: UnitTest
    # Unit testing tasks
  - job: IntegrationTest
    # Integration testing tasks

- stage: Security
  displayName: 'Security Scanning'
  jobs:
  - job: CodeScan
    # Code security scanning
  - job: DependencyScan
    # Dependency security scanning

- stage: Publish
  displayName: 'Publish Artifacts'
  jobs:
  - job: Package
    # Package and publish tasks
```

#### 2. Variable Management

- Use variable groups for environment-specific configuration
- Use pipeline variables for configuration that is specific to a pipeline
- Use template parameters for configuration that varies between template usages

```yaml
variables:
- group: python-project-variables

- name: pythonVersion
  value: '3.11'
```

## Versioning Strategy

### Current Versions

- **v1**: Current stable version of all templates
- **v2**: Next generation templates (under development)

### Version History

#### v1 (Current Stable)

The v1 templates provide a comprehensive set of pipeline templates for building, testing, and analyzing Python projects.

Key features:
- Two-level template hierarchy (atomic/composite)
- Support for Poetry dependency management
- Comprehensive test, lint, and security scanning
- Integration with Azure DevOps pipelines

#### v2 (In Development)

The v2 templates are a work in progress and will introduce the following improvements:

Planned features:
- Simplified parameter naming
- Better cross-platform support
- Container-based execution for consistent environments
- Improved template discoverability
- Enhanced documentation
- Support for additional Python package managers

### Transition Strategy

#### Using v1 Templates

All v1 templates are stable and can be used in production pipelines. To use v1 templates:

```yaml
# azure-pipelines.yaml
extends:
  template: .azuredevops/v1/pipelines/python.yaml
  parameters:
    projectName: 'your-project-name'
```
