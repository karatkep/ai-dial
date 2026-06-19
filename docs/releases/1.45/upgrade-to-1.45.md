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

- ai-dial-quickapps-backend 0.8.0 requires a mandatory change to ai-dial-core config: remove the legacy quick_apps2-style global routes entry for /v1/configuration-support/* and add the new dial:applicationTypeRoutes block to the QuickApps entry under applicationTypeSchemas (see PR #319 for exact snippet); failure to do so will silently break all configuration-support endpoints.
- ai-dial-admin-deployment-manager-backend 0.17.0 adds API-key authentication by validating the Api-Key header against ai-dial-core's /v1/user/info endpoint — ensure ai-dial-core is upgraded and reachable before deploying ai-dial-admin-deployment-manager-backend 0.17.0, otherwise API-key auth will fail.
- ai-dial-adapter-openai 0.40.0 deprecates DIAL_USE_FILE_STORAGE: if this was previously set to false/unset to keep DIAL storage disabled, that is no longer sufficient — storage is now auto-enabled whenever DIAL_URL is set, so remove DIAL_URL from ai-dial-adapter-openai's config if storage must remain disabled.
- ai-dial-chat-themes 0.16.0 adds new default logos and favicons for both DIAL Admin and DIAL Chat — teams using ai-dial-admin-frontend or ai-dial-chat with custom theme overrides should verify these new asset defaults do not conflict with their customisations after upgrading ai-dial-chat-themes.
- ai-dial-admin-deployment-manager-backend 0.17.0 contains multiple critical breaking changes (NODE_POOLS format overhaul, roles config removal) with a mandatory external upgrade guide — review https://github.com/epam/ai-dial-admin-deployment-manager-backend/blob/0.17.0/docs/upgrade-plans/0.17.0.md in full before upgrading any component in this release, as misconfigured NODE_POOLS or missing roles-mapping will break all managed deployments.

### ai-dial-admin-deployment-manager-backend

> [!CAUTION]
> This release includes high-priority changes. Please review the [full upgrade guide](https://github.com/epam/ai-dial-admin-deployment-manager-backend/blob/0.17.0/docs/upgrade-plans/0.17.0.md) before proceeding.

#### Breaking changes

**NODE_POOLS config format changed from label-key/capacity to explicit Kubernetes scheduling primitives; new YAML document format required**

The NODE_POOLS configuration has been replaced with a YAML document that uses explicit 'nodeSelector', 'affinity', and 'tolerations' primitives per pool. Two new create-time default fields, NODE_POOL_DEFAULT and NODE_POOL_DEFAULT_MODEL, have been added. The previous label-key/capacity format is no longer valid.

| Previous configuration | Required action |
|---|---|
| NODE_POOLS configured with label-key/capacity format | Migrate NODE_POOLS to YAML document format using explicit 'nodeSelector', 'affinity', and 'tolerations' primitives per pool. Set NODE_POOL_DEFAULT and NODE_POOL_DEFAULT_MODEL as needed for create-time defaults. |

**Removed deprecated 'config.rest.security.default.allowedRoles' and 'providers.*.allowed-roles' config properties**

The previously deprecated config properties 'config.rest.security.default.allowedRoles' and 'providers.*.allowed-roles' have been removed. Deployments must migrate to the 'roles-mapping' configuration.

| Previous configuration | Required action |
|---|---|
| 'config.rest.security.default.allowedRoles' set in configuration | Remove 'config.rest.security.default.allowedRoles' and migrate role definitions to 'roles-mapping'. |
| 'providers.*.allowed-roles' set in configuration | Remove 'providers.*.allowed-roles' entries and migrate role definitions to 'roles-mapping'. |

#### New environment variables

| Variable | Default | Required | Description |
|---|---|---|---|
| `NODE_POOL_DEFAULT` | — | No | Specifies the default node pool used at create time for deployments (introduced as part of the NODE_POOLS YAML document rework). |
| `NODE_POOL_DEFAULT_MODEL` | — | No | Specifies the default node pool used at create time for model deployments (introduced as part of the NODE_POOLS YAML document rework). |

#### Config / Helm changes

- **Removed** `config.rest.security.default.allowedRoles`: Previously deprecated property for allowed roles has been fully removed. Migrate to 'roles-mapping'.
- **Removed** `providers.*.allowed-roles`: Previously deprecated per-provider allowed-roles property has been fully removed. Migrate to 'roles-mapping'.
- **Default changed** `NODE_POOLS`: `label-key/capacity format` → `YAML document with explicit 'nodeSelector', 'affinity', and 'tolerations' primitives per pool` — The NODE_POOLS configuration format has changed. The previous label-key/capacity schema is no longer accepted; must be rewritten as a YAML document with Kubernetes scheduling primitives.
- **Added** `roles-mapping`: Replacement for the removed 'config.rest.security.default.allowedRoles' and 'providers.*.allowed-roles' properties. Role access control must now be configured here.

> [!NOTE]
> New API-key authentication via DIAL Core added alongside existing JWT/OIDC — The service now validates the 'Api-Key' header against DIAL Core's '/v1/user/info' endpoint. This is additive — existing JWT/OIDC flows are unaffected — but DIAL Core connectivity must be available if API-key auth is used.

> [!NOTE]
> Node Pool Configuration now GA with nodeSelector/affinity/tolerations per pool and create-time default stamping — Node pools are generally available. Each pool entry in the YAML document supports 'nodeSelector', 'affinity', and 'tolerations'. Create-time defaults are stamped via NODE_POOL_DEFAULT and NODE_POOL_DEFAULT_MODEL.

### ai-dial-quickapps-backend

#### Breaking changes

**DIAL Core global routes entry for /v1/configuration-support/* must be removed and replaced with dial:applicationTypeRoutes on the QuickApps application type** _(cross-component)_

The /v1/configuration-support/* endpoints are no longer served via a global DIAL Core routes entry. They are now declared on the QuickApps application type itself via dial:applicationTypeRoutes. The old global entry will be silently ignored, meaning configuration-support endpoints will stop working unless the migration is performed.

| Previous configuration | Required action |
|---|---|
| DIAL Core global routes block contains a quick_apps2-style entry for /v1/configuration-support/* | Remove the quick_apps2-style entry from DIAL Core's global routes block and add the new dial:applicationTypeRoutes block to the QuickApps entry under applicationTypeSchemas — apply the schema snippet from PR #319 verbatim |

**Time Awareness and DIAL Prompt Skills features now active regardless of ENABLE_PREVIEW_FEATURES**

features.timestamp (Time Awareness) and the skills config field / DialPromptSkillsModule (DIAL Prompt Skills) have graduated to GA and are now always active. Previously they required ENABLE_PREVIEW_FEATURES to be enabled. Deployments that relied on ENABLE_PREVIEW_FEATURES=false to suppress these features will now have them active.

| Previous configuration | Required action |
|---|---|
| ENABLE_PREVIEW_FEATURES unset or false — Time Awareness and DIAL Prompt Skills were inactive | Accept that features.timestamp and skills/DialPromptSkillsModule are now always active; review app manifests for unintended use of these fields |

#### New environment variables

| Variable | Default | Required | Description |
|---|---|---|---|
| `DEFAULT_ORCHESTRATOR_DEPLOYMENT_ID` | — | No | Default DIAL deployment id used as the orchestrator model when a QuickApp manifest omits orchestrator.deployment. Also surfaces as the JSON-schema default for that field so DIAL Core can pre-fill new manifests. Apps can override per-app. |
| `DEFAULT_FILE_LOADING_SIZE_LIMIT` | `10485760` | No | Deployment-wide cap in bytes (> 0; default 10 MiB) on files the agent downloads. Overridden per-app by features.file_loading.size_limit in the manifest. Replaces the previously hardcoded 10 MiB cap. |
| `USE_SYSTEM_CA_CERTS` | — | No | When set to 1, merges every *.crt file under /certificates/ with the Alpine system CA bundle at container startup and exports SSL_CERT_FILE to the merged path so outbound HTTP calls trust private/corporate root CAs. Opt-in; unset keeps existing behaviour. |

#### Config / Helm changes

- **Removed** `routes.<quick_apps2-style entry>`: Global DIAL Core routes entry for /v1/configuration-support/* endpoints must be removed. It will be silently ignored if left in place, causing those endpoints to become unreachable.
- **Deprecated** `DialDeploymentConfig.name` → `DialDeploymentConfig.deployment_id`: The name field in DialDeploymentConfig is deprecated in favour of deployment_id. Legacy key still validates via validation_alias and the published JSON schema retains a deprecated sibling under anyOf so existing manifests remain valid.
- **Deprecated** `DialMCPToolSet.dial_id` → `DialMCPToolSet.deployment_id`: The dial_id field in DialMCPToolSet is deprecated in favour of deployment_id. Legacy key still validates via validation_alias.
- **Deprecated** `features.stage_display (display.stage.show = false)` → `features.stage_display.level`: The legacy display.stage.show = false pattern is still honored at info level with a warning but is deprecated. Use the new features.stage_display.level (error/info/debug) instead.
- **Added** `applicationTypeSchemas.<quickapps-entry>.dial:applicationTypeRoutes`: New block required in DIAL Core's applicationTypeSchemas for the QuickApps application type to serve /v1/configuration-support/* endpoints. Replaces the former global routes entry.
- **Added** `applicationTypeSchemas.<quickapps-entry>.dial:attachmentPaths`: New field on the QuickApps application type entry letting DIAL Core enforce ACL on prompt URLs in skills/validate request bodies.

> [!NOTE]
> Custom corporate root CAs can be trusted by mounting *.crt files under /certificates/ and setting USE_SYSTEM_CA_CERTS=1 — When USE_SYSTEM_CA_CERTS=1 is set, the container merges all *.crt files under /certificates/ with the Alpine system CA bundle at startup and exports SSL_CERT_FILE. Required for deployments behind TLS-intercepting proxies.

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
| `DIAL_USE_FILE_STORAGE` | DIAL_USE_FILE_STORAGE is deprecated. DIAL Storage is now enabled automatically when DIAL_URL is set. |

**Migration:** _DIAL_USE_FILE_STORAGE=True explicitly set_ → No immediate action required; variable is deprecated but still functional. Plan to remove it in a future upgrade.

**Migration:** _DIAL_USE_FILE_STORAGE=False or unset to disable storage_ → Verify new automatic enablement behavior does not unintentionally activate storage; remove DIAL_URL if storage should stay disabled.

### ai-dial-chat-themes

#### Config / Helm changes

- **Added** `config.json / bg-inverted`: New 'bg-inverted' color property added to config.json theme configuration.

