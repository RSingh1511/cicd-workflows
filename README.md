# Reusable GitHub Actions Workflows

This repository contains **reusable GitHub Actions workflows** designed to standardize CI/CD processes across multiple repositories
in `SCORE`.  
These workflows integrate with **Bazel** and provide a consistent way to run **documentation builds, license checks, static analysis, tests, formatting checks and copyright verification**

## Available Workflows

| Workflow                | Description                                                        |
|-------------------------|--------------------------------------------------------------------|
| **Documentation Build** | Builds project documentation and deploys it to GitHub Pages         |
| **Documentation Cleanup** | Cleans up old documentation versions from the `gh-pages` branch   |
| **License Check**       | Verifies OSS licenses and compliance                               |
| **Static Code Analysis**| Runs Clang-Tidy, Clippy, Pylint, and other linters                 |
| **Tests**               | Executes tests using GoogleTest, Rust test, or pytest              |
| **Rust Coverage**       | Computes Rust code coverage and uploads HTML reports              |
| **C++ Coverage**        | Computes C++ code coverage using LCOV and uploads HTML reports     |
| **Formatting Check**    | Verifies code formatting using Bazel-based tools                   |
| **Copyright Check**     | Ensures all source files have the required copyright headers        |
| **Required Approvals**     | Enforces stricter CODEOWNERS rules for multi-team approvals         |
| **QNX Build (Gated)**   | Builds QNX Bazel targets with environment-gated secrets for forks   |

---

## Using the Workflows in Your Repository

To use a reusable workflow, create a workflow file inside **your repository** (e.g., `.github/workflows/ci.yml`) and reference the appropriate workflow from this repository.

### **1️ Documentation Build Workflow**
**Usage Example**
```yaml 
name: Documentation CI

on:
  pull_request:
  push:
    branches:
      - main

jobs:
  docs:
    uses: eclipse-score/cicd-workflows/.github/workflows/docs.yml@main
    with:
      retention-days: 3
      # Optionally override:
      # workflow-version: main
      # bazel-target: "//docs:github-pages"
```
This workflow:

✅ Builds project documentation  
✅ Uploads it as an artifact  
✅ Deploys it to **GitHub Pages** on push to `main`  

---

### **2️ Documentation Cleanup Workflow**
**Usage Example**
```yaml
name: Documentation Cleanup

on:
  schedule:
    - cron: '0 2 * * *' # every day at 2am UTC

jobs:
  docs-cleanup:
    uses: eclipse-score/cicd-workflows/.github/workflows/docs-cleanup.yml@main
    with:
      workflow-version: main
    secrets:
      token: ${{ secrets.GITHUB_TOKEN }}
```

This workflow:

✅ Cleans up old documentation versions from the `gh-pages` branch  
✅ Runs daily at 2am UTC  

---

### **3️ License Check Workflow**
**Usage Example**
```yaml
name: License Check CI

on:
  pull_request:
  push:
    branches:
      - main

jobs:
  license-check:
    uses: eclipse-score/cicd-workflows/.github/workflows/license-check.yml@main
    with:
      repo-url: "${{ github.server_url }}/${{ github.repository }}" # optional, this is the default
      bazel-target: "run //:license-check" # optional, this is the default
    secrets:
      dash-api-token: ${{ secrets.ECLIPSE_GITLAB_API_TOKEN }} # mandatory - the Eclispe DASH API token 
```

This workflow:

✅ Runs **DASH license compliance checks** for **Rust, C++, and Python**  
✅ Uses the **organization secret** `ECLIPSE_GITLAB_API_TOKEN`  
✅ Comments results directly on the **Pull Request**

> ℹ️ **Note:** You can override the Bazel command using the `bazel-target` input.  
> **Default:** `run //:license-check`

---

### **4️ Static Code Analysis Workflow**
**Usage Example**
```yaml
name: Static Analysis CI

on:
  pull_request:
  push:
    branches:
      - main

jobs:
  static-analysis:
    uses: eclipse-score/cicd-workflows/.github/workflows/static-analysis.yml@main
    with:
      bazel-targets: "//..."            # optional, default
      bazel-config: "lint"             # optional, default
      bazel-args: "--@aspect_rules_lint//lint:fail_on_violation=true"  # optional
```

This workflow:  
✅ Runs **Clippy** via Bazel on the selected targets  
✅ Publishes **Clippy reports** as an artifact  
✅ Fails the job if Bazel fails or if any Clippy report is non-empty  
✅ Writes a summary to the GitHub job summary  

Inputs:
- `bazel-targets`: Bazel targets to build (default: `//...`)
- `bazel-config`: Bazel config to apply (default: `lint`, set empty to disable)
- `bazel-args`: Extra Bazel args (e.g., `--@aspect_rules_lint//lint:fail_on_violation=true`)

---

### **5️ Tests Workflow**
**Usage Example**
```yaml
name: Test CI

on:
  pull_request:
  push:
    branches:
      - main

jobs:
  tests:
    uses: eclipse-score/cicd-workflows/.github/workflows/tests.yml@main
```

This workflow:  
✅ Runs **GoogleTest** for C++  
✅ Runs **Rust Unit Tests**  
✅ Runs **pytest** for Python  

---

### **6️ Rust Coverage Workflow**
**Usage Example**
```yaml
name: Rust Coverage CI

on:
  pull_request:
    types: [opened, reopened, synchronize]
  workflow_dispatch:

jobs:
  rust-coverage:
    uses: eclipse-score/cicd-workflows/.github/workflows/rust-coverage.yml@main
    with:
      bazel-test-targets: "//src/rust/..."
      bazel-test-config-flags: "--config=per-x86_64-linux --config=ferrocene-coverage"
      bazel-test-args: "--nocache_test_results"
      coverage-target: "//:rust_coverage"
      min-coverage: 90
      coverage-artifact-name: "rust-coverage-html"
```

This workflow:  
✅ Runs **Rust tests** with coverage instrumentation  
✅ Generates **coverage reports** via Bazel  
✅ Uploads the **HTML coverage report** as an artifact  

---

### **7️ C++ Coverage Workflow**
**Usage Example**
```yaml
name: C++ Coverage CI

on:
  pull_request:
    types: [opened, reopened, synchronize]
  push:
    branches:
      - main
  merge_group:
    types: [checks_requested]

jobs:
  coverage-report:
    uses: eclipse-score/cicd-workflows/.github/workflows/cpp-coverage.yml@main
    with:
      bazel-target: "//..."
```

---

### **8️⃣ Formatting Check Workflow**
**Usage Example**
```yaml
name: Formatting Check CI

on:
  pull_request:
  merge_group:
    types: [checks_requested]

jobs:
  formatting-check:
    uses: eclipse-score/cicd-workflows/.github/workflows/format-check.yml@main
    with:
      bazel-target: "test //:format.check" # optional, this is the default
```

This workflow:  
✅ Runs a **Bazel-based formatting check** (e.g., `buildifier`, `clang-format`, etc.)  
✅ Can be integrated into Pull Requests and Merge Queues  
✅ Ensures code adheres to formatting rules before merge

> ℹ️ **Note:** You can override the Bazel command using the `bazel-target` input.  
> **Default:** `test //:format.check`

---

### **9️⃣ Copyright Check Workflow**
**Usage Example**
```yaml
name: Copyright Check CI

on:
  pull_request:
  push:
    branches:
      - main

jobs:
  copyright-check:
    uses: eclipse-score/cicd-workflows/.github/workflows/copyright.yml@main
    with:
      bazel-target: "run //:copyright-check" # optional, this is the default
```

This workflow:  
✅ Runs a **Bazel-based copyright**
✅ Ensures all source files have **Eclipse Foundation** headers

> ℹ️ **Note:** You can override the Bazel command using the `bazel-target` input.  
> **Default:** `run //:copyright-check`

---
### **10️ Required Approvals Workflow**

This workflow enforces **stricter CODEOWNERS checks** than GitHub’s defaults.  
Normally, GitHub requires approval from *any one* codeowner when multiple are listed.  
With this workflow, you can enforce that **all required teams approve** (or set a minimum count).

**Usage Example**

```yaml
name: Enforce Approvals

on:
  pull_request:
    types: [opened, reopened, synchronize, ready_for_review]
  pull_request_review:
    types: [submitted, edited, dismissed]

jobs:
  enforce:
    uses: eclipse-score/cicd-workflows/.github/workflows/required-approvals.yml@main
    with:
      pat_secret: SCORE_BOT_PAT
      # Optional overrides:
      # min_approvals: 2
      # approval_mode: ALL
      # org_name: qorix-group
```

**Defaults**  
- `org_name`: `score`  
- `min_approvals`: `1`  
- `approval_mode`: `ALL`  
- `require_all_approvals_latest_commit`: always `true`  

**Key Features**  
✅ Enforces that *all relevant CODEOWNERS* approve (`ALL` mode)  
✅ Invalidates approvals on new commits (`require_all_approvals_latest_commit`)  
✅ Works with **org secrets** (e.g. `SCORE_BOT_PAT`) that must have `repo` + `read:org` scopes  
✅ Compatible with branch protection rules → can be marked as **required**  

---


### **11️ QNX Build (Gated) Workflow**

Use this workflow when you need QNX secrets for forked PRs and want a manual approval gate via an environment.

**Usage Example**

```yaml
name: Scrample QNX (gated)

on:
  pull_request_target:
    types: [opened, reopened, synchronize]

jobs:
  qnx-build:
    uses: eclipse-score/cicd-workflows/.github/workflows/qnx-build.yml@main
    permissions:
      contents: read
      pull-requests: read
    with:
      bazel-target: "//..." # optional, default shown
      bazel-config: "x86_64-qnx"     # optional, default shown
      credential-helper: ".github/tools/qnx_credential_helper.py" # optional, default shown
      environment-name: "workflow-approval" # optional, default shown
    secrets:
      score-qnx-license: ${{ secrets.SCORE_QNX_LICENSE }}
      score-qnx-user: ${{ secrets.SCORE_QNX_USER }}
      score-qnx-password: ${{ secrets.SCORE_QNX_PASSWORD }}
```

**Notes**
- Runs on `pull_request_target` so maintainers can approve the `workflow-approval` environment before secrets are used.
- Installs the QNX license, builds with the configured Bazel target/config, and cleans up the license directory.

---


##  How to Update Workflows
Since these workflows are centralized, updates in the `cicd-workflows` repository will **automatically apply to all repositories using them**. If you need a specific version, reference a **tagged release** instead of `main`:
```yaml
uses: eclipse-score/cicd-workflows/.github/workflows/tests.yml@v1.0.0
```

---

## ⚡️ Bazel Setup with Shared Caching

To improve performance and reduce redundant downloads across workflow runs, the reusable workflows configure Bazel with shared caching:

```yaml
- name: Setup Bazel with shared caching
  uses: bazel-contrib/setup-bazel@0.18.0
  with:
    disk-cache: true
    repository-cache: true
    bazelisk-cache: true
    cache-save: ${{ github.event_name == 'push' }}
```

### Benefits

- **`disk-cache`**: Stores compiled Bazel outputs across jobs.
- **`repository-cache`**: Caches external dependencies (e.g., modules, WORKSPACE fetches).
- **`bazelisk-cache`**: Avoids re-downloading Bazel binaries.
- **`cache-save`**: Saves the cache only on push events to avoid unnecessary cache updates (e.g., from pull requests).

This setup significantly reduces CI build time and improves reuse across different workflows.


### **Summary**
✅ **Standardized** CI/CD workflows across all projects  
✅ **Reusable & Maintainable** with centralized updates  
✅ **Bazel-powered** for consistent testing & analysis

## 🏃‍♂️ Runner Selection Logic

All workflows in this repository use the following logic for selecting the runner:

```yaml
runs-on: ${{ vars.REPO_RUNNER_LABELS && fromJSON(vars.REPO_RUNNER_LABELS) || 'ubuntu-latest' }}
```

This means:

- If your repository defines a variable named `REPO_RUNNER_LABELS` (e.g., in repository or organization settings), its value will be used as the runner label(s).  
  This allows you to use **self-hosted runners** or any custom runner configuration.
- If `REPO_RUNNER_LABELS` is **not set**, the workflow will default to GitHub-hosted `ubuntu-latest`.

**Why?**  
This approach allows forked repositories or projects with special requirements to use their own runners, while everyone else gets a reliable default.

> ℹ️ **Tip:** To use a self-hosted runner, set the `REPO_RUNNER_LABELS` variable in your repository or organization settings to the label(s) of your runner.
