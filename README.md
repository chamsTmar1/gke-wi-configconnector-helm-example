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
- gcp.projectId # Google Cloud project ID
- gcp.gsaEmail # Email of the Google Service Account
- gcp.bucketName # Target GCS bucket name
- app.namespace # Kubernetes namespace for deployment

## Installation

a/ Using command-line flags:

```
helm install wi-test . \
  --set gcp.projectId=<PROJECT_ID> \
  --set gcp.gsaEmail=app-gsa@<PROJECT_ID>.iam.gserviceaccount.com \
  --set gcp.bucketName=cc-wi-demo-<PROJECT_ID> \
  --set app.namespace=<NAMESPACE>
```

b/ Using a values file:

`helm install wi-test . -f my-values.yaml`

## Verifying Workload Identity

Check pod logs:

`kubectl logs gcp-wi-test-pod -n <NAMESPACE>`

Expected behavior:
- The test script initializes a GCS client
- The script lists objects in the specified bucket
- Output confirms Workload Identity authentication

## Uninstalling

`helm uninstall wi-test -n <NAMESPACE>`
