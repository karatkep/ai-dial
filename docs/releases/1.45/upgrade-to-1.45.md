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
   - ai-dial-quickapps-backend: `0.8.1`
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

<!-- 🤖 AI-generated draft for DIAL 1.45 — review before merging -->

#### Cross-component notes

- ai-dial-admin-deployment-manager-backend 0.17.0 introduces API-key authentication by validating the 'Api-Key' header against ai-dial-core's '/v1/user/info' endpoint; ensure ai-dial-core is reachable from the deployment manager and that DIAL Core connectivity/config (e.g., Core URL) is configured before upgrading the deployment manager.
- ai-dial-chat-themes 0.16.0 adds default logos and favicons for both ai-dial-chat and ai-dial-admin-frontend; if either component uses customized theme overrides, verify those overrides are not silently replaced or broken after upgrading ai-dial-chat-themes.
- ai-dial-adapter-openai 0.40.0 deprecates DIAL_USE_FILE_STORAGE with no stated replacement; if this env var is used in your deployment, audit whether file storage behavior changes affect ai-dial-core or any adapter that depends on file routing through the OpenAI adapter, and plan removal before a future release drops it entirely.
- ai-dial-admin-deployment-manager-backend 0.17.0 has a published external upgrade guide (https://github.com/epam/ai-dial-admin-deployment-manager-backend/blob/0.17.0/docs/upgrade-plans/0.17.0.md) rated critical; review and execute it before upgrading any dependent admin components (ai-dial-admin-frontend 0.17.1, ai-dial-admin-backend 0.17.1).

### ai-dial-admin-deployment-manager-backend

> [!CAUTION]
> This release includes high-priority changes. Please review the [full upgrade guide](https://github.com/epam/ai-dial-admin-deployment-manager-backend/blob/0.17.0/docs/upgrade-plans/0.17.0.md) before proceeding.

#### Breaking changes

**NODE_POOLS config format changed from label-key/capacity to YAML document with explicit Kubernetes scheduling primitives; new NODE_POOL_DEFAULT and NODE_POOL_DEFAULT_MODEL create-time defaults added**

The NODE_POOLS configuration is now a YAML document. The previous label-key/capacity config format is replaced with explicit 'nodeSelector', 'affinity', and 'tolerations' primitives per pool. Two new default-stamping env vars (NODE_POOL_DEFAULT and NODE_POOL_DEFAULT_MODEL) are introduced for create-time defaults.

| Previous configuration | Required action |
|---|---|
| NODE_POOLS configured with label-key/capacity format | Migrate NODE_POOLS to YAML document format using explicit nodeSelector/affinity/tolerations primitives per pool. Review full upgrade guide for schema details. |
| No NODE_POOL_DEFAULT or NODE_POOL_DEFAULT_MODEL set | Optionally set NODE_POOL_DEFAULT and/or NODE_POOL_DEFAULT_MODEL env vars to define create-time default pool assignments. |

**Removed deprecated config properties 'config.rest.security.default.allowedRoles' and 'providers.*.allowed-roles'; must migrate to 'roles-mapping'**

The previously deprecated 'config.rest.security.default.allowedRoles' and 'providers.*.allowed-roles' config properties have been removed entirely. Deployments still using these properties must migrate to the 'roles-mapping' configuration.

| Previous configuration | Required action |
|---|---|
| 'config.rest.security.default.allowedRoles' set in config | Remove 'config.rest.security.default.allowedRoles' and replace with equivalent 'roles-mapping' configuration. |
| 'providers.*.allowed-roles' set in config | Remove 'providers.*.allowed-roles' entries and replace with equivalent 'roles-mapping' configuration. |

#### New environment variables

| Variable | Default | Required | Description |
|---|---|---|---|
| `NODE_POOL_DEFAULT` | — | No | Specifies the default node pool to stamp at resource create-time when no explicit pool is provided. |
| `NODE_POOL_DEFAULT_MODEL` | — | No | Specifies the default node pool for model resources at create-time. |

#### Config / Helm changes

- **Removed** `config.rest.security.default.allowedRoles`: Previously deprecated config property for allowed roles; now fully removed. Migrate to 'roles-mapping'.
- **Removed** `providers.*.allowed-roles`: Previously deprecated per-provider allowed-roles config; now fully removed. Migrate to 'roles-mapping'.
- **Added** `roles-mapping`: Replacement configuration for the removed 'config.rest.security.default.allowedRoles' and 'providers.*.allowed-roles' properties.

> [!NOTE]
> NODE_POOLS now uses explicit Kubernetes scheduling primitives (nodeSelector, affinity, tolerations) — Node Pool Configuration is generally available. Each pool in the YAML document supports 'nodeSelector', 'affinity', and 'tolerations' fields directly. The previous label-key/capacity abstraction is gone.

> [!NOTE]
> New API-key authentication via DIAL Core added alongside existing JWT/OIDC flow — The 'Api-Key' header is now validated against DIAL Core's '/v1/user/info' endpoint. This is an additive change but may require DIAL Core connectivity and configuration to function correctly.

> [!NOTE]
> Documentation warning added regarding H2 usage — A warning note regarding H2 (HTTP/2 or H2 database) usage was added to documentation. Operators should review the upgrade guide for details.

### ai-dial-adapter-vertexai

#### Breaking changes

**Veo: `pubsub_topic` field removed from config**

The `pubsub_topic` field has been removed from the Veo model configuration. Any deployment that includes this field must remove it before or during upgrade.

| Previous configuration | Required action |
|---|---|
| `pubsub_topic` field present in Veo model config | Remove the `pubsub_topic` field from the Veo model configuration |

#### Config / Helm changes

- **Removed** `veo.pubsub_topic`: The `pubsub_topic` field has been removed from the Veo model configuration.

### ai-dial-adapter-openai

#### Deprecated environment variables

| Variable | Description |
|---|---|
| `DIAL_USE_FILE_STORAGE` | DIAL_USE_FILE_STORAGE is deprecated. No replacement explicitly mentioned in release notes. |

**Migration:** _DIAL_USE_FILE_STORAGE is set_ → Review deprecation impact; plan to remove this env var in a future upgrade

### ai-dial-chat-themes

#### Config / Helm changes

- **Added** `config.json / bg-inverted`: New 'bg-inverted' property added to config.json for theme configuration.

