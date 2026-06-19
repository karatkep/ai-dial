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

<!-- 🤖 AI-generated draft for DIAL 1.45 — review before merging -->

#### Cross-component notes

- ai-dial-quickapps-backend 0.8.0 requires a coordinated change to ai-dial-core configuration: remove the legacy quick_apps2-style global routes entry for /v1/configuration-support/* and add the new dial:applicationTypeRoutes block to the QuickApps applicationTypeSchemas entry — failure to do both atomically will make /v1/configuration-support/* endpoints unreachable. Apply the schema snippet from https://github.com/epam/ai-dial-quickapps-backend/pull/319 and upgrade ai-dial-core configuration in the same deployment window as ai-dial-quickapps-backend.
- ai-dial-admin-deployment-manager-backend 0.17.0 has a mandatory external upgrade guide at https://github.com/epam/ai-dial-admin-deployment-manager-backend/blob/0.17.0/docs/upgrade-plans/0.17.0.md that must be read before deploying — it contains critical NODE_POOLS YAML format migration and roles-mapping config migration that cannot be reconstructed from release notes alone. Do not deploy this component without completing both migrations in advance.
- ai-dial-adapter-openai 0.40.0 deprecates DIAL_USE_FILE_STORAGE and now auto-enables DIAL storage whenever DIAL_URL is set — if any environment previously relied on DIAL_URL being set while DIAL_USE_FILE_STORAGE=False to keep storage disabled, that environment will silently gain active storage after this upgrade. Audit all adapter-openai deployments where storage should remain disabled and remove DIAL_URL if necessary.
- ai-dial-admin-deployment-manager-backend 0.17.0 adds API-key authentication by validating the Api-Key header against ai-dial-core's /v1/user/info endpoint — ensure network connectivity between ai-dial-admin-deployment-manager-backend and ai-dial-core's /v1/user/info is confirmed before upgrading, or API-key-authenticated requests will fail.
- ai-dial-code-interpreter 0.2.0 adds session ID verification that may reject existing clients not currently supplying valid session IDs — audit all callers of ai-dial-code-interpreter (including ai-dial-core route targets and any direct integrations) to confirm session IDs are correctly provided before upgrading.
- ai-dial-chat-themes 0.16.0 updates default logos and favicons for both ai-dial-chat and ai-dial-admin-frontend — deployments that mount custom overrides for these assets should verify overrides are still applied correctly after upgrading ai-dial-chat-themes, as new defaults have been added for both components.
- ai-dial-mind-map-backend 0.14.1 introduces a public_url feature (PR #76) whose configuration mechanism is unconfirmed in release notes — DevOps should inspect PR #76 before deploying to determine whether a new env var, config key, or Helm value must be set, particularly if ai-dial-mind-map-frontend or reverse-proxy routing depends on the backend's public URL.

### ai-dial-admin-deployment-manager-backend

> [!CAUTION]
> This release includes high-priority changes. Please review the [full upgrade guide](https://github.com/epam/ai-dial-admin-deployment-manager-backend/blob/0.17.0/docs/upgrade-plans/0.17.0.md) before proceeding.

#### Breaking changes

**NODE_POOLS config format changed from label-key/capacity to explicit Kubernetes scheduling primitives; new env vars NODE_POOL_DEFAULT and NODE_POOL_DEFAULT_MODEL added**

The NODE_POOLS configuration is now a YAML document and must express scheduling via explicit 'nodeSelector', 'affinity', and 'tolerations' primitives per pool. The old label-key/capacity format is no longer accepted. Two new create-time default fields NODE_POOL_DEFAULT and NODE_POOL_DEFAULT_MODEL are introduced.

| Previous configuration | Required action |
|---|---|
| NODE_POOLS configured using label-key/capacity format | Rewrite NODE_POOLS as a YAML document using explicit nodeSelector/affinity/tolerations primitives per pool. Set NODE_POOL_DEFAULT and NODE_POOL_DEFAULT_MODEL as needed for create-time defaults. |

**Removed deprecated config properties 'config.rest.security.default.allowedRoles' and 'providers.*.allowed-roles'; must migrate to 'roles-mapping'**

The previously deprecated config keys 'config.rest.security.default.allowedRoles' and 'providers.*.allowed-roles' have been removed entirely. Deployments still using these keys must migrate to the 'roles-mapping' configuration.

| Previous configuration | Required action |
|---|---|
| 'config.rest.security.default.allowedRoles' set in config | Remove this key and configure equivalent access control via 'roles-mapping'. |
| 'providers.*.allowed-roles' set in config | Remove these keys and configure equivalent access control via 'roles-mapping'. |

#### New environment variables

| Variable | Default | Required | Description |
|---|---|---|---|
| `NODE_POOL_DEFAULT` | — | No | Specifies the default node pool applied at create-time for deployments. Introduced as part of the new explicit Kubernetes scheduling primitives node pool configuration. |
| `NODE_POOL_DEFAULT_MODEL` | — | No | Specifies the default node pool for model deployments applied at create-time. Introduced as part of the new explicit Kubernetes scheduling primitives node pool configuration. |

#### Config / Helm changes

- **Removed** `config.rest.security.default.allowedRoles`: Previously deprecated config key has been fully removed. Migrate to 'roles-mapping'.
- **Removed** `providers.*.allowed-roles`: Previously deprecated per-provider config key has been fully removed. Migrate to 'roles-mapping'.
- **Default changed** `NODE_POOLS`: `label-key/capacity format` → `YAML document with explicit nodeSelector/affinity/tolerations primitives per pool` — NODE_POOLS must now be expressed as a YAML document using Kubernetes scheduling primitives. The previous label-key/capacity format is no longer supported.

> [!NOTE]
> New API-key authentication via DIAL Core added alongside existing JWT/OIDC — The service now validates the 'Api-Key' header against DIAL Core's '/v1/user/info' endpoint. Operators deploying with DIAL Core API keys must ensure network connectivity between this service and DIAL Core's /v1/user/info endpoint.

> [!NOTE]
> Node Pool Configuration is now generally available with explicit Kubernetes scheduling primitives — nodeSelector, affinity, and tolerations can now be specified per pool. Create-time default stamping via NODE_POOL_DEFAULT and NODE_POOL_DEFAULT_MODEL is available.

### ai-dial-quickapps-backend

#### Breaking changes

**DIAL Core global routes entry for /v1/configuration-support/* must be removed and replaced with dial:applicationTypeRoutes on the QuickApps application type** _(cross-component)_

The /v1/configuration-support/* endpoints are no longer served via a global DIAL Core routes block. They are now declared on the QuickApps application type itself via dial:applicationTypeRoutes. The old global route entry will be silently ignored, meaning the endpoints will stop working unless the new applicationTypeRoutes block is added.

| Previous configuration | Required action |
|---|---|
| DIAL Core global routes block contains a quick_apps2-style entry for /v1/configuration-support/* | Remove the quick_apps2-style entry from DIAL Core's global routes block |
| QuickApps entry under applicationTypeSchemas does not have a dial:applicationTypeRoutes block | Add the new dial:applicationTypeRoutes block to the QuickApps entry under applicationTypeSchemas, applying the schema snippet from PR #319 verbatim |

**Time Awareness and DIAL Prompt Skills features graduate to GA — active regardless of ENABLE_PREVIEW_FEATURES**

features.timestamp (Time Awareness) and the skills config field / DialPromptSkillsModule (DIAL Prompt Skills) are no longer gated by ENABLE_PREVIEW_FEATURES. They will now be active for all apps that configure them, even if ENABLE_PREVIEW_FEATURES is not set.

| Previous configuration | Required action |
|---|---|
| ENABLE_PREVIEW_FEATURES was not set; features.timestamp or skills were inactive even if configured | Review apps using features.timestamp or skills — they will now be active. Ensure this is intended behavior. |

#### New environment variables

| Variable | Default | Required | Description |
|---|---|---|---|
| `DEFAULT_ORCHESTRATOR_DEPLOYMENT_ID` | — | No | Default DIAL deployment id used as the orchestrator model when a QuickApp manifest omits orchestrator.deployment. Also surfaces as the JSON-schema default for that field so DIAL Core can pre-fill new manifests. Apps can override per-app. |
| `DEFAULT_FILE_LOADING_SIZE_LIMIT` | `10485760` | No | Deployment-wide cap (bytes, > 0; default 10 MiB) on files the agent downloads. Overridden per-app by features.file_loading.size_limit in the manifest. Replaces a previously hardcoded 10 MiB limit. |
| `USE_SYSTEM_CA_CERTS` | — | No | When set to 1, merges every *.crt file under /certificates/ with the Alpine system CA bundle at container startup and exports SSL_CERT_FILE so outbound HTTP calls trust private/corporate root CAs. Opt-in; unset keeps existing behaviour. |

#### Config / Helm changes

- **Removed** `routes.<quick_apps2-style entry>`: The global DIAL Core routes entry for /v1/configuration-support/* must be removed. It will be silently ignored if left in place, causing those endpoints to become unreachable unless the new dial:applicationTypeRoutes block is added.
- **Deprecated** `DialDeploymentConfig.name` → `DialDeploymentConfig.deployment_id`: The name field in DialDeploymentConfig is deprecated. Use deployment_id instead. Legacy key still validates via validation_alias and is retained as a deprecated sibling in the published JSON schema under anyOf so existing manifests remain valid.
- **Deprecated** `DialMCPToolSet.dial_id` → `DialMCPToolSet.deployment_id`: The dial_id field in DialMCPToolSet is deprecated. Use deployment_id instead. Legacy key still validates via validation_alias and is retained as a deprecated sibling in the published JSON schema under anyOf so existing manifests remain valid.
- **Deprecated** `features.display.stage.show` → `features.stage_display.level`: display.stage.show = false is deprecated in favor of the new features.stage_display.level field (error/info/debug). The old value is still honored at info level with a warning logged.
- **Added** `applicationTypeSchemas.<quickapps-entry>.dial:applicationTypeRoutes`: New block required on the QuickApps application type entry in DIAL Core's applicationTypeSchemas config. Routes /v1/configuration-support/* endpoints via the application type instead of a global routes block. Schema snippet must be applied from PR #319.
- **Added** `applicationTypeSchemas.<quickapps-entry>.dial:attachmentPaths`: New field allowing DIAL Core to enforce ACL on prompt URLs in skills/validate request bodies.

> [!NOTE]
> Corporate root CA support via USE_SYSTEM_CA_CERTS and /certificates/ mount — Set USE_SYSTEM_CA_CERTS=1 and mount *.crt files under /certificates/ to trust private/corporate root CAs for all outbound HTTP calls. Container merges the certs with the Alpine system bundle at startup and exports SSL_CERT_FILE.

### ai-dial-adapter-vertexai

#### Breaking changes

**Veo: `pubsub_topic` field removed from config**

The `pubsub_topic` field has been removed from the Veo model configuration. Any deployment config that includes this field must be updated to remove it.

| Previous configuration | Required action |
|---|---|
| Veo config contains `pubsub_topic` field | Remove `pubsub_topic` from the Veo model configuration |

#### Config / Helm changes

- **Removed** `veo.pubsub_topic`: The `pubsub_topic` field has been removed from the Veo model configuration.

> [!NOTE]
> WIF support added for AWS container credential providers — Workload Identity Federation (WIF) is now supported for AWS container credential providers, enabling token-based authentication without static credentials.

### ai-dial-adapter-openai

#### Deprecated environment variables

| Variable | Description |
|---|---|
| `DIAL_USE_FILE_STORAGE` | DIAL_USE_FILE_STORAGE is deprecated. DIAL Storage is now enabled automatically when DIAL_URL is set. |

**Migration:** _DIAL_USE_FILE_STORAGE=True explicitly set_ → No immediate action required; var is deprecated but still works. Plan to remove it in a future release.

**Migration:** _DIAL_USE_FILE_STORAGE=False or unset to disable storage_ → Verify new automatic enablement behavior does not unintentionally activate storage; remove DIAL_URL if storage should stay disabled.

### ai-dial-chat-themes

#### Config / Helm changes

- **Added** `config.json / bg-inverted`: New 'bg-inverted' property added to config.json theme configuration.

### ai-dial-code-interpreter

> [!NOTE]
> Session ID verification added — Session ID verification is now enforced. If clients or callers do not supply a valid session ID, requests may be rejected. Review integration points to ensure session IDs are correctly provided.

