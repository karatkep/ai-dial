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

- ai-dial-quickapps-backend 0.8.0 requires a coordinated change to ai-dial-core configuration: remove the quick_apps2-style global routes entry for /v1/configuration-support/* from DIAL Core's routes block AND add the new dial:applicationTypeRoutes block to the QuickApps entry under applicationTypeSchemas — upgrade both components together and apply the DIAL Core config change atomically with the QuickApps backend upgrade to avoid the endpoints becoming unreachable.
- ai-dial-admin-deployment-manager-backend 0.17.0 adds API-key authentication by validating the 'Api-Key' header against ai-dial-core's /v1/user/info endpoint — ensure network connectivity and valid credentials between ai-dial-admin-deployment-manager-backend and ai-dial-core are in place before upgrading.
- ai-dial-adapter-openai 0.40.0 deprecates DIAL_USE_FILE_STORAGE: DIAL Storage is now auto-enabled whenever DIAL_URL is set, which means any deployment that previously set DIAL_USE_FILE_STORAGE=false to disable storage must now either unset DIAL_URL or take explicit action to prevent storage from being auto-enabled — audit all OpenAI adapter deployments for this env var before upgrading.
- ai-dial-chat-themes 0.16.0 updates favicon and logo assets for both DIAL Admin and DIAL Chat — deployments using custom theme overrides for ai-dial-chat or ai-dial-admin-frontend should verify the new default assets do not conflict with or silently override their custom branding.
- ai-dial-admin-deployment-manager-backend 0.17.0 is a critical-severity release with multiple breaking changes (NODE_POOLS format rewrite, roles-mapping migration) and a mandatory external upgrade guide — review https://github.com/epam/ai-dial-admin-deployment-manager-backend/blob/0.17.0/docs/upgrade-plans/0.17.0.md before upgrading any other component that depends on deployment manager, and upgrade this component last after all config changes are prepared.
- ai-dial-quickapps-backend 0.8.0 graduates Time Awareness and DIAL Prompt Skills features to GA, making them unconditionally active in all QuickApps regardless of ENABLE_PREVIEW_FEATURES — review all app manifests and verify behavior in staging before upgrading production, particularly for deployments that previously relied on ENABLE_PREVIEW_FEATURES being unset to suppress these features.

### ai-dial-admin-deployment-manager-backend

> [!CAUTION]
> This release includes high-priority changes. Please review the [full upgrade guide](https://github.com/epam/ai-dial-admin-deployment-manager-backend/blob/0.17.0/docs/upgrade-plans/0.17.0.md) before proceeding.

#### Breaking changes

**NODE_POOLS config format changed: label-key/capacity replaced with explicit Kubernetes scheduling primitives; now a YAML document**

The NODE_POOLS environment variable/config previously used label-key/capacity configuration. It is now a YAML document supporting explicit 'nodeSelector', 'affinity', and 'tolerations' primitives per pool. Additionally, new create-time default fields NODE_POOL_DEFAULT and NODE_POOL_DEFAULT_MODEL have been introduced.

| Previous configuration | Required action |
|---|---|
| NODE_POOLS configured with label-key/capacity format | Rewrite NODE_POOLS as a YAML document using explicit 'nodeSelector', 'affinity', and 'tolerations' primitives per pool. Set NODE_POOL_DEFAULT and NODE_POOL_DEFAULT_MODEL as needed for create-time defaults. |

**Removed deprecated config properties 'config.rest.security.default.allowedRoles' and 'providers.*.allowed-roles'**

The previously deprecated config properties 'config.rest.security.default.allowedRoles' and 'providers.*.allowed-roles' have been fully removed. Operators must migrate to the 'roles-mapping' mechanism.

| Previous configuration | Required action |
|---|---|
| 'config.rest.security.default.allowedRoles' or 'providers.*.allowed-roles' present in configuration | Remove these properties and migrate access-control configuration to the 'roles-mapping' construct. Consult the full upgrade guide for 'roles-mapping' syntax. |

#### Environment variables with changed defaults

| Variable | Old default | New default | Description |
|---|---|---|---|
| `NODE_POOLS` | `label-key/capacity format` | `YAML document with explicit nodeSelector/affinity/tolerations primitives per pool` | NODE_POOLS must now be supplied as a YAML document. The previous label-key/capacity schema is no longer accepted. |

#### New environment variables

| Variable | Default | Required | Description |
|---|---|---|---|
| `NODE_POOL_DEFAULT` | — | No | Specifies the default node pool to stamp at create-time for resources when no explicit pool is specified. |
| `NODE_POOL_DEFAULT_MODEL` | — | No | Specifies the default node pool to stamp at create-time for model resources. |

#### Config / Helm changes

- **Removed** `config.rest.security.default.allowedRoles`: Previously deprecated property for allowed roles under the default security config. Fully removed in this release.
- **Removed** `providers.*.allowed-roles`: Previously deprecated per-provider allowed-roles property. Fully removed in this release.
- **Added** `roles-mapping`: New mechanism for access-control role configuration, replacing the removed 'allowedRoles' and 'allowed-roles' properties.

> [!NOTE]
> Node Pool Configuration GA: explicit Kubernetes scheduling primitives (nodeSelector, affinity, tolerations) per pool — Node Pool Configuration is now generally available. Each pool entry in the NODE_POOLS YAML document may define 'nodeSelector', 'affinity', and/or 'tolerations'. Create-time defaults are stamped via NODE_POOL_DEFAULT and NODE_POOL_DEFAULT_MODEL.

> [!NOTE]
> New API-key authentication via DIAL Core: 'Api-Key' header validated against Core's /v1/user/info — An additional authentication path has been added. The 'Api-Key' header is now validated against DIAL Core's '/v1/user/info' endpoint alongside the existing JWT/OIDC flow. Operators must ensure network connectivity and valid credentials between this service and DIAL Core if API-key auth is to be used.

### ai-dial-quickapps-backend

#### Breaking changes

**DIAL Core global routes entry for /v1/configuration-support/* must be removed and replaced with dial:applicationTypeRoutes on the QuickApps application type** _(cross-component)_

The /v1/configuration-support/* endpoints are no longer served via a global DIAL Core routes entry. They are now declared on the QuickApps application type via dial:applicationTypeRoutes. The old global route entry will be silently ignored, meaning the endpoints will be unreachable unless the new block is added.

| Previous configuration | Required action |
|---|---|
| DIAL Core global routes block contains a quick_apps2-style entry for /v1/configuration-support/* | Remove the quick_apps2-style entry from DIAL Core's global routes block |
| QuickApps entry under applicationTypeSchemas lacks dial:applicationTypeRoutes | Add the new dial:applicationTypeRoutes block to the QuickApps entry under applicationTypeSchemas using the schema snippet from PR #319 |

**Time Awareness and DIAL Prompt Skills features graduate to GA — now always active regardless of ENABLE_PREVIEW_FEATURES**

features.timestamp (Time Awareness) and the skills config field / DialPromptSkillsModule (DIAL Prompt Skills) are no longer gated by ENABLE_PREVIEW_FEATURES. They are active for all apps unconditionally after this upgrade.

| Previous configuration | Required action |
|---|---|
| ENABLE_PREVIEW_FEATURES was unset/false and Time Awareness / DIAL Prompt Skills were intentionally disabled | Review all app manifests — these features are now always on; ensure manifests do not inadvertently expose unwanted behavior |
| ENABLE_PREVIEW_FEATURES was set to enable these features | No functional change; verify behavior in staging before upgrading production |

#### New environment variables

| Variable | Default | Required | Description |
|---|---|---|---|
| `DEFAULT_ORCHESTRATOR_DEPLOYMENT_ID` | — | No | Default DIAL deployment id used as the orchestrator model when a QuickApp manifest omits orchestrator.deployment. Also surfaces as the JSON-schema default for that field so DIAL Core can pre-fill new manifests. Apps can override per-app. |
| `DEFAULT_FILE_LOADING_SIZE_LIMIT` | `10485760` | No | Deployment-wide cap (bytes, > 0; default 10 MiB) on files the agent downloads. Overridden per-app by features.file_loading.size_limit in the manifest. Replaces the previously hardcoded 10 MiB cap. |
| `USE_SYSTEM_CA_CERTS` | — | No | When set to 1, merges every *.crt file under /certificates/ with the Alpine system CA bundle at container startup and exports SSL_CERT_FILE to the merged path so outbound HTTP calls trust private/corporate root CAs. Opt-in; unset keeps existing behaviour. |

#### Config / Helm changes

- **Removed** `DIAL Core global routes: quick_apps2-style /v1/configuration-support/* entry`: The global DIAL Core routes entry for /v1/configuration-support/* is no longer used and must be removed. It will be silently ignored if left in place, and the endpoints will not be served unless the new dial:applicationTypeRoutes block is added.
- **Deprecated** `app manifest: DialDeploymentConfig.name` → `deployment_id`: Legacy key name in DialDeploymentConfig still validates via validation_alias and is retained in published JSON schema under anyOf, but will be removed in a future version.
- **Deprecated** `app manifest: DialMCPToolSet.dial_id` → `deployment_id`: Legacy key dial_id in DialMCPToolSet still validates via validation_alias and is retained in published JSON schema under anyOf, but will be removed in a future version.
- **Deprecated** `app manifest: display.stage.show = false` → `features.stage_display.level`: display.stage.show = false is still honored (treated as info level) with a warning, but is deprecated in favor of features.stage_display.level.
- **Added** `applicationTypeSchemas.<quickapps-entry>.dial:applicationTypeRoutes`: New block required on the QuickApps application type entry in DIAL Core's applicationTypeSchemas to serve /v1/configuration-support/* endpoints. Must be added using the schema snippet from PR #319.
- **Added** `applicationTypeSchemas.<quickapps-entry>.dial:attachmentPaths`: New field that lets DIAL Core enforce ACL on prompt URLs in skills/validate request bodies.
- **Added** `app manifest: features.dial_files`: [Preview] Per-app opt-in for new DIAL files toolset exposing list_files, read_file_lines, search_in_file, write_file, edit_file, delete_file, copy_file, move_file. Supports optional enabled_tools allowlist.
- **Added** `app manifest: features.stage_display.level`: New per-app threshold controlling which tool-execution stages surface in DIAL UI: error / info (default) / debug. Deprecates display.stage.show = false.
- **Added** `app manifest: features.file_loading.size_limit`: Per-app override for the agent file download size limit in bytes. Overrides DEFAULT_FILE_LOADING_SIZE_LIMIT env var.
- **Added** `app manifest: hooks`: [Preview] New array for config-driven synthetic tool-call injection. Declare (ASSISTANT/tool_calls, TOOL) injections without Python using on_request_start event with always / append_if_changed frequency.
- **Added** `app manifest: override`: New optional field carrying a JSON Merge Patch (RFC 7396) for per-app overrides of predefined tool and toolset templates.
- **Added** `predefined content layer: default_configuration.json`: Operators can ship a default_configuration.json at each predefined content layer (built-in + PREDEFINED_EXTRA_PATHS). Files are shallow-merged in layer order and exposed via GET /v1/configuration-support/default-configuration.
- **Added** `dial-app toolset: transport`: New transport field (auto | mcp | chat-completion, default auto) on dial-app toolset config. Allows admins to pin routing to MCP or chat completion instead of auto-detecting via features.mcp.

> [!NOTE]
> Time Awareness and DIAL Prompt Skills are now GA — active unconditionally — features.timestamp (Time Awareness) and the skills config field / DialPromptSkillsModule (DIAL Prompt Skills) are no longer gated by ENABLE_PREVIEW_FEATURES. Any deployment that previously relied on ENABLE_PREVIEW_FEATURES being unset to suppress these features will see them active after upgrade.

> [!NOTE]
> DIAL Core route config restructured: /v1/configuration-support/* moves from global routes to dial:applicationTypeRoutes — Operators must remove the quick_apps2-style global route and add dial:applicationTypeRoutes to the QuickApps applicationTypeSchemas entry. The dial:attachmentPaths field also enables DIAL Core ACL enforcement on prompt URLs in skills/validate requests.

> [!NOTE]
> New USE_SYSTEM_CA_CERTS option for TLS-intercepting proxy environments — When USE_SYSTEM_CA_CERTS=1, the container merges *.crt files mounted under /certificates/ with the Alpine system CA bundle at startup and sets SSL_CERT_FILE. Required for deployments behind corporate TLS-intercepting proxies.

> [!NOTE]
> DialDeploymentConfig.name and DialMCPToolSet.dial_id renamed to deployment_id; legacy keys deprecated — Existing manifests using name or dial_id continue to validate via validation_alias and the published JSON schema retains deprecated siblings under anyOf. Operators should migrate manifests to use deployment_id to avoid breakage when the legacy keys are removed in a future version.

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
> WIF support added for AWS container credential providers — Workload Identity Federation (WIF) support has been added for AWS container credential providers. Deployments that use AWS-based credentials with VertexAI may now leverage WIF-based authentication.

### ai-dial-adapter-openai

#### Deprecated environment variables

| Variable | Description |
|---|---|
| `DIAL_USE_FILE_STORAGE` | DIAL_USE_FILE_STORAGE is deprecated. DIAL Storage is now enabled automatically when DIAL_URL is set. |

**Migration:** _DIAL_USE_FILE_STORAGE explicitly set to enable storage_ → Remove DIAL_USE_FILE_STORAGE; ensure DIAL_URL is set if storage should remain enabled

**Migration:** _DIAL_USE_FILE_STORAGE explicitly set to disable storage_ → Review new behavior — storage may now be auto-enabled when DIAL_URL is set; unset DIAL_URL or take other steps if storage must stay disabled

### ai-dial-chat-themes

#### Config / Helm changes

- **Added** `config.json / bg-inverted`: New 'bg-inverted' color property added to config.json theme configuration.

