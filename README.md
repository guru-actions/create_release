# CloudBees Unify Release Action

A CloudBees custom action that automates the creation and execution of CloudBees Unify application releases.

This action:
1. Builds a release manifest from the most recent component artifacts
2. Creates a CloudBees Unify application release
3. Starts the release workflow
4. Waits for the release to complete (with polling)
5. Reports the final result
6. Optionally closes the release based on success/failure status

## Usage

```yaml
- name: Create and run CloudBees Unify release
  uses: https://github.com/guru-actions/create_release@main
  with:
    cb_api_token: ${{ secrets.CB_API_TOKEN }}
    cb_org_id: "8d16bd77-d506-45e6-a0f4-8884ce16d301"
    cb_application_id: "e3925a1f-9165-4767-9d6e-a96bb4edc80a"
    cb_workflow_id: "af8a9fae-4e6b-4c79-90bf-d39ede9e59a7"
    cb_environment: "squid-demo-3"
    release_name_prefix: "my-release"
```

## Inputs

### Required Inputs

| Input | Description |
|-------|-------------|
| `cb_api_token` | CloudBees API token for authentication |
| `cb_org_id` | CloudBees organization ID |
| `cb_application_id` | CloudBees application ID |
| `cb_workflow_id` | CloudBees workflow/automation ID for the release |
| `cb_environment` | Target environment name for the release (e.g., `squid-demo-3`, `squid-prod`) |

### Optional Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `cb_base_url` | CloudBees API base URL | `https://api.cloudbees.io` |
| `cb_artifact_labels` | Optional Unify artifact labels to filter artifacts (comma-separated, supports key=value pairs, e.g., `prod=true,stable=true` or `demo,v2`) | `""` (empty, no filtering) |
| `allow_latest_version` | Allow artifacts with version "latest" to be selected (by default, "latest" versions are excluded) | `false` |
| `component_overrides` | Optional component version overrides (comma-separated `componentId=version` pairs, e.g., `097aaa38-4753-4471-97f4-8e7265bd7bdc8=1.44,abc123=2.0`) | `""` (no overrides) |
| `override_component_id` | ⚠️ **Deprecated:** Optional component ID to pin to a specific version (use `component_overrides` instead) | `""` (no override) |
| `override_version` | ⚠️ **Deprecated:** Optional version to use for overridden component (use `component_overrides` instead) | `""` (no override) |
| `release_name_prefix` | Prefix for auto-generated release name | `unify-release` |
| `max_wait_attempts` | Maximum polling attempts to wait for release completion | `60` |
| `wait_sleep_seconds` | Seconds to sleep between polling attempts | `10` |
| `close_on_pass` | Automatically close the release if it succeeds | `false` |
| `close_on_fail` | Automatically close the release if it fails | `false` |
| `skip_release_on_missing_artifacts` | Skip release creation if any linked components don't have matching artifacts. When `true` and components are missing: action succeeds but no release is created, status output is set to `SKIPPED_MISSING_ARTIFACTS`. When `false` (default): release is created with available components only. | `false` |
| `selection_report_format` | Format for the `selection_report` output. Options: `text` (default) or `markdown`. When set to `markdown`, the report is formatted with markdown syntax for use with `cloudbees-io/publish-evidence-item@v1` or similar tools. | `text` |

## Outputs

| Output | Description |
|--------|-------------|
| `manifest` | Generated manifest JSON with component artifact versions |
| `selection_report` | Human-readable report showing component selection and overrides |
| `release_id` | Created release ID |
| `release_name` | Created release name (e.g., `unify-release-20231208-143022`) |
| `run_id` | Automation run ID from the release execution |
| `status` | Final release status (`SUCCEEDED`, `FAILED`, `TIMEOUT`, `SKIPPED_MISSING_ARTIFACTS`) |

## Examples

### Basic Usage

```yaml
jobs:
  deploy:
    steps:
      - name: Create release
        uses: https://github.com/guru-actions/create_release@main
        with:
          cb_api_token: ${{ secrets.CB_API_TOKEN }}
          cb_org_id: ${{ vars.CB_ORG_ID }}
          cb_application_id: ${{ vars.CB_APPLICATION_ID }}
          cb_workflow_id: ${{ vars.CB_WORKFLOW_ID }}
          cb_environment: "squid-demo-3"
```

### With Multiple Component Version Overrides

Pin multiple components to specific versions using comma-separated `componentId=version` pairs:

```yaml
- name: Create release with multiple component overrides
  uses: https://github.com/guru-actions/create_release@main
  with:
    cb_api_token: ${{ secrets.CB_API_TOKEN }}
    cb_org_id: ${{ vars.CB_ORG_ID }}
    cb_application_id: ${{ vars.CB_APPLICATION_ID }}
    cb_workflow_id: ${{ vars.CB_WORKFLOW_ID }}
    cb_environment: "squid-preprod"
    component_overrides: "097aaa38-4753-4471-97f4-8e7265bd7bdc8=1.44,abc123-def456-789=2.0,xyz789-012=3.5"
```

### With Single Component Version Override (Legacy)

⚠️ **Deprecated:** Use `component_overrides` instead for better flexibility.

Pin a single component to a specific version:

```yaml
- name: Create release with component override (legacy)
  uses: https://github.com/guru-actions/create_release@main
  with:
    cb_api_token: ${{ secrets.CB_API_TOKEN }}
    cb_org_id: ${{ vars.CB_ORG_ID }}
    cb_application_id: ${{ vars.CB_APPLICATION_ID }}
    cb_workflow_id: ${{ vars.CB_WORKFLOW_ID }}
    cb_environment: "squid-preprod"
    override_component_id: "097aaa38-4753-4471-97f4-8e7265bd7bdc8"
    override_version: "1.44"
```

### With Artifact Label Filtering

Filter artifacts by specific labels. Supports multiple labels (comma-separated) and key=value pairs:

**Single label:**
```yaml
- name: Create production release
  uses: https://github.com/guru-actions/create_release@main
  with:
    cb_api_token: ${{ secrets.CB_API_TOKEN }}
    cb_org_id: ${{ vars.CB_ORG_ID }}
    cb_application_id: ${{ vars.CB_APPLICATION_ID }}
    cb_workflow_id: ${{ vars.CB_WORKFLOW_ID }}
    cb_environment: "squid-prod"
    cb_artifact_labels: "prod"
    release_name_prefix: "prod-release"
```

**Multiple labels with key=value pairs:**
```yaml
- name: Create release with multiple label filters
  uses: https://github.com/guru-actions/create_release@main
  with:
    cb_api_token: ${{ secrets.CB_API_TOKEN }}
    cb_org_id: ${{ vars.CB_ORG_ID }}
    cb_application_id: ${{ vars.CB_APPLICATION_ID }}
    cb_workflow_id: ${{ vars.CB_WORKFLOW_ID }}
    cb_environment: "squid-prod"
    cb_artifact_labels: "prod=true,stable=true,region=us-east"
    release_name_prefix: "prod-release"
```

**Mixed labels:**
```yaml
- name: Create release with mixed labels
  uses: https://github.com/guru-actions/create_release@main
  with:
    cb_api_token: ${{ secrets.CB_API_TOKEN }}
    cb_org_id: ${{ vars.CB_ORG_ID }}
    cb_application_id: ${{ vars.CB_APPLICATION_ID }}
    cb_workflow_id: ${{ vars.CB_WORKFLOW_ID }}
    cb_environment: "squid-demo"
    cb_artifact_labels: "demo,dev=true,v2"
```

### With "latest" Versions Allowed

By default, the action excludes artifacts with version "latest". To include them:

```yaml
- name: Create release allowing latest versions
  uses: https://github.com/guru-actions/create_release@main
  with:
    cb_api_token: ${{ secrets.CB_API_TOKEN }}
    cb_org_id: ${{ vars.CB_ORG_ID }}
    cb_application_id: ${{ vars.CB_APPLICATION_ID }}
    cb_workflow_id: ${{ vars.CB_WORKFLOW_ID }}
    cb_environment: "squid-dev"
    allow_latest_version: "true"
```

### With Custom Wait Times

Adjust polling behavior for longer-running releases:

```yaml
- name: Create release with extended wait time
  uses: https://github.com/guru-actions/create_release@main
  with:
    cb_api_token: ${{ secrets.CB_API_TOKEN }}
    cb_org_id: ${{ vars.CB_ORG_ID }}
    cb_application_id: ${{ vars.CB_APPLICATION_ID }}
    cb_workflow_id: ${{ vars.CB_WORKFLOW_ID }}
    cb_environment: "squid-prod"
    max_wait_attempts: "120"  # Wait up to 20 minutes (120 * 10s)
    wait_sleep_seconds: "10"
```

### With Auto-Close on Success

Automatically close releases after successful deployment:

```yaml
- name: Create and close release on success
  uses: https://github.com/guru-actions/create_release@main
  with:
    cb_api_token: ${{ secrets.CB_API_TOKEN }}
    cb_org_id: ${{ vars.CB_ORG_ID }}
    cb_application_id: ${{ vars.CB_APPLICATION_ID }}
    cb_workflow_id: ${{ vars.CB_WORKFLOW_ID }}
    cb_environment: "squid-prod"
    close_on_pass: "true"  # Auto-close successful releases
    close_on_fail: "false" # Keep failed releases open for investigation
```

### With Auto-Close on All Outcomes

Automatically close releases regardless of success or failure:

```yaml
- name: Create release and always close
  uses: https://github.com/guru-actions/create_release@main
  with:
    cb_api_token: ${{ secrets.CB_API_TOKEN }}
    cb_org_id: ${{ vars.CB_ORG_ID }}
    cb_application_id: ${{ vars.CB_APPLICATION_ID }}
    cb_workflow_id: ${{ vars.CB_WORKFLOW_ID }}
    cb_environment: "squid-dev"
    close_on_pass: "true"  # Auto-close on success
    close_on_fail: "true"  # Auto-close on failure
```

### With Skip on Missing Artifacts

Skip release creation if any components don't have matching artifacts (useful for strict deployment pipelines):

```yaml
- name: Create release with strict artifact requirements
  id: release
  uses: https://github.com/guru-actions/create_release@main
  with:
    cb_api_token: ${{ secrets.CB_API_TOKEN }}
    cb_org_id: ${{ vars.CB_ORG_ID }}
    cb_application_id: ${{ vars.CB_APPLICATION_ID }}
    cb_workflow_id: ${{ vars.CB_WORKFLOW_ID }}
    cb_environment: "squid-prod"
    cb_artifact_labels: "prod=true"
    skip_release_on_missing_artifacts: "true"  # Skip if any component missing

- name: Check if release was skipped
  run: |
    if [ "${{ steps.release.outputs.status }}" = "SKIPPED_MISSING_ARTIFACTS" ]; then
      echo "⚠️  Release was skipped due to missing component artifacts"
      echo "This is expected behavior - action succeeded without creating release"
    else
      echo "✅ Release completed with status: ${{ steps.release.outputs.status }}"
    fi
```

**Note:** When `skip_release_on_missing_artifacts` is enabled:
- The action succeeds (exit code 0) even when skipping
- The `status` output is set to `SKIPPED_MISSING_ARTIFACTS`
- The selection report still shows which components were found/missing
- No release is created in CloudBees Unify

### Using Outputs

Access the outputs from the action:

```yaml
- name: Create release
  id: release
  uses: https://github.com/guru-actions/create_release@main
  with:
    cb_api_token: ${{ secrets.CB_API_TOKEN }}
    cb_org_id: ${{ vars.CB_ORG_ID }}
    cb_application_id: ${{ vars.CB_APPLICATION_ID }}
    cb_workflow_id: ${{ vars.CB_WORKFLOW_ID }}
    cb_environment: "squid-demo-3"

- name: Report results
  run: |
    echo "Release Name: ${{ steps.release.outputs.release_name }}"
    echo "Release ID: ${{ steps.release.outputs.release_id }}"
    echo "Run ID: ${{ steps.release.outputs.run_id }}"
    echo "Status: ${{ steps.release.outputs.status }}"
    echo "Manifest: ${{ steps.release.outputs.manifest }}"
    echo ""
    echo "Component Selection Report:"
    echo "${{ steps.release.outputs.selection_report }}"
```

### Using Selection Report for Evidence

The `selection_report` output provides a formatted report perfect for audit trails:

```yaml
- name: Create release
  id: release
  uses: https://github.com/guru-actions/create_release@main
  with:
    cb_api_token: ${{ secrets.CB_API_TOKEN }}
    cb_org_id: ${{ vars.CB_ORG_ID }}
    cb_application_id: ${{ vars.CB_APPLICATION_ID }}
    cb_workflow_id: ${{ vars.CB_WORKFLOW_ID }}
    cb_environment: "prod"
    component_overrides: "comp1=1.0,comp2=2.0"
    cb_artifact_labels: "prod=true,stable=true"

- name: Save evidence report
  run: |
    cat > release-evidence.txt << 'EOF'
    ${{ steps.release.outputs.selection_report }}
    EOF
    cat release-evidence.txt
```

**Example Report Output (Text Format):**
```
Component Selection Report
==========================

Configuration:
- Exclude 'latest' versions: false
- Artifact label filters: prod=true,stable=true
- Component overrides: comp1=1.0,comp2=2.0
- Legacy override: none=none

Components Selected:
- gururepservice/squid-ui                           2025.11.20.1-67d3957ed97f
- gururepservice/octopus-payments                   2025.12.11.1-e1e0acae2a62 (OVERRIDDEN)
- gururepservice/urchin-analytics                   2025.11.10.1-3c8efeb1e313
- gururepservice/nautilus-inventory                 2025.12.11.1-27b9b22d267c (OVERRIDDEN)
- codlocker-assets                                  1.39
```

### Using Markdown Format with CloudBees Evidence Item

You can format the selection report as markdown for use with `cloudbees-io/publish-evidence-item@v1`:

```yaml
- name: Create release
  id: release
  uses: https://github.com/guru-actions/create_release@main
  with:
    cb_api_token: ${{ secrets.CB_API_TOKEN }}
    cb_org_id: ${{ vars.CB_ORG_ID }}
    cb_application_id: ${{ vars.CB_APPLICATION_ID }}
    cb_workflow_id: ${{ vars.CB_WORKFLOW_ID }}
    cb_environment: "squid-demo-3"
    selection_report_format: "markdown"  # Enable markdown formatting

- name: Publish release evidence
  kind: deploy
  uses: cloudbees-io/publish-evidence-item@v1
  with:
    format: MARKDOWN
    content: ${{ steps.release.outputs.selection_report }}
```

**Example Markdown Report Output:**
```markdown
## Component Selection Report

### Configuration

- **Exclude 'latest' versions:** `false`
- **Artifact label filters:** `prod=true,stable=true`
- **Component overrides:** `comp1=1.0,comp2=2.0`
- **Legacy override:** `none=none`

### Components Selected

- **gururepservice/squid-ui**: `2025.11.20.1-67d3957ed97f`
- **gururepservice/octopus-payments**: `2025.12.11.1-e1e0acae2a62` _(OVERRIDDEN)_
- **gururepservice/urchin-analytics**: `2025.11.10.1-3c8efeb1e313`
- **gururepservice/nautilus-inventory**: `2025.12.11.1-27b9b22d267c` _(OVERRIDDEN)_
- **codlocker-assets**: `1.39`

### Summary

| Metric | Count |
|--------|-------|
| Total linked components | 5 |
| Components with artifacts | 5 |
| Components without artifacts | 0 |
```

## How It Works

### 1. Build Manifest

The action fetches the application's linked components and retrieves the most recent non-"latest" artifact for each component. It generates a manifest JSON structure:

```json
{
  "artifactVersions": [
    {
      "componentId": "...",
      "artifactName": "...",
      "version": "1.42"
    }
  ]
}
```

### 2. Create Release

POSTs to the CloudBees Unify API to create a new release with:
- Auto-generated release name (e.g., `unify-release-20231208-143022`)
- The manifest from step 1
- The target environment as an input parameter

### 3. Start Release

Triggers the release workflow to begin execution.

### 4. Wait for Completion

Polls the release status every 10 seconds (configurable) until:
- **Success**: The automation run completes with `SUCCEEDED` status
- **Failure**: The automation run completes with `FAILED`, `ERROR`, `CANCELLED` status
- **Timeout**: Maximum wait attempts reached without completion

### 5. Report Result

Reports the final status and exits:
- Exit code `0` for `SUCCEEDED`
- Exit code `1` for `FAILED` or `TIMEOUT`

### 6. Close Release (Optional)

Optionally closes the release based on configuration:
- If `close_on_pass: "true"` and release succeeded, closes the release
- If `close_on_fail: "true"` and release failed/timed out, closes the release
- By default, releases remain open for manual inspection

## Requirements

- CloudBees API token with permissions to:
  - Read application and component information
  - Create and run releases
  - Query automation run status

## License

MIT