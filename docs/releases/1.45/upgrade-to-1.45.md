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

**Spring Boot upgraded from 3.5 to 4.0.6 (Spring Framework 7, Hibernate 7.2); JSON serialization migrated to Jackson 3**

Major framework version bumps may require configuration and behavior changes. Jackson 3 migration caused a known regression in image entrypoint/cmd binding (fixed in #367). Review the full upgrade guide for migration steps.

| Previous configuration | Required action |
|---|---|
| Running on Spring Boot 3.5 / Spring Framework 6.x / Hibernate 6.x / Jackson 2 | Review upgrade guide at docs/upgrade-plans/0.18.0.md for all required migration steps before upgrading |

**OpenTelemetry configuration overhauled: env vars renamed/removed and default changed to off**

Three env var changes: (1) OTEL_SDK_DISABLED replaced by OTEL_EXPORT_ENABLED (inverted logic — previously set to 'true' to disable, now must be set to 'true' to enable). (2) OTEL_EXPORTER_OTLP_PROTOCOL replaced by OTEL_EXPORTER_OTLP_TRANSPORT. (3) OTEL_EXPORTER_OTLP_HEADERS removed entirely. Deployments using telemetry export will silently stop exporting until migrated.

| Previous configuration | Required action |
|---|---|
| OTEL_SDK_DISABLED=false (or unset) to enable telemetry export | Replace with OTEL_EXPORT_ENABLED=true |
| OTEL_SDK_DISABLED=true to disable telemetry export | Remove OTEL_SDK_DISABLED; OTEL_EXPORT_ENABLED defaults to false (off), so no action needed to keep it disabled |
| OTEL_EXPORTER_OTLP_PROTOCOL set to a transport protocol value | Rename to OTEL_EXPORTER_OTLP_TRANSPORT with the same value |
| OTEL_EXPORTER_OTLP_HEADERS set with OTLP authentication/custom headers | Remove OTEL_EXPORTER_OTLP_HEADERS; consult upgrade guide for replacement mechanism if headers are required |

##### Removed environment variables

| Variable | Description |
|---|---|
| `OTEL_SDK_DISABLED` | Replaced by OTEL_EXPORT_ENABLED with inverted logic. Must be migrated or telemetry export will silently stop. |
| `OTEL_EXPORTER_OTLP_PROTOCOL` | Replaced by OTEL_EXPORTER_OTLP_TRANSPORT. |
| `OTEL_EXPORTER_OTLP_HEADERS` | Removed entirely with no named replacement. Deployments using this for OTLP authentication headers must consult the upgrade guide. |

##### New environment variables

| Variable | Default | Required | Description |
|---|---|---|---|
| `OTEL_EXPORT_ENABLED` | `false` | No | Replaces OTEL_SDK_DISABLED with inverted logic. Set to 'true' to enable OpenTelemetry export. Off by default. |
| `OTEL_EXPORTER_OTLP_TRANSPORT` | — | No | Replaces OTEL_EXPORTER_OTLP_PROTOCOL. Specifies the OTLP transport protocol for telemetry export. |

---

#### ai-dial-admin-evaluation-framework-backend `0.1.0-rc.0`

##### Breaking changes

**EvalSummary CSV column-group separator changed from `:` to `::`**

CSV export headers now join hierarchical column families with `::` instead of `:`. For example, `data:prompt` becomes `data::prompt` and `metric:Accuracy:score` becomes `metric::Accuracy::score`. Any downstream consumer that parses exported CSV headers by splitting on `:` must be updated.

| Previous configuration | Required action |
|---|---|
| Consumer splits CSV header on `:` to parse column families (e.g. `data:prompt`, `metric:Accuracy:score`) | Update parser/consumer to split on `::` instead of `:` (e.g. `data::prompt`, `metric::Accuracy::score`) |

##### New environment variables

| Variable | Default | Required | Description |
|---|---|---|---|
| `TEST_CASE_BULK_MAX_DELETE_IDS` | `10000` | No | Maximum number of IDs accepted in a single bulk-delete-by-IDs request (`DELETE /test-cases:bulk`). Must be ≥ 1. |

---

#### ai-dial-adapter-vertexai `0.36.0`

##### Breaking changes

**Veo: `pubsub_topic` field removed from config**

The `pubsub_topic` field has been removed from the Veo model configuration. Any deployment config referencing this field must be updated to remove it.

| Previous configuration | Required action |
|---|---|
| Veo config contains `pubsub_topic` field | Remove `pubsub_topic` from the Veo model configuration |

##### Config / Helm changes

- **Removed** `veo.pubsub_topic`: The `pubsub_topic` field has been removed from the Veo model configuration.

---

#### ai-dial-admin-backend `0.18.0-rc.0`

##### Breaking changes

**ApplicationResourceDto: flat `applicationTypeSchemaId` field replaced by polymorphic `source` field**

The flat `applicationTypeSchemaId` field on `ApplicationResourceDto` and `CreateApplicationResourceDto` has been removed and replaced with a polymorphic `source` field. The `source` field is a `$type`-discriminated object with `schema` and `endpoints` variants. Any API client, integration, or config that sets or reads `applicationTypeSchemaId` must be updated to use the new `source` structure.

| Previous configuration | Required action |
|---|---|
| API payloads/clients using `applicationTypeSchemaId` field on ApplicationResourceDto or CreateApplicationResourceDto | Update all API clients, integrations, and config payloads to use the new polymorphic `source` field (`$type`-discriminated with `schema` and `endpoints` variants) instead of `applicationTypeSchemaId` |

##### Config / Helm changes

- **Default changed** `features.maxTokensSupported`: `unset/not present` → `true` — New field defaults to true when not explicitly configured.
- **Default changed** `features.customTemperatureSupported`: `unset/not present` → `true` — New field defaults to true when not explicitly configured.
- **Default changed** `applicationProperties`: `unset/null` → `empty map {}` — applicationProperties for application assets now defaults to an empty map instead of being absent/null.
- **Added** `features.maxTokensSupported`: New configuration property introduced in DIAL Core v0.45.0. Defaults to `true`.
- **Added** `features.maxCompletionTokensSupported`: New configuration property introduced in DIAL Core v0.45.0.
- **Added** `features.customTemperatureSupported`: New configuration property introduced in DIAL Core v0.45.0. Defaults to `true`.
- **Added** `features.reasoningEfforts`: New configuration property introduced in DIAL Core v0.45.0.
- **Added** `upstreams.secretExtraData`: New configuration property introduced in DIAL Core v0.45.0.
- **Added** `models.embeddingDimensions`: New configuration property introduced in DIAL Core v0.45.0.

---

#### ai-dial-quickapps-backend `0.9.0-rc.1`

##### Breaking changes

**DIAL files tools are now GA and active regardless of ENABLE_PREVIEW_FEATURES**

The `list` / `read_lines` / `search` / `find` / `write` / `edit` / `delete` / `copy` / `move` tools and the `features.dial_files` config field are no longer gated by `ENABLE_PREVIEW_FEATURES`. Any deployment that previously kept `ENABLE_PREVIEW_FEATURES` disabled to suppress these tools will now have them active. Only the `tool_call_result_offload` sub-feature remains behind the preview flag.

| Previous configuration | Required action |
|---|---|
| ENABLE_PREVIEW_FEATURES=false (or unset) — DIAL files tools were inactive | After upgrade these tools will be active unconditionally. If you wish to disable them, use the `features.dial_files` config field explicitly. Review whether exposing these tools to users is acceptable before upgrading. |
| ENABLE_PREVIEW_FEATURES=true — DIAL files tools were active | No action required; behavior is unchanged. |

##### Config / Helm changes

- **Added** `features.dial_files`: Config field controlling the DIAL files tools (list/read_lines/search/find/write/edit/delete/copy/move). Previously only respected when ENABLE_PREVIEW_FEATURES was enabled; now active unconditionally (GA). Use this field to explicitly disable the tools if needed.

---

#### ai-dial-adapter-openai `0.40.0`

##### Deprecated environment variables

| Variable | Description |
|---|---|
| `DIAL_USE_FILE_STORAGE` | DIAL_USE_FILE_STORAGE is deprecated. Based on the deprecation pattern, DIAL Storage is likely enabled automatically when DIAL_URL is set; verify current README for exact replacement behavior. |

**Migration:** _DIAL_USE_FILE_STORAGE is set in deployment_ → Review updated documentation and remove DIAL_USE_FILE_STORAGE; verify storage behavior is still correct after removal

---

#### ai-dial-chat-themes `0.16.0`

##### Config / Helm changes

- **Added** `config.json / bg-inverted`: New 'bg-inverted' color property added to config.json theme configuration.

---

#### ai-dial-core `0.45.0-rc.0`

##### Config / Helm changes

- **Added** `features.reasoningEffortsSupported`: New feature flag to indicate whether reasoning efforts are supported by a deployment.
- **Added** `features.maxTokensSupported / features.maxCompletionTokensSupported / features.temperatureSupported`: New feature flags to expose max_tokens, max_completion_tokens, and temperature capability indicators in model/deployment listing.
- **Added** `features.availableEndpoints`: New flags to expose available endpoints per deployment in feature listings.
- **Added** `roles.readonly-admin`: New readonly-admin role introduced to allow reading user data without write access.

---
