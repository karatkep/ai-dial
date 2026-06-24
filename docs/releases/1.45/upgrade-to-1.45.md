# Instructions

## Versions

1. Helm chart versions:
   - dial: `6.4.0`
   - dial-core: `6.0.0`
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

#### ai-dial-admin-deployment-manager-backend `0.18.0-rc.0`

> [!CAUTION]
> This release includes high-priority changes. Please review the [full upgrade guide](https://github.com/epam/ai-dial-admin-deployment-manager-backend/blob/0.18.0-rc.0/docs/upgrade-plans/0.18.0.md) before proceeding.

##### Breaking changes

**Spring Boot upgraded to 4.0.6 (Spring Framework 7, Hibernate 7.2) with Jackson 3 JSON serialization**

Major framework version bump. Jackson 3 migration is known to have caused at least one regression (image entrypoint/cmd binding). Deployments relying on existing serialization behavior should validate after upgrade.

| Previous configuration | Required action |
|---|---|
| Running on Spring Boot 3.5 with Jackson 2 | Review upgrade guide; validate JSON serialization behavior for all payloads, particularly image entrypoint/cmd fields (regression fixed in #367 but other edge cases may exist) |

**OpenTelemetry env vars renamed/removed and default changed: OTEL_SDK_DISABLED replaced by OTEL_EXPORT_ENABLED (inverted logic), OTEL_EXPORTER_OTLP_PROTOCOL replaced by OTEL_EXPORTER_OTLP_TRANSPORT, OTEL_EXPORTER_OTLP_HEADERS removed**

OTel export is now off by default. Three env var changes with inverted semantics on the main toggle. Existing telemetry exporters will stop silently if not migrated.

| Previous configuration | Required action |
|---|---|
| OTEL_SDK_DISABLED=false (telemetry enabled) | Remove OTEL_SDK_DISABLED and set OTEL_EXPORT_ENABLED=true instead (logic is inverted) |
| OTEL_SDK_DISABLED=true (telemetry disabled) | Remove OTEL_SDK_DISABLED; telemetry is now off by default so no replacement needed unless you want to keep it explicitly disabled via OTEL_EXPORT_ENABLED=false |
| OTEL_EXPORTER_OTLP_PROTOCOL set to any value | Rename to OTEL_EXPORTER_OTLP_TRANSPORT with the same value |
| OTEL_EXPORTER_OTLP_HEADERS set | Env var removed; find alternative configuration method per upgrade guide |

##### Removed environment variables

| Variable | Description |
|---|---|
| `OTEL_SDK_DISABLED` | Replaced by OTEL_EXPORT_ENABLED with inverted logic. Telemetry export is now off by default. |
| `OTEL_EXPORTER_OTLP_PROTOCOL` | Replaced by OTEL_EXPORTER_OTLP_TRANSPORT. |
| `OTEL_EXPORTER_OTLP_HEADERS` | Removed with no stated replacement. Refer to upgrade guide for alternative configuration. |

##### Environment variables with changed defaults

| Variable | Old default | New default | Description |
|---|---|---|---|
| `OTEL_EXPORT_ENABLED` | `enabled (via OTEL_SDK_DISABLED=false or unset)` | `false (off by default)` | OpenTelemetry export is now disabled by default. Previously the SDK was enabled unless OTEL_SDK_DISABLED was set to true. |

##### New environment variables

| Variable | Default | Required | Description |
|---|---|---|---|
| `OTEL_EXPORT_ENABLED` | `false` | No | Replaces OTEL_SDK_DISABLED with inverted logic. Set to true to enable OpenTelemetry export. Default is now off. |
| `OTEL_EXPORTER_OTLP_TRANSPORT` | — | No | Replaces OTEL_EXPORTER_OTLP_PROTOCOL for specifying the OTLP exporter transport protocol. |

---

#### ai-dial-admin-evaluation-framework-backend `0.1.0-rc.0`

##### Breaking changes

**EvalSummary CSV export column-group separator changed from `:` to `::`**

The separator used between hierarchical column family segments in exported CSV headers has changed from single colon `:` to double colon `::`. For example, `data:prompt` becomes `data::prompt` and `metric:Accuracy:score` becomes `metric::Accuracy::score`. Any downstream consumer that parses CSV headers by splitting on `:` will misparse the new format.

| Previous configuration | Required action |
|---|---|
| Consumer splits CSV headers on `:` to parse column families (e.g. `data:prompt`, `metric:Accuracy:score`) | Update CSV header parsing logic to split on `::` instead of `:` (e.g. `data::prompt`, `metric::Accuracy::score`) |

##### New environment variables

| Variable | Default | Required | Description |
|---|---|---|---|
| `TEST_CASE_BULK_MAX_DELETE_IDS` | `10000` | No | Maximum number of IDs accepted in a single bulk-delete-by-IDs request (`DELETE /test-cases:bulk`). Must be ≥ 1. |

---

#### ai-dial-adapter-vertexai `0.36.0`

##### Breaking changes

**Veo: `pubsub_topic` field removed from config**

The `pubsub_topic` field has been removed from the Veo model configuration. Any deployment config referencing this field must be updated.

| Previous configuration | Required action |
|---|---|
| Veo config contains `pubsub_topic` field | Remove the `pubsub_topic` field from the Veo model configuration |

##### Config / Helm changes

- **Removed** `veo.pubsub_topic`: The `pubsub_topic` field has been removed from the Veo model configuration.

---

#### ai-dial-admin-backend `0.18.0-rc.0`

##### Breaking changes

**ApplicationResourceDto: flat `applicationTypeSchemaId` field replaced by polymorphic `source` field**

The `applicationTypeSchemaId` field on `ApplicationResourceDto` and `CreateApplicationResourceDto` has been removed and replaced with a polymorphic `source` field. The `source` field is a `$type`-discriminated object supporting `schema` and `endpoints` variants. Any API client, automation, or integration that reads or writes `applicationTypeSchemaId` must be updated to use the new `source` structure.

| Previous configuration | Required action |
|---|---|
| API payloads use flat `applicationTypeSchemaId` field on ApplicationResourceDto / CreateApplicationResourceDto | Update all API clients and integrations to use the new polymorphic `source` field (`$type`-discriminated with `schema` and `endpoints` variants) instead of `applicationTypeSchemaId` |

##### Config / Helm changes

- **Default changed** `applicationProperties (application assets)`: `null / unset` → `empty map {}` — `applicationProperties` for application assets now defaults to an empty map instead of being absent/null.
- **Added** `features.maxTokensSupported`: New configuration property introduced in DIAL Core v0.45.0. Defaults to true.
- **Added** `features.maxCompletionTokensSupported`: New configuration property introduced in DIAL Core v0.45.0.
- **Added** `features.customTemperatureSupported`: New configuration property introduced in DIAL Core v0.45.0. Defaults to true.
- **Added** `features.reasoningEfforts`: New configuration property introduced in DIAL Core v0.45.0.
- **Added** `upstreams.secretExtraData`: New configuration property introduced in DIAL Core v0.45.0.
- **Added** `models.embeddingDimensions`: New configuration property introduced in DIAL Core v0.45.0.

---

#### ai-dial-quickapps-backend `0.9.0-rc.1`

##### Breaking changes

**DIAL files tools graduated to GA — now active regardless of ENABLE_PREVIEW_FEATURES**

The list/read_lines/search/find/write/edit/delete/copy/move tools and the features.dial_files config field are no longer gated by ENABLE_PREVIEW_FEATURES. Any deployment that previously relied on ENABLE_PREVIEW_FEATURES=false to suppress these tools will now have them active. The tool_call_result_offload sub-feature (features.dial_files.tool_call_result_offload and TOOL_CALL_RESULT_OFFLOAD__* env vars) remains behind the preview flag.

| Previous configuration | Required action |
|---|---|
| ENABLE_PREVIEW_FEATURES=false — DIAL files tools were suppressed | Review whether DIAL files tools should now be explicitly disabled via features.dial_files config; ENABLE_PREVIEW_FEATURES=false no longer suppresses them |
| ENABLE_PREVIEW_FEATURES=true — DIAL files tools were active | No action required; behavior is unchanged |

##### Config / Helm changes

- **Added** `features.dial_files`: Config field for DIAL files tools (list/read_lines/search/find/write/edit/delete/copy/move) is now GA and active regardless of ENABLE_PREVIEW_FEATURES. Previously only honoured when ENABLE_PREVIEW_FEATURES was enabled.
- **Added** `internal_attachments_available_context[].max_depth`: [Preview] New field on folder context entries bounding recursion depth when a DIAL folder is attached as a context source.

---

#### ai-dial-adapter-openai `0.40.0`

##### Deprecated environment variables

| Variable | Description |
|---|---|
| `DIAL_USE_FILE_STORAGE` | DIAL_USE_FILE_STORAGE is deprecated. DIAL Storage is now enabled automatically when DIAL_URL is set. |

**Migration:** _DIAL_USE_FILE_STORAGE=True explicitly set_ → No immediate action required; variable is deprecated but still functional. Plan to remove it in a future release.
**Migration:** _DIAL_USE_FILE_STORAGE not set or False_ → No action needed; verify storage behavior aligns with expectations when DIAL_URL is configured.

---

#### ai-dial-chat-themes `0.16.0`

##### Config / Helm changes

- **Added** `config.json / bg-inverted`: New 'bg-inverted' property added to config.json for theming.

---

#### ai-dial-core `0.45.0-rc.0`

##### Config / Helm changes

- **Added** `features.reasoningEffortsSupported`: New feature flag to indicate that a model/deployment supports reasoning efforts configuration.
- **Added** `features.maxTokensSupported / features.maxCompletionTokensSupported / features.temperatureSupported`: New feature flags to expose max_tokens, max_completion_tokens, and temperature support per deployment.
- **Added** `features.availableEndpoints`: New flags to expose available endpoints per deployment in the features listing.
- **Added** `features.reasoningEfforts (string array)`: Reasoning efforts can now be exposed as a string array in the features listing for applicable deployments.

---
