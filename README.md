# GCP Workload Identity Test Application (Helm Chart)

This repository contains a Helm chart used to validate Google Cloud Workload Identity on a GKE cluster configured with Config Connector. The chart deploys a test pod that authenticates to Google Cloud Storage using a Kubernetes ServiceAccount bound to a Google Service Account.

## Purpose

The chart verifies:
- Workload Identity is correctly configured between the Kubernetes ServiceAccount and Google Service Account
- Config Connector successfully provisions the required IAM and storage resources
- A pod can use Workload Identity without service account keys

## Chart Structure

```
├── Chart.yaml
├── templates
│   ├── configmap.yaml
│   ├── _helpers.tpl
│   ├── pod.yaml
│   └── serviceaccount.yaml
└── values.yaml
```

## Requirements

- GKE cluster with Workload Identity enabled
- Config Connector installed in cluster mode
- Namespace annotated with the correct Google Cloud project ID
- Corresponding Google Service Account and IAM bindings created via Config Connector
- A Google Cloud Storage bucket with read access granted to the Google Service Account

## Configuration

Set required values through Helm flags or a custom values file.

Required values are:
- `gcp.projectId # Google Cloud project ID`
- `gcp.gsaEmail # Email of the Google Service Account`
- `gcp.bucketName # Target GCS bucket name`
- `app.namespace # Kubernetes namespace for deployment`

## Installation

**a/ Using command-line flags:**

```
helm install wi-test . \
  --set gcp.projectId=<PROJECT_ID> \
  --set gcp.gsaEmail=app-gsa@<PROJECT_ID>.iam.gserviceaccount.com \
  --set gcp.bucketName=cc-wi-demo-<PROJECT_ID> \
  --set app.namespace=<NAMESPACE>
```

**b/ Using a values file:**

`helm install wi-test . -f my-values.yaml`

## Verifying Workload Identity

Check pod logs:

`kubectl logs gcp-wi-test-pod -n <NAMESPACE>`

Expected behavior:
- The test script initializes a GCS client
- The script lists objects in the specified bucket
- Output confirms Workload Identity authentication

## Volume Mount Configuration

The pod requires a projected volume that combines:
1. Kubernetes service account token
2. GCP credential configuration (from ConfigMap `sts-config`)

Default paths (following Google Cloud conventions):
- Mount path: `/var/run/secrets/tokens/gcp-ksa`
- Credential config: `/var/run/secrets/tokens/gcp-ksa/config`
- Token: `/var/run/secrets/tokens/gcp-ksa/token`

These paths can be customized, but must match the `GOOGLE_APPLICATION_CREDENTIALS` 
environment variable and the `credential-source-file` in your STS configuration.

## Configuration

### GCP Settings (`gcp`)

| Parameter | Description |
|-----------|-------------|
| `projectId` | GCP Project ID |
| `gsaEmail` | GCP Service Account email |
| `bucketName` | GCS bucket name for testing |
| `oidcProviderUrl` | Full WIF provider path |
| `tokenAudience` | Service account token audience |


### Token Audience Configuration

The `tokenAudience` parameter controls what audience is requested in the Kubernetes 
service account token. This must match one of the `allowedAudiences` in your GCP 
Workload Identity Pool Provider.

**Option 1: Full WIF provider URL**
```yaml
gcp:
  oidcProviderUrl: "projects/123/locations/global/workloadIdentityPools/my-pool/providers/my-provider"
  tokenAudience: "//iam.googleapis.com/projects/123/locations/global/workloadIdentityPools/my-pool/providers/my-provider"
```

Requires your OIDC provider to have:
```yaml
oidc:
  allowedAudiences:
    - "//iam.googleapis.com/projects/123/locations/global/workloadIdentityPools/my-pool/providers/my-provider"
```

**Option 2: Custom audience**
```yaml
gcp:
  tokenAudience: "whatever.com"
```

Requires your OIDC provider to have:
```yaml
oidc:
  allowedAudiences:
    - "whatever.com"
```