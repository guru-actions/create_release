# CloudBees Unify Release Action

A CloudBees custom action that automates the creation and execution of CloudBees Unify application releases.

This action:
1. Builds a release manifest from the most recent component artifacts
2. Creates a CloudBees Unify application release
3. Starts the release workflow
4. Waits for the release to complete (with polling)
5. Reports the final result

## Usage

```yaml
- name: Create and run CloudBees Unify release
  uses: github.com/guru-actions/create_release@main
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
| `override_component_id` | Optional component ID to pin to a specific version | `""` (no override) |
| `override_version` | Optional version to use for overridden component | `""` (no override) |
| `release_name_prefix` | Prefix for auto-generated release name | `unify-release` |
| `max_wait_attempts` | Maximum polling attempts to wait for release completion | `60` |
| `wait_sleep_seconds` | Seconds to sleep between polling attempts | `10` |

## Outputs

| Output | Description |
|--------|-------------|
| `manifest` | Generated manifest JSON with component artifact versions |
| `release_id` | Created release ID |
| `release_name` | Created release name (e.g., `unify-release-20231208-143022`) |
| `run_id` | Automation run ID from the release execution |
| `status` | Final release status (`SUCCEEDED`, `FAILED`, `TIMEOUT`) |

## Examples

### Basic Usage

```yaml
jobs:
  deploy:
    steps:
      - name: Create release
        uses: github.com/guru-actions/create_release@main
        with:
          cb_api_token: ${{ secrets.CB_API_TOKEN }}
          cb_org_id: ${{ vars.CB_ORG_ID }}
          cb_application_id: ${{ vars.CB_APPLICATION_ID }}
          cb_workflow_id: ${{ vars.CB_WORKFLOW_ID }}
          cb_environment: "squid-demo-3"
```

### With Component Version Override

Pin a specific component to a specific version:

```yaml
- name: Create release with component override
  uses: github.com/guru-actions/create_release@main
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
  uses: github.com/guru-actions/create_release@main
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
  uses: github.com/guru-actions/create_release@main
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
  uses: github.com/guru-actions/create_release@main
  with:
    cb_api_token: ${{ secrets.CB_API_TOKEN }}
    cb_org_id: ${{ vars.CB_ORG_ID }}
    cb_application_id: ${{ vars.CB_APPLICATION_ID }}
    cb_workflow_id: ${{ vars.CB_WORKFLOW_ID }}
    cb_environment: "squid-demo"
    cb_artifact_labels: "demo,dev=true,v2"
```

### With Custom Wait Times

Adjust polling behavior for longer-running releases:

```yaml
- name: Create release with extended wait time
  uses: github.com/guru-actions/create_release@main
  with:
    cb_api_token: ${{ secrets.CB_API_TOKEN }}
    cb_org_id: ${{ vars.CB_ORG_ID }}
    cb_application_id: ${{ vars.CB_APPLICATION_ID }}
    cb_workflow_id: ${{ vars.CB_WORKFLOW_ID }}
    cb_environment: "squid-prod"
    max_wait_attempts: "120"  # Wait up to 20 minutes (120 * 10s)
    wait_sleep_seconds: "10"
```

### Using Outputs

Access the outputs from the action:

```yaml
- name: Create release
  id: release
  uses: github.com/guru-actions/create_release@main
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

## Requirements

- CloudBees API token with permissions to:
  - Read application and component information
  - Create and run releases
  - Query automation run status

## License

MIT