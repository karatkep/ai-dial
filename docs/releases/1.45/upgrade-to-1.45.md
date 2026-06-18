# Instructions

## Versions

1. Helm chart versions:
   - dial: `6.4.0`
   - dial-core: `5.2.1`
   - dial-extension: `3.1.1`
   - dial-admin: `0.15.0`
2. Main components versions:
   - ai-dial-adapter-bedrock: `0.40.0`
   - ai-dial-adapter-openai: `0.40.0`
   - ai-dial-adapter-vertexai: `0.36.0`
   - ai-dial-adapter-dial: `0.15.0`
   - ai-dial-chat-themes: `0.16.0`
   - ai-dial-chat: `0.46.3`
   - ai-dial-core: `0.44.4`
   - ai-dial-analytics-realtime: `0.24.2`
   - ai-dial-rag: `0.42.0`
   - ai-dial-log-parser: `0.3.0`
   - ai-dial-code-interpreter: `0.2.0`
   - ai-dial-app-controller: `0.4.0`
   - ai-dial-app-builder-python: `0.1.0`
   - ai-dial-quickapps-backend: `0.8.0`
   - ai-dial-mind-map-backend: `0.14.1`
   - ai-dial-mind-map-frontend: `0.13.0`
   - ai-dial-admin-backend: `0.17.1`
   - ai-dial-admin-frontend: `0.17.1`
   - ai-dial-admin-deployment-manager-backend: `0.17.0`

## Before upgrade

### General notes

- Please review the [Config changes](#config-changes) chapter carefully for each component that is used in your DIAL installation. Changes in components' configuration may be required.
- Please check if any image tag overrides (`image.tag`) are present and remove them if they are not required anymore.
- Please check and add `image.repository` to change the image location for `redis`, `postgresql`, `keycloak` and `keycloakConfigCli` components to start using alternative Docker registries (e.g. Amazon ECR Public Gallery) if required.

### Release-specific notes

#### ai-dial-quickapps-backend `0.8.0`

### New environment variables

| Variable                             | Default    | Description                                                                                                                                                                                                                                     |
|--------------------------------------|------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `DEFAULT_ORCHESTRATOR_DEPLOYMENT_ID` | —          | Default DIAL deployment id used as the orchestrator model when a QuickApp manifest omits `orchestrator.deployment`. Also surfaces as the JSON-schema `default` for that field so DIAL Core can pre-fill new manifests. Apps can override per-app. |
| `DEFAULT_FILE_LOADING_SIZE_LIMIT`    | `10485760` | Deployment-wide cap (bytes, `> 0`; default 10 MiB) on files the agent downloads. Overridden per-app by `features.file_loading.size_limit` in the manifest.                                                                                       |
| `USE_SYSTEM_CA_CERTS`                | unset      | When set to `1`, merges every `*.crt` file under `/certificates/` with the Alpine system CA bundle at container startup and exports `SSL_CERT_FILE` to the merged path so outbound HTTP calls trust private/corporate root CAs. Opt-in; unset keeps existing behaviour. |

### Behavioral changes

> [!NOTE]
> Two preview-gated features have graduated to GA and are now active regardless of `ENABLE_PREVIEW_FEATURES`:
>
> - **Time Awareness** — `features.timestamp` (#290)
> - **DIAL Prompt Skills** — the `skills` config field and `DialPromptSkillsModule` (#291)

### DIAL Configuration changes

> [!IMPORTANT]
> Operators must update DIAL Core's configuration when upgrading to this release. The `/v1/configuration-support/*` endpoints are no longer served via a global DIAL Core `routes` entry — they are declared on the QuickApps application type itself.
>
> **Required migration** (#319):
>
> 1. **Remove** any `quick_apps2`-style entry from DIAL Core's global `routes` block (it will be ignored from now on).
> 2. **Add** the new `dial:applicationTypeRoutes` block to the QuickApps entry under `applicationTypeSchemas` — apply the schema snippet from [PR #319](https://github.com/epam/ai-dial-quickapps-backend/pull/319) verbatim.

### Schema deprecations

> [!CAUTION]
> Still accepted in app manifests, but will be removed in future versions (#287).

| Legacy key                | Replacement     | Affected config model    |
|---------------------------|-----------------|--------------------------|
| `name` (deployment field) | `deployment_id` | `DialDeploymentConfig`   |
| `dial_id`                 | `deployment_id` | `DialMCPToolSet`         |

---

#### ai-dial-admin-deployment-manager-backend `0.17.0`

This release includes **many critical and high-priority changes**. Please review the [full upgrade guide](https://github.com/epam/ai-dial-admin-deployment-manager-backend/blob/0.17.0/docs/upgrade-plans/0.17.0.md) before proceeding.

## Config changes

### ai-dial-adapter-bedrock

<!-- TODO: add config changes for ai-dial-adapter-bedrock -->

### ai-dial-adapter-openai

<!-- TODO: add config changes for ai-dial-adapter-openai -->

### ai-dial-adapter-vertexai

<!-- TODO: add config changes for ai-dial-adapter-vertexai -->

### ai-dial-adapter-dial

<!-- TODO: add config changes for ai-dial-adapter-dial -->

### ai-dial-chat-themes

<!-- TODO: add config changes for ai-dial-chat-themes -->

### ai-dial-chat

<!-- TODO: add config changes for ai-dial-chat -->

### ai-dial-core

<!-- TODO: add config changes for ai-dial-core -->

### ai-dial-analytics-realtime

<!-- TODO: add config changes for ai-dial-analytics-realtime -->

### ai-dial-rag

<!-- TODO: add config changes for ai-dial-rag -->

### ai-dial-log-parser

<!-- TODO: add config changes for ai-dial-log-parser -->

### ai-dial-code-interpreter

<!-- TODO: add config changes for ai-dial-code-interpreter -->

### ai-dial-app-controller

<!-- TODO: add config changes for ai-dial-app-controller -->

### ai-dial-app-builder-python

<!-- TODO: add config changes for ai-dial-app-builder-python -->

### ai-dial-quickapps-backend

<!-- TODO: add config changes for ai-dial-quickapps-backend -->

### ai-dial-mind-map-backend

<!-- TODO: add config changes for ai-dial-mind-map-backend -->

### ai-dial-mind-map-frontend

<!-- TODO: add config changes for ai-dial-mind-map-frontend -->

### ai-dial-admin-backend

<!-- TODO: add config changes for ai-dial-admin-backend -->

### ai-dial-admin-frontend

<!-- TODO: add config changes for ai-dial-admin-frontend -->

### ai-dial-admin-deployment-manager-backend

<!-- TODO: add config changes for ai-dial-admin-deployment-manager-backend -->
