## Table of Contents

- [Overview](#overview)
- [Requirements](#requirements)
  - [Setting Up Infrastructure Credentials](#setting-up-infrastructure-credentials)
  - [Configuring E2E Credentials](#configuring-e2e-credentials)
  - [Adding External Secrets](#adding-external-secrets)
- [FAQs](#faqs)
- [Troubleshooting](#troubleshooting)
- [Contact Us](#contact-us)

## Overview

This document is designed to assist QE teams in preparing their testing repositories to run within Konflux CI. We'll cover all necessary preparations and requirements in detail to ensure a smooth integration and testing process.

> **Note:** For any questions or to brainstorm specific solutions, please reach out to us on the Slack channel [#forum-konflux-qe](https://slack.com).

## Requirements

To run integration tests in Konflux, there are several requirements:

1. **Application in Konflux**: Ensure you have created an application in Konflux.
2. [Setup external secrets](#adding-external-secrets) to sync all necessary secrets to run your integration tests. To run Konflux E2E, there are 2 mandatory secrets that need to be synced from Vault:
    1. **konflux-e2e-secrets**: Required to install and launch the Konflux E2E Framework.
    2. **konflux-test-infra**: Required to launch Rosa installation.

### Setting Up Infrastructure Credentials

**Infrastructure credentials** are stored in the vault with the name `konflux-test-infra`. If your team is not part of Konflux, create your own **Infrastructure credentials** in the vault using this structure as a reference:

**github-bot-commenter-token**: "ey...."

**cloud-credential-{*aws-region*}**:
```json
{
  "aws": {
    "region": "us-east-2",
    "access-key-secret": "none",
    "access-key-id": "none",
    "aws-account-id": "none",
    "rosa-hcp": {
      "rosa-token": "ey....",
      "aws-oidc-config-id": "none",
      "operator-roles-prefix": "none",
      "subnets-ids": "none",
      "install-role-arn": "none",
      "support-role-arn": "none",
      "worker-role-arn": "none"
    }
  }
}
```

### Configuring E2E Credentials

**E2E credentials** are stored in the vault with the name `konflux-e2e-secrets`. If your team is not part of Konflux, create your own **E2E credentials** in the vault using your own structure.

### Adding External Secrets

1. **Modify YAML**:
    - Adjust the following example YAML configuration to suit your tenant namespace:
    ```yaml
    ---
    apiVersion: external-secrets.io/v1beta1
    kind: ExternalSecret
    metadata:
      annotations:
        argocd.argoproj.io/sync-options: SkipDryRunOnMissingResource=true
        argocd.argoproj.io/sync-wave: '-1'
      name: <secret-name>
      namespace: <tenant-namespace>
    spec:
      dataFrom:
        - extract:
            conversionStrategy: Default
            decodingStrategy: None
            key: production/qe/<secret-name> # in case of Konflux components
      refreshInterval: 10m
      secretStoreRef:
        kind: ClusterSecretStore
        name: appsre-stonesoup-vault
      target:
        creationPolicy: Owner
        deletionPolicy: Delete
        name: <secret-name>
    ```

2. **Apply Configuration**:
    - Apply the YAML configuration to create the external secrets in your tenant namespace:
    ```sh
    kubectl apply -f <your-external-secret-config>.yaml
    ```

## FAQs

**Q:** How often do the secrets refresh?
**A:** Secrets are refreshed every 10 minutes as specified by the `refreshInterval` in the YAML configuration.

## Troubleshooting

- **Issue:** Integration test fails to start.
  - **Solution:** Verify that your AWS credentials and GitHub tokens are correctly configured and have the necessary permissions. Double-check the secret names and values in your vault.

- **Issue:** Unable to add a GitHub repository URL.
  - **Solution:** Ensure the URL is correct and accessible, and that you have the required permissions to access the repository. Check if there are any network restrictions or firewall rules blocking the access.

## Contact Us

For further assistance, feel free to reach out via the Slack channel [#forum-konflux-qe](https://slack.com)..
