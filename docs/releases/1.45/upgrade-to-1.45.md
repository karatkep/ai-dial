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

- ai-dial-quickapps-backend 0.8.0 has a CRITICAL cross-component change: the `/v1/configuration-support/*` global route entry must be REMOVED from ai-dial-core's `routes` block AND a new `dial:applicationTypeRoutes` block must be ADDED to the QuickApps entry under ai-dial-core's `applicationTypeSchemas` — retrieve the exact schema snippet from https://github.com/epam/ai-dial-quickapps-backend/pull/319 before upgrading.
- ai-dial-admin-deployment-manager-backend 0.17.0 adds API-key authentication by validating the `Api-Key` header against ai-dial-core's `/v1/user/info` endpoint — ensure ai-dial-core is reachable from ai-dial-admin-deployment-manager-backend before or during upgrade, and that ai-dial-core's API-key configuration is consistent.
- ai-dial-admin-deployment-manager-backend 0.17.0 has a mandatory external upgrade guide at https://github.com/epam/ai-dial-admin-deployment-manager-backend/blob/0.17.0/docs/upgrade-plans/0.17.0.md covering critical breaking changes (NODE_POOLS format rewrite, roles-mapping migration) — DO NOT upgrade this component without reviewing that guide first.
- ai-dial-chat-themes 0.16.0 ships new default logos and favicons for both DIAL Admin and DIAL Chat — verify these do not conflict with existing custom theme overrides in ai-dial-chat and ai-dial-admin-frontend deployments.
- ai-dial-adapter-openai 0.40.0 deprecates `DIAL_USE_FILE_STORAGE`; the replacement behavior (likely auto-enabled when `DIAL_URL` is set) is not fully documented in release notes — consult PR #445 in the ai-dial-adapter-openai repo before removing this variable to confirm DIAL Core storage connectivity is not silently broken.
- ai-dial-adapter-vertexai 0.36.0 now supports Mistral models — if routing Mistral through VertexAI, ensure ai-dial-core's model routing configuration is updated to reference the new VertexAI adapter endpoints for those model IDs.
- ai-dial-mind-map-backend 0.14.1 references a new `public_url` feature whose configuration surface (env var, config key, or Helm value) is undocumented in release notes — verify whether this requires a corresponding URL entry in ai-dial-core's routing or gateway config before deploying.

### ai-dial-admin-deployment-manager-backend

> [!CAUTION]
> This release includes high-priority changes. Please review the [full upgrade guide](https://github.com/epam/ai-dial-admin-deployment-manager-backend/blob/0.17.0/docs/upgrade-plans/0.17.0.md) before proceeding.

#### Breaking changes

**NODE_POOLS config format changed to YAML document with explicit Kubernetes scheduling primitives; new NODE_POOL_DEFAULT and NODE_POOL_DEFAULT_MODEL create-time defaults added**

The previous node pool configuration based on label-key/capacity config has been replaced. NODE_POOLS must now be expressed as a YAML document containing explicit nodeSelector, affinity, and tolerations primitives per pool. Two new variables, NODE_POOL_DEFAULT and NODE_POOL_DEFAULT_MODEL, are added for create-time default stamping.

| Previous configuration | Required action |
|---|---|
| NODE_POOLS configured with label-key/capacity format | Rewrite NODE_POOLS as a YAML document using explicit nodeSelector, affinity, and tolerations fields per pool. Configure NODE_POOL_DEFAULT and NODE_POOL_DEFAULT_MODEL as needed. |

**Removed deprecated config properties 'config.rest.security.default.allowedRoles' and 'providers.*.allowed-roles'; must migrate to 'roles-mapping'**

The previously deprecated config keys config.rest.security.default.allowedRoles and providers.*.allowed-roles have been fully removed. Deployments still using these keys must migrate to the roles-mapping configuration before upgrading.

| Previous configuration | Required action |
|---|---|
| config.rest.security.default.allowedRoles set in config | Remove config.rest.security.default.allowedRoles and replace with equivalent roles-mapping configuration. |
| providers.*.allowed-roles set in config | Remove providers.*.allowed-roles and replace with equivalent roles-mapping configuration. |

#### New environment variables

| Variable | Default | Required | Description |
|---|---|---|---|
| `NODE_POOL_DEFAULT` | — | No | Specifies the default node pool to stamp at create time for general workloads, introduced as part of the new YAML-document NODE_POOLS configuration. |
| `NODE_POOL_DEFAULT_MODEL` | — | No | Specifies the default node pool to stamp at create time for model workloads, introduced as part of the new YAML-document NODE_POOLS configuration. |

#### Config / Helm changes

- **Removed** `config.rest.security.default.allowedRoles`: Previously deprecated config property fully removed. Must migrate to roles-mapping.
- **Removed** `providers.*.allowed-roles`: Previously deprecated config property fully removed. Must migrate to roles-mapping.
- **Default changed** `NODE_POOLS`: `label-key/capacity config format` → `YAML document with explicit nodeSelector, affinity, and tolerations primitives per pool` — NODE_POOLS must be rewritten in the new YAML document format. The old label-key/capacity format is no longer accepted.

> [!NOTE]
> API-key authentication via DIAL Core added alongside existing JWT/OIDC flow — A new authentication path validates the Api-Key HTTP header against DIAL Core's /v1/user/info endpoint. This supplements rather than replaces the existing JWT/OIDC flow. Operators using API-key auth must ensure DIAL Core connectivity is available from this service.

> [!NOTE]
> Node Pool Configuration now GA with explicit Kubernetes scheduling primitives — nodeSelector, affinity, and tolerations are now first-class fields per node pool in the YAML document. This is generally available as of this release.

### ai-dial-quickapps-backend

#### Breaking changes

**DIAL Core global routes entry for `/v1/configuration-support/*` must be removed and replaced with `dial:applicationTypeRoutes` on the QuickApps application type** _(cross-component)_

The `/v1/configuration-support/*` endpoints are no longer served via a global DIAL Core `routes` entry. They are now declared on the QuickApps application type itself via `dial:applicationTypeRoutes`. Failing to migrate will leave the old route block dangling (ignored) and the new endpoints unreachable unless the new block is added.

| Previous configuration | Required action |
|---|---|
| DIAL Core global `routes` block contains a `quick_apps2`-style entry for `/v1/configuration-support/*` | Remove the `quick_apps2`-style entry from DIAL Core's global `routes` block |
| QuickApps entry under `applicationTypeSchemas` lacks `dial:applicationTypeRoutes` | Add the new `dial:applicationTypeRoutes` block to the QuickApps entry under `applicationTypeSchemas` using the schema snippet from PR #319 |

#### New environment variables

| Variable | Default | Required | Description |
|---|---|---|---|
| `DEFAULT_ORCHESTRATOR_DEPLOYMENT_ID` | — | No | Default DIAL deployment id used as the orchestrator model when a QuickApp manifest omits `orchestrator.deployment`. Also surfaces as the JSON-schema `default` for that field so DIAL Core can pre-fill new manifests. Apps can override per-app. |
| `DEFAULT_FILE_LOADING_SIZE_LIMIT` | `10485760` | No | Deployment-wide cap (bytes, > 0; default 10 MiB) on files the agent downloads. Overridden per-app by `features.file_loading.size_limit` in the manifest. Previously this was a hardcoded 10 MiB cap. |
| `USE_SYSTEM_CA_CERTS` | — | No | When set to `1`, merges every `*.crt` file under `/certificates/` with the Alpine system CA bundle at container startup and exports `SSL_CERT_FILE` to the merged path so outbound HTTP calls trust private/corporate root CAs. Opt-in; unset keeps existing behaviour. |

#### Config / Helm changes

- **Removed** `routes[quick_apps2] (DIAL Core global routes block)`: The `quick_apps2`-style global route entry for `/v1/configuration-support/*` must be removed from DIAL Core's global `routes` block; it will be ignored from now on and replaced by `dial:applicationTypeRoutes`.
- **Deprecated** `DialDeploymentConfig.name` → `DialDeploymentConfig.deployment_id`: The `name` field in deployment config is deprecated in favour of `deployment_id`. Still accepted via `validation_alias`; will be removed in a future version. The published JSON schema retains deprecated siblings under an `anyOf` so existing manifests stay valid.
- **Deprecated** `DialMCPToolSet.dial_id` → `DialMCPToolSet.deployment_id`: The `dial_id` field in MCP toolset config is deprecated in favour of `deployment_id`. Still accepted via `validation_alias`; will be removed in a future version.
- **Deprecated** `app manifest: display.stage.show` → `features.stage_display.level`: The `display.stage.show = false` pattern is deprecated in favour of the new per-app `features.stage_display.level` field (values: `error` / `info` / `debug`). The old value is still honored at `info` level with a warning.
- **Added** `applicationTypeSchemas[QuickApps].dial:applicationTypeRoutes`: New block required on the QuickApps application type entry in DIAL Core's `applicationTypeSchemas` to serve `/v1/configuration-support/*` endpoints. Must be added using the schema snippet from PR #319.
- **Added** `applicationTypeSchemas[QuickApps].dial:attachmentPaths`: New field on the QuickApps application type entry allowing DIAL Core to enforce ACL on prompt URLs in `skills/validate` request bodies.

> [!NOTE]
> Time Awareness and DIAL Prompt Skills features graduated to GA — now always active — Both features are no longer gated by `ENABLE_PREVIEW_FEATURES`. `features.timestamp` (Time Awareness) and the `skills` config field / `DialPromptSkillsModule` (DIAL Prompt Skills) are now active for all deployments regardless of the preview flag.

### ai-dial-adapter-vertexai

#### Breaking changes

**Veo: `pubsub_topic` field removed from config**

The `pubsub_topic` field has been removed from the Veo model configuration. Any deployment config referencing this field must be updated.

| Previous configuration | Required action |
|---|---|
| Veo config contains `pubsub_topic` field | Remove `pubsub_topic` from the Veo model configuration |

#### Config / Helm changes

- **Removed** `veo.pubsub_topic`: The `pubsub_topic` field has been removed from the Veo model configuration.

### ai-dial-adapter-openai

#### Deprecated environment variables

| Variable | Description |
|---|---|
| `DIAL_USE_FILE_STORAGE` | DIAL_USE_FILE_STORAGE is deprecated. Based on the deprecation pattern, DIAL Storage is likely now enabled automatically when DIAL_URL is set. |

**Migration:** _DIAL_USE_FILE_STORAGE explicitly set_ → Plan to remove DIAL_USE_FILE_STORAGE from deployment config; review adapter documentation for new storage enablement behavior

### ai-dial-chat-themes

#### Config / Helm changes

- **Added** `config.json / bg-inverted`: New 'bg-inverted' color property added to config.json theme configuration.

