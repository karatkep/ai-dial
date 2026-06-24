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
   - ai-dial-core: `0.45.0-rc.0`
   - ai-dial-analytics-realtime: `0.24.2`
   - ai-dial-rag: `0.42.0`
   - ai-dial-log-parser: `0.3.0`
   - ai-dial-code-interpreter: `0.2.0`
   - ai-dial-app-controller: `0.4.0`
   - ai-dial-app-builder-python: `0.1.0`
   - ai-dial-quickapps-backend: `0.9.0-rc.1`
   - ai-dial-mind-map-backend: `0.14.1`
   - ai-dial-mind-map-frontend: `0.13.0`
   - ai-dial-admin-backend: `0.18.0-rc.0`
   - ai-dial-admin-frontend: `0.18.0-rc.0`
   - ai-dial-admin-deployment-manager-backend: `0.18.0-rc.0`
   - ai-dial-admin-evaluation-framework-backend: `0.1.0-rc.0`

## Before upgrade

### General notes

- Please review the [Config changes](#config-changes) chapter carefully for each component that is used in your DIAL installation. Changes in components' configuration may be required.
- Please check if any image tag overrides (`image.tag`) are present and remove them if they are not required anymore.
- Please check and add `image.repository` to change the image location for `redis`, `postgresql`, `keycloak` and `keycloakConfigCli` components to start using alternative Docker registries (e.g. Amazon ECR Public Gallery) if required.

### Release-specific notes

#### ai-dial-admin-evaluation-framework-backend `0.1.0-rc.0`

##### Breaking changes

**EvalSummary CSV column-group separator changed from `:` to `::`**

Any downstream consumer that parses exported CSV headers by splitting on `:` will break. Column names like `data:prompt` are now `data::prompt`; `metric:Accuracy:score` is now `metric::Accuracy::score`.

| Previous configuration | Required action |
|---|---|
| CSV header parser splits on single `:` (e.g. `data:prompt`, `metric:Accuracy:score`) | Update parser to split on `::` instead of `:` to match new separator (e.g. `data::prompt`, `metric::Accuracy::score`) |

##### New environment variables

| Variable | Default | Required | Description |
|---|---|---|---|
| `TEST_CASE_BULK_MAX_DELETE_IDS` | `10000` | No | Maximum number of IDs accepted in a single bulk-delete-by-IDs request (`DELETE /test-cases:bulk`). Must be ≥ 1. |

---

#### ai-dial-adapter-vertexai `0.36.0`

##### Breaking changes

**Veo: `pubsub_topic` field removed from config**

The `pubsub_topic` field has been removed from the Veo model configuration. Any existing config that includes this field must be updated to remove it.

| Previous configuration | Required action |
|---|---|
| Veo config contains `pubsub_topic` field | Remove the `pubsub_topic` field from the Veo model configuration before upgrading |

##### Config / Helm changes

- **Removed** `veo.pubsub_topic`: The `pubsub_topic` field has been removed from the Veo model configuration.

---

#### ai-dial-admin-backend `0.18.0-rc.0`

##### Breaking changes

**Flat `applicationTypeSchemaId` field replaced by polymorphic `source` field on ApplicationResourceDto and CreateApplicationResourceDto**

The `applicationTypeSchemaId` field has been removed from `ApplicationResourceDto` and `CreateApplicationResourceDto`. It is replaced by a `source` field that is a `$type`-discriminated polymorphic object supporting `schema` and `endpoints` variants. Any API clients, automation, or config payloads that set or read `applicationTypeSchemaId` must be updated to use the new `source` structure.

| Previous configuration | Required action |
|---|---|
| API payloads use flat `applicationTypeSchemaId` field on ApplicationResourceDto / CreateApplicationResourceDto | Replace `applicationTypeSchemaId` with the polymorphic `source` object (discriminated by `$type`, variants: `schema` and `endpoints`) in all API calls, integrations, and stored payloads |

##### Config / Helm changes

- **Default changed** `applicationProperties`: `unset / null` → `empty map `{}`` — The `applicationProperties` field for application assets now defaults to an empty map instead of being absent/null.
- **Added** `features.maxTokensSupported`: New configuration property introduced in DIAL Core v0.45.0. Defaults to `true`.
- **Added** `features.maxCompletionTokensSupported`: New configuration property introduced in DIAL Core v0.45.0.
- **Added** `features.customTemperatureSupported`: New configuration property introduced in DIAL Core v0.45.0. Defaults to `true`.
- **Added** `features.reasoningEfforts`: New configuration property introduced in DIAL Core v0.45.0.
- **Added** `upstreams.secretExtraData`: New configuration property for upstreams introduced in DIAL Core v0.45.0.
- **Added** `models.embeddingDimensions`: New configuration property for models introduced in DIAL Core v0.45.0.

---

#### ai-dial-admin-deployment-manager-backend `0.18.0-rc.0`

##### Breaking changes

**Spring Boot upgraded to 4.x**

The application framework has been upgraded to Spring Boot 4.x. This is a major version bump that may affect configuration property names, actuator endpoints, security defaults, and other Spring Boot-managed behaviors. Review Spring Boot 4.x migration guide for any incompatible configuration or property changes.

| Previous configuration | Required action |
|---|---|
| Running with Spring Boot 3.x defaults and configuration | Review Spring Boot 4.x migration guide; audit application.properties/application.yaml and any Spring Boot-related env vars for renamed or removed properties before upgrading |

---

#### ai-dial-quickapps-backend `0.9.0-rc.1`

##### Breaking changes

**DIAL files tools graduated to GA — now active regardless of ENABLE_PREVIEW_FEATURES**

The tools list/read_lines/search/find/write/edit/delete/copy/move and the features.dial_files config field are no longer gated by ENABLE_PREVIEW_FEATURES. Any deployment that previously relied on ENABLE_PREVIEW_FEATURES=false to suppress these tools will find them active after upgrade. Only tool_call_result_offload (features.dial_files.tool_call_result_offload and its TOOL_CALL_RESULT_OFFLOAD__* env defaults) remains behind the preview flag.

| Previous configuration | Required action |
|---|---|
| ENABLE_PREVIEW_FEATURES=false; DIAL files tools were inactive | After upgrade, DIAL files tools will be active. If you need to suppress them, disable via features.dial_files config rather than relying on the preview flag. |
| ENABLE_PREVIEW_FEATURES=true; DIAL files tools were active | No action required; behavior unchanged. |

##### Config / Helm changes

- **Added** `features.dial_files`: Config field for DIAL files tools is now GA and active regardless of ENABLE_PREVIEW_FEATURES. Previously only effective when ENABLE_PREVIEW_FEATURES was enabled.

---

#### ai-dial-adapter-openai `0.40.0`

##### Deprecated environment variables

| Variable | Description |
|---|---|
| `DIAL_USE_FILE_STORAGE` | DIAL_USE_FILE_STORAGE is deprecated. No replacement explicitly mentioned in release notes. |

**Migration:** _DIAL_USE_FILE_STORAGE is set in deployment_ → Plan to remove DIAL_USE_FILE_STORAGE; review updated README/documentation for new file storage configuration behavior

---

#### ai-dial-chat-themes `0.16.0`

##### Config / Helm changes

- **Added** `config.json / bg-inverted`: New 'bg-inverted' color property added to config.json theme configuration.

---

#### ai-dial-core `0.45.0-rc.0`

##### Config / Helm changes

- **Added** `features.reasoningEffortsSupported`: New feature flag to indicate whether reasoning efforts are supported by a model/deployment.
- **Added** `features.max_tokens / features.max_completion_tokens / features.temperature`: New feature flags to expose max_tokens, max_completion_tokens, and temperature capabilities in model listings.
- **Added** `features (available endpoints flags)`: New flags added to expose available endpoints in feature listings.
- **Added** `dial-unified-config (Configuration API / MergedConfigStore / secret encryption)`: New server-side unified configuration API introduced, including a MergedConfigStore and secret encryption support.
- **Added** `roles.readonly-admin`: New readonly-admin role introduced to allow reading user data without write access.

---
