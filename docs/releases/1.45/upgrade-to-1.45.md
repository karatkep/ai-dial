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

- ai-dial-quickapps-backend 0.8.0 requires a coordinated change to ai-dial-core config: remove the global `routes` entry for `/v1/configuration-support/*` (e.g. `quick_apps2`-style entry) and add `dial:applicationTypeRoutes` to the QuickApps entry under `applicationTypeSchemas`; retrieve the exact schema snippet from https://github.com/epam/ai-dial-quickapps-backend/pull/319 before upgrading either component.
- Upgrade ai-dial-core BEFORE or simultaneously with ai-dial-quickapps-backend 0.8.0, since the new `dial:applicationTypeRoutes` mechanism depends on Core support for type-scoped routes; applying the QuickApps upgrade without the Core config change will make `/v1/configuration-support/*` endpoints unreachable.
- ai-dial-admin-deployment-manager-backend 0.17.0 adds API-key authentication by calling ai-dial-core's `/v1/user/info` endpoint; ensure ai-dial-core is reachable from the deployment manager and that the Core version in this release supports that endpoint before upgrading the deployment manager.
- ai-dial-admin-deployment-manager-backend 0.17.0 has a mandatory external upgrade guide at https://github.com/epam/ai-dial-admin-deployment-manager-backend/blob/0.17.0/docs/upgrade-plans/0.17.0.md covering critical NODE_POOLS config format changes and removed role config properties; review and execute that guide before deploying any other component that depends on node pool scheduling.
- ai-dial-adapter-openai 0.40.0 deprecates `DIAL_USE_FILE_STORAGE` and now auto-enables DIAL Storage when `DIAL_URL` is set; if any environment previously set `DIAL_USE_FILE_STORAGE=False` alongside `DIAL_URL` to intentionally disable storage, that suppression will no longer work after this upgrade and storage will activate automatically.
- ai-dial-chat-themes 0.16.0 adds default logos and favicons for both DIAL Chat and DIAL Admin; teams managing ai-dial-chat or ai-dial-admin-frontend with custom favicon/logo overrides should verify those overrides still take precedence after upgrading themes to avoid unintended asset replacement.

### ai-dial-admin-deployment-manager-backend

> [!CAUTION]
> This release includes high-priority changes. Please review the [full upgrade guide](https://github.com/epam/ai-dial-admin-deployment-manager-backend/blob/0.17.0/docs/upgrade-plans/0.17.0.md) before proceeding.

#### Breaking changes

**NODE_POOLS config replaced: label-key/capacity format replaced with explicit Kubernetes scheduling primitives; format changed to YAML document**

The previous NODE_POOLS configuration using node pool label-key/capacity style has been replaced. NODE_POOLS is now a YAML document supporting explicit 'nodeSelector', 'affinity', and 'tolerations' primitives per pool. Two new create-time default fields, NODE_POOL_DEFAULT and NODE_POOL_DEFAULT_MODEL, have been introduced.

| Previous configuration | Required action |
|---|---|
| NODE_POOLS configured with label-key/capacity style config | Migrate NODE_POOLS to the new YAML document format using explicit 'nodeSelector', 'affinity', and 'tolerations' primitives. Set NODE_POOL_DEFAULT and NODE_POOL_DEFAULT_MODEL as needed for create-time defaults. |

**Removed deprecated config properties 'config.rest.security.default.allowedRoles' and 'providers.*.allowed-roles'; must migrate to 'roles-mapping'**

The previously deprecated configuration properties 'config.rest.security.default.allowedRoles' and 'providers.*.allowed-roles' have been removed entirely. Deployments still using these properties must migrate to the 'roles-mapping' configuration.

| Previous configuration | Required action |
|---|---|
| 'config.rest.security.default.allowedRoles' or 'providers.*.allowed-roles' present in configuration | Remove 'config.rest.security.default.allowedRoles' and 'providers.*.allowed-roles' from configuration and migrate role definitions to 'roles-mapping'. |

#### New environment variables

| Variable | Default | Required | Description |
|---|---|---|---|
| `NODE_POOL_DEFAULT` | — | No | Specifies the default node pool to stamp at create time when no explicit pool is selected. Part of the new NODE_POOLS YAML document configuration. |
| `NODE_POOL_DEFAULT_MODEL` | — | No | Specifies the default node pool for model deployments at create time. Part of the new NODE_POOLS YAML document configuration. |

#### Config / Helm changes

- **Removed** `config.rest.security.default.allowedRoles`: Previously deprecated property fully removed. Migrate to 'roles-mapping'.
- **Removed** `providers.*.allowed-roles`: Previously deprecated property fully removed. Migrate to 'roles-mapping'.
- **Added** `node-pools[*].nodeSelector`: Explicit Kubernetes nodeSelector primitive per node pool, replacing the previous label-key/capacity config style.
- **Added** `node-pools[*].affinity`: Explicit Kubernetes affinity primitive per node pool, replacing the previous label-key/capacity config style.
- **Added** `node-pools[*].tolerations`: Explicit Kubernetes tolerations primitive per node pool, replacing the previous label-key/capacity config style.

> [!NOTE]
> New API-key authentication via DIAL Core added alongside existing JWT/OIDC — The service now validates the 'Api-Key' header against DIAL Core's '/v1/user/info' endpoint. This is an additive change alongside the existing JWT/OIDC flow; however, operators should verify DIAL Core connectivity and ensure the Core endpoint is reachable from this service.

> [!NOTE]
> Node Pool Configuration is now Generally Available with Kubernetes scheduling primitives — Node Pool Configuration graduates from preview/experimental to GA. Operators must migrate existing NODE_POOLS configuration from the old label-key/capacity format to the new YAML document format using nodeSelector, affinity, and tolerations. The full migration procedure is in the external upgrade guide.

### ai-dial-quickapps-backend

#### Breaking changes

**DIAL Core global `routes` entry for `/v1/configuration-support/*` must be removed and replaced with `dial:applicationTypeRoutes` on the QuickApps application type** _(cross-component)_

The `/v1/configuration-support/*` endpoints are no longer served via a global DIAL Core `routes` entry. They are now declared on the QuickApps application type itself via `dial:applicationTypeRoutes`. Failing to migrate will cause these endpoints to be unreachable or incorrectly routed.

| Previous configuration | Required action |
|---|---|
| DIAL Core global `routes` block contains a `quick_apps2`-style entry for `/v1/configuration-support/*` | Remove the `quick_apps2`-style entry from DIAL Core's global `routes` block, then add the new `dial:applicationTypeRoutes` block to the QuickApps entry under `applicationTypeSchemas` using the schema snippet from PR #319 |

#### New environment variables

| Variable | Default | Required | Description |
|---|---|---|---|
| `DEFAULT_ORCHESTRATOR_DEPLOYMENT_ID` | — | No | Default DIAL deployment id used as the orchestrator model when a QuickApp manifest omits `orchestrator.deployment`. Also surfaces as the JSON-schema `default` for that field so DIAL Core can pre-fill new manifests. Apps can override per-app. |
| `DEFAULT_FILE_LOADING_SIZE_LIMIT` | `10485760` | No | Deployment-wide cap (bytes, `> 0`; default 10 MiB) on files the agent downloads. Overridden per-app by `features.file_loading.size_limit` in the manifest. Replaces previously hardcoded 10 MiB limit. |
| `USE_SYSTEM_CA_CERTS` | — | No | When set to `1`, merges every `*.crt` file under `/certificates/` with the Alpine system CA bundle at container startup and exports `SSL_CERT_FILE` to the merged path so outbound HTTP calls trust private/corporate root CAs. Opt-in; unset keeps existing behaviour. |

#### Config / Helm changes

- **Removed** `DIAL Core global `routes` entry for `/v1/configuration-support/*` (e.g. `quick_apps2`-style entry)`: Must be removed from DIAL Core's global `routes` block. It will be ignored from now on and functionality moves to `dial:applicationTypeRoutes` on the application type.
- **Deprecated** `app manifest: DialDeploymentConfig.name` → `deployment_id`: Legacy key `name` on `DialDeploymentConfig` is deprecated. Still accepted via `validation_alias` and retained in published JSON schema under `anyOf`. Will be removed in a future version.
- **Deprecated** `app manifest: DialMCPToolSet.dial_id` → `deployment_id`: Legacy key `dial_id` on `DialMCPToolSet` is deprecated. Still accepted via `validation_alias` and retained in published JSON schema under `anyOf`. Will be removed in a future version.
- **Deprecated** `app manifest: display.stage.show = false` → `features.stage_display.level`: The `display.stage.show = false` config is deprecated. Still honored at `info` level with a warning. Use `features.stage_display.level` instead.
- **Added** `applicationTypeSchemas[QuickApps].dial:applicationTypeRoutes`: New block required on the QuickApps application type entry in DIAL Core's `applicationTypeSchemas`. Replaces the previously global `routes` entry for `/v1/configuration-support/*`. Schema snippet must be applied from PR #319.
- **Added** `applicationTypeSchemas[QuickApps].dial:attachmentPaths`: New field on the QuickApps application type that lets DIAL Core enforce ACL on prompt URLs in `skills/validate` request bodies.
- **Added** `app manifest: hooks[]`: [Preview] New `hooks` array in app manifest for config-driven synthetic tool-call injection. Supports `on_request_start` event with `always` / `append_if_changed` frequency. Declares `(ASSISTANT/tool_calls, TOOL)` injections without writing Python.
- **Added** `app manifest: features.dial_files`: [Preview] New per-app opt-in field to enable DIAL files toolset. Accepts optional `enabled_tools` allowlist. Exposes tools: `list_files`, `read_file_lines`, `search_in_file`, `write_file`, `edit_file`, `delete_file`, `copy_file`, `move_file` against `agent_home_dir` (default `files/{appdata}/`).
- **Added** `app manifest: features.stage_display.level`: New per-app field for tool-execution stage display threshold: `error` / `info` (default) / `debug`. The `debug` level reveals synthetic/system stages. Deprecates `display.stage.show = false`.
- **Added** `app manifest: features.file_loading.size_limit`: New per-app override (bytes) for the agent file-download size limit. Overrides `DEFAULT_FILE_LOADING_SIZE_LIMIT` env var for the specific app.
- **Added** `app manifest: override`: New optional `override` field carrying a JSON Merge Patch (RFC 7396) for per-app overrides of predefined tool and toolset templates.
- **Added** `app manifest: dial-app toolset transport field`: New `transport: "auto" | "mcp" | "chat-completion"` field (default `"auto"`) on the `dial-app` toolset. Admins can pin MCP or chat-completion routing; `"auto"` detects via `features.mcp == true` on the deployment.
- **Added** `predefined content layer: default_configuration.json`: Operators can now ship a `default_configuration.json` at each predefined content layer (built-in + `PREDEFINED_EXTRA_PATHS`). Files are shallow-merged in layer order and exposed via the new `GET /v1/configuration-support/default-configuration` endpoint.

> [!NOTE]
> Time Awareness (`features.timestamp`) graduated to GA — no longer gated by `ENABLE_PREVIEW_FEATURES` — Previously required `ENABLE_PREVIEW_FEATURES` to be active. Now enabled unconditionally for all apps using `features.timestamp`. Operators relying on the preview gate to suppress this feature must find an alternative.

> [!NOTE]
> DIAL Prompt Skills (`skills` config field, `DialPromptSkillsModule`) graduated to GA — no longer gated by `ENABLE_PREVIEW_FEATURES` — Previously required `ENABLE_PREVIEW_FEATURES` to be active. Now enabled unconditionally. Operators relying on the preview gate to suppress this feature must find an alternative.

> [!NOTE]
> `dial:applicationTypeRoutes` replaces global DIAL Core route for `/v1/configuration-support/*`; `dial:attachmentPaths` added for ACL enforcement — Type-scoped DIAL Core routes are now co-located with the QuickApps application type. The global route entry must be removed and `dial:applicationTypeRoutes` added to the application type schema. Additionally `dial:attachmentPaths` enables DIAL Core ACL enforcement on prompt URLs in `skills/validate` requests.

> [!NOTE]
> Field renaming: `DialDeploymentConfig.name` → `deployment_id`, `DialMCPToolSet.dial_id` → `deployment_id`; legacy keys still valid via `anyOf` — Existing manifests using the old keys continue to validate and function. Operators should plan migration of stored manifests to the new canonical field names before legacy keys are removed in a future release.

### ai-dial-adapter-vertexai

#### Breaking changes

**Veo: `pubsub_topic` field removed from config**

The `pubsub_topic` field has been removed from the Veo model configuration. Any deployment config that includes this field must be updated to remove it.

| Previous configuration | Required action |
|---|---|
| Veo config includes `pubsub_topic` field | Remove `pubsub_topic` from the Veo model configuration |

#### Config / Helm changes

- **Removed** `veo.pubsub_topic`: The `pubsub_topic` field has been removed from the Veo model configuration.

### ai-dial-adapter-openai

#### Deprecated environment variables

| Variable | Description |
|---|---|
| `DIAL_USE_FILE_STORAGE` | DIAL_USE_FILE_STORAGE is deprecated. DIAL Storage is now enabled automatically when DIAL_URL is set. |

**Migration:** _DIAL_USE_FILE_STORAGE=True explicitly set_ → No immediate action required; behavior is unchanged, but plan to remove the variable as it will be removed in a future release

**Migration:** _DIAL_USE_FILE_STORAGE=False or unset with DIAL_URL set_ → Review whether storage should remain disabled; if so, ensure removal of DIAL_URL or await updated guidance before next upgrade

### ai-dial-chat-themes

#### Config / Helm changes

- **Added** `config.json / bg-inverted`: New 'bg-inverted' property added to config.json for theme configuration.

### ai-dial-code-interpreter

> [!NOTE]
> Session ID verification added — Session ID verification has been introduced. If your deployment relies on any client behaviour that bypasses session validation, this may require testing.

