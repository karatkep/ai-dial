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

- ai-dial-quickapps-backend 0.8.0 has a critical cross-component breaking change: the global DIAL Core routes entry for /v1/configuration-support/* must be REMOVED from ai-dial-core config AND replaced with a new dial:applicationTypeRoutes block on the QuickApps applicationTypeSchemas entry before or during upgrade. Consult https://github.com/epam/ai-dial-quickapps-backend/pull/319 for the exact schema snippet to apply to ai-dial-core configuration.
- ai-dial-admin-deployment-manager-backend 0.17.0 introduces API-key authentication by validating the 'Api-Key' header against ai-dial-core's /v1/user/info endpoint; ensure ai-dial-core is reachable from ai-dial-admin-deployment-manager-backend and upgraded before or alongside it if this auth method is to be used.
- ai-dial-adapter-openai 0.40.0 deprecates DIAL_USE_FILE_STORAGE with no stated replacement; if this variable is currently set in deployments that rely on ai-dial-core file storage integration, audit the interaction before removing it to avoid silent breakage.
- ai-dial-chat-themes 0.16.0 ships new default logos and favicons for both DIAL Chat and DIAL Admin; teams deploying ai-dial-chat or ai-dial-admin-frontend with custom theme overrides should verify the new defaults in config.json (including the new bg-inverted property and neutral color) do not conflict with existing customizations before releasing to production.
- ai-dial-admin-deployment-manager-backend 0.17.0 has a dedicated external upgrade plan at https://github.com/epam/ai-dial-admin-deployment-manager-backend/blob/0.17.0/docs/upgrade-plans/0.17.0.md rated critical; upgrade this component last in the release sequence and fully execute that guide, including migrating NODE_POOLS to the new YAML format and replacing config.rest.security.default.allowedRoles / providers.*.allowed-roles with roles-mapping.
- ai-dial-code-interpreter 0.2.0 adds session ID verification that may reject requests from any integrated component (e.g. ai-dial-core, ai-dial-chat) that does not supply a valid session ID; verify that all upstream callers of the code interpreter correctly pass session IDs before upgrading, and consult PR #12 for the exact header/parameter contract.

### ai-dial-admin-deployment-manager-backend

> [!CAUTION]
> This release includes high-priority changes. Please review the [full upgrade guide](https://github.com/epam/ai-dial-admin-deployment-manager-backend/blob/0.17.0/docs/upgrade-plans/0.17.0.md) before proceeding.

#### Breaking changes

**NODE_POOLS config replaced: label-key/capacity format removed, now requires explicit Kubernetes scheduling primitives as YAML document; new NODE_POOL_DEFAULT and NODE_POOL_DEFAULT_MODEL vars added**

The NODE_POOLS environment variable format has changed from a label-key/capacity-based config to a YAML document containing explicit 'nodeSelector', 'affinity', and 'tolerations' per pool. Two new create-time default vars (NODE_POOL_DEFAULT and NODE_POOL_DEFAULT_MODEL) are also introduced. Deployments using the old format will break without migration.

| Previous configuration | Required action |
|---|---|
| NODE_POOLS set using old label-key/capacity config format | Rewrite NODE_POOLS as a YAML document with explicit 'nodeSelector', 'affinity', and 'tolerations' primitives per pool; set NODE_POOL_DEFAULT and NODE_POOL_DEFAULT_MODEL as needed |

**Removed deprecated config properties 'config.rest.security.default.allowedRoles' and 'providers.*.allowed-roles'; must migrate to 'roles-mapping'**

The previously deprecated config keys 'config.rest.security.default.allowedRoles' and 'providers.*.allowed-roles' have been fully removed. Deployments that still define these keys must migrate to the 'roles-mapping' configuration.

| Previous configuration | Required action |
|---|---|
| 'config.rest.security.default.allowedRoles' defined in config | Remove this key and configure equivalent access control via 'roles-mapping' |
| 'providers.*.allowed-roles' defined in config | Remove this key and configure equivalent access control via 'roles-mapping' |

#### New environment variables

| Variable | Default | Required | Description |
|---|---|---|---|
| `NODE_POOL_DEFAULT` | — | No | Specifies the default node pool to stamp at create-time for general deployments. |
| `NODE_POOL_DEFAULT_MODEL` | — | No | Specifies the default node pool to stamp at create-time for model deployments. |

#### Config / Helm changes

- **Removed** `config.rest.security.default.allowedRoles`: Previously deprecated config key fully removed. Migrate to 'roles-mapping'.
- **Removed** `providers.*.allowed-roles`: Previously deprecated config key fully removed. Migrate to 'roles-mapping'.

> [!NOTE]
> NODE_POOLS is now a YAML document with explicit Kubernetes scheduling primitives (nodeSelector, affinity, tolerations) per pool — Node Pool Configuration is generally available. Each pool entry in the YAML document must specify 'nodeSelector', 'affinity', and/or 'tolerations'. The old label-key/capacity format is no longer accepted.

> [!NOTE]
> New API-key authentication via DIAL Core: 'Api-Key' header validated against Core's '/v1/user/info' alongside existing JWT/OIDC — The service now supports API-key authentication by validating the 'Api-Key' header against DIAL Core's '/v1/user/info' endpoint. This supplements (does not replace) the existing JWT/OIDC flow. Operators should ensure DIAL Core connectivity is available if API-key auth is to be used.

### ai-dial-quickapps-backend

#### Breaking changes

**DIAL Core global routes entry for /v1/configuration-support/* must be removed and replaced with applicationTypeRoutes on the QuickApps application type** _(cross-component)_

The /v1/configuration-support/* endpoints are no longer served via a global DIAL Core routes block. They are now declared on the QuickApps application type itself via dial:applicationTypeRoutes. Any existing global routes entry (e.g. quick_apps2-style) must be removed and the new dial:applicationTypeRoutes block must be added to the QuickApps entry under applicationTypeSchemas.

| Previous configuration | Required action |
|---|---|
| DIAL Core global routes block contains a quick_apps2-style entry for /v1/configuration-support/* | Remove the global routes entry for /v1/configuration-support/* from DIAL Core configuration |
| QuickApps entry under applicationTypeSchemas lacks dial:applicationTypeRoutes block | Add the new dial:applicationTypeRoutes block to the QuickApps entry under applicationTypeSchemas, applying the schema snippet from PR #319 verbatim |

#### New environment variables

| Variable | Default | Required | Description |
|---|---|---|---|
| `DEFAULT_ORCHESTRATOR_DEPLOYMENT_ID` | — | No | Default DIAL deployment id used as the orchestrator model when a QuickApp manifest omits orchestrator.deployment. Also surfaces as the JSON-schema default for that field so DIAL Core can pre-fill new manifests. Apps can override per-app. |
| `DEFAULT_FILE_LOADING_SIZE_LIMIT` | `10485760` | No | Deployment-wide cap (bytes, > 0; default 10 MiB) on files the agent downloads. Overridden per-app by features.file_loading.size_limit in the manifest. Replaces a previously hardcoded 10 MiB limit. |
| `USE_SYSTEM_CA_CERTS` | — | No | When set to 1, merges every *.crt file under /certificates/ with the Alpine system CA bundle at container startup and exports SSL_CERT_FILE to the merged path so outbound HTTP calls trust private/corporate root CAs. Opt-in; unset keeps existing behaviour. |

#### Config / Helm changes

- **Removed** `DIAL Core global routes block: quick_apps2-style entry for /v1/configuration-support/*`: Must be removed from DIAL Core's global routes block. This entry will be ignored from now on; the routes are now declared on the application type itself.
- **Deprecated** `app manifest: DialDeploymentConfig.name` → `deployment_id`: Legacy key name on DialDeploymentConfig still validates via validation_alias and the published JSON schema retains it under anyOf, but will be removed in a future version.
- **Deprecated** `app manifest: DialMCPToolSet.dial_id` → `deployment_id`: Legacy key dial_id on DialMCPToolSet still validates via validation_alias and the published JSON schema retains it under anyOf, but will be removed in a future version.
- **Deprecated** `app manifest: display.stage.show = false` → `features.stage_display.level`: display.stage.show = false is still honored at info level with a warning but is deprecated in favor of the new features.stage_display.level field.
- **Added** `applicationTypeSchemas[QuickApps].dial:applicationTypeRoutes`: New block required in the QuickApps application type entry under DIAL Core's applicationTypeSchemas. Replaces the former global routes entry for /v1/configuration-support/* endpoints.
- **Added** `applicationTypeSchemas[QuickApps].dial:attachmentPaths`: New field allowing DIAL Core to enforce ACL on prompt URLs in skills/validate request bodies.
- **Added** `app manifest: features.dial_files`: Per-app opt-in for the new DIAL files toolset (Preview). Exposes file operations (list, read, write, edit, delete, copy, move) against an agent_home_dir. Supports optional enabled_tools allowlist.
- **Added** `app manifest: features.stage_display.level`: New per-app field controlling which tool-execution stages surface in the DIAL UI. Values: error / info (default) / debug. debug reveals synthetic/system stages for manifest authors.
- **Added** `app manifest: features.file_loading.size_limit`: Per-app override for the file download size cap. Overrides the deployment-wide DEFAULT_FILE_LOADING_SIZE_LIMIT env var.
- **Added** `app manifest: override`: New optional field carrying a JSON Merge Patch (RFC 7396) for per-app overrides of predefined tool and toolset templates.
- **Added** `app manifest: hooks`: [Preview] New array in the app manifest for config-driven synthetic tool-call injection. Declare (ASSISTANT/tool_calls, TOOL) injections without Python code. Supports on_request_start event with always / append_if_changed frequency.
- **Added** `app manifest: dial-app toolset transport field`: New transport field on the dial-app toolset with values auto / mcp / chat-completion (default auto). Lets admins pin the routing choice rather than relying on automatic detection via features.mcp.
- **Added** `predefined content layer: default_configuration.json`: Operators can ship a default_configuration.json at each predefined content layer (built-in + PREDEFINED_EXTRA_PATHS). Layers are shallow-merged in order and exposed via the new GET /v1/configuration-support/default-configuration endpoint.

> [!NOTE]
> Time Awareness (features.timestamp) graduated to GA — no longer gated by ENABLE_PREVIEW_FEATURES — features.timestamp is now active for all deployments regardless of whether ENABLE_PREVIEW_FEATURES is set. Previously this feature was preview-only.

> [!NOTE]
> DIAL Prompt Skills (skills config field, DialPromptSkillsModule) graduated to GA — no longer gated by ENABLE_PREVIEW_FEATURES — The skills config field and DialPromptSkillsModule are now active for all deployments regardless of whether ENABLE_PREVIEW_FEATURES is set.

> [!NOTE]
> dial:applicationTypeRoutes replaces global DIAL Core routes for /v1/configuration-support/* — Configuration-support routes are now co-located with the QuickApps application type via dial:applicationTypeRoutes. The global routes entry must be removed and the new block applied. See PR #319 for the exact schema snippet.

> [!NOTE]
> DialDeploymentConfig.name and DialMCPToolSet.dial_id renamed to deployment_id; legacy keys still accepted via anyOf — The canonical field name is now deployment_id in both DialDeploymentConfig and DialMCPToolSet. Legacy keys name and dial_id continue to validate via validation_alias and the published JSON schema retains deprecated siblings under anyOf, so existing manifests remain valid without immediate changes. However operators should plan to migrate manifests before a future removal.

### ai-dial-adapter-vertexai

#### Breaking changes

**Veo: `pubsub_topic` field removed from config**

The `pubsub_topic` field has been removed from the Veo model configuration. Any deployment config that includes this field must have it removed before or during upgrade.

| Previous configuration | Required action |
|---|---|
| Veo config contains `pubsub_topic` field | Remove the `pubsub_topic` field from the Veo model configuration |

#### Config / Helm changes

- **Removed** `veo.pubsub_topic`: The `pubsub_topic` field has been removed from the Veo model configuration.

### ai-dial-adapter-openai

#### Deprecated environment variables

| Variable | Description |
|---|---|
| `DIAL_USE_FILE_STORAGE` | DIAL_USE_FILE_STORAGE is being deprecated. Based on the release notes no explicit replacement is mentioned; operators should review whether this variable is currently set and assess impact. |

**Migration:** _DIAL_USE_FILE_STORAGE is set in deployment_ → Review current usage; the variable is deprecated and may be removed in a future release. Remove or replace as directed by updated documentation.

**Migration:** _DIAL_USE_FILE_STORAGE is not set_ → No immediate action required.

### ai-dial-chat-themes

#### Config / Helm changes

- **Added** `config.json / bg-inverted`: New 'bg-inverted' property added to config.json for theme configuration.

### ai-dial-code-interpreter

> [!NOTE]
> Session ID verification added — Session ID verification has been introduced. Deployments that interact with the code interpreter should ensure session IDs are correctly passed and managed; requests with invalid or missing session IDs may now be rejected.

