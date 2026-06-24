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
   - ai-dial-core: `0.44.5`
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
   - ai-dial-admin-evaluation-framework-backend: `0.1.0-rc.0`

## Before upgrade

### General notes

- Please review the [Config changes](#config-changes) chapter carefully for each component that is used in your DIAL installation. Changes in components' configuration may be required.
- Please check if any image tag overrides (`image.tag`) are present and remove them if they are not required anymore.
- Please check and add `image.repository` to change the image location for `redis`, `postgresql`, `keycloak` and `keycloakConfigCli` components to start using alternative Docker registries (e.g. Amazon ECR Public Gallery) if required.

### Release-specific notes

#### ai-dial-admin-deployment-manager-backend `0.17.0`

> [!CAUTION]
> This release includes high-priority changes. Please review the [full upgrade guide](https://github.com/epam/ai-dial-admin-deployment-manager-backend/blob/0.17.0/docs/upgrade-plans/0.17.0.md) before proceeding.

##### Breaking changes

**NODE_POOLS config replaced: label-key/capacity schema replaced with explicit Kubernetes scheduling primitives; format changed to YAML document**

The NODE_POOLS environment variable is now a YAML document. The previous label-key/capacity config structure is no longer valid. Two new create-time defaults are introduced: NODE_POOL_DEFAULT and NODE_POOL_DEFAULT_MODEL. Pools now use explicit nodeSelector, affinity, and tolerations fields per pool.

| Previous configuration | Required action |
|---|---|
| NODE_POOLS configured with label-key/capacity schema | Rewrite NODE_POOLS as a YAML document using nodeSelector/affinity/tolerations primitives per pool. Set NODE_POOL_DEFAULT and NODE_POOL_DEFAULT_MODEL as appropriate. |

**Removed deprecated config properties 'config.rest.security.default.allowedRoles' and 'providers.*.allowed-roles'; migrate to 'roles-mapping'**

The previously deprecated config keys 'config.rest.security.default.allowedRoles' and 'providers.*.allowed-roles' have been removed entirely. Deployments still using these keys must migrate to 'roles-mapping' before upgrading.

| Previous configuration | Required action |
|---|---|
| 'config.rest.security.default.allowedRoles' set in configuration | Remove 'config.rest.security.default.allowedRoles' and configure equivalent access control via 'roles-mapping'. |
| 'providers.*.allowed-roles' set in provider configuration | Remove 'providers.*.allowed-roles' and configure equivalent access control via 'roles-mapping'. |

##### New environment variables

| Variable | Default | Required | Description |
|---|---|---|---|
| `NODE_POOL_DEFAULT` | — | No | Specifies the default node pool to use at create-time for deployments. Introduced as part of the NODE_POOLS YAML document restructuring. |
| `NODE_POOL_DEFAULT_MODEL` | — | No | Specifies the default node pool to use at create-time for model deployments. Introduced as part of the NODE_POOLS YAML document restructuring. |

##### Config / Helm changes

- **Removed** `config.rest.security.default.allowedRoles`: Previously deprecated config key for role-based access control. Now fully removed; must migrate to 'roles-mapping'.
- **Removed** `providers.*.allowed-roles`: Previously deprecated per-provider role config key. Now fully removed; must migrate to 'roles-mapping'.
- **Removed** `node_pools[*].label-key`: Old node pool label-key field removed as part of restructuring NODE_POOLS to use explicit Kubernetes scheduling primitives.
- **Removed** `node_pools[*].capacity`: Old node pool capacity field removed as part of restructuring NODE_POOLS to use explicit Kubernetes scheduling primitives.
- **Added** `node_pools[*].nodeSelector`: Explicit Kubernetes nodeSelector field per node pool entry in the NODE_POOLS YAML document.
- **Added** `node_pools[*].affinity`: Explicit Kubernetes affinity field per node pool entry in the NODE_POOLS YAML document.
- **Added** `node_pools[*].tolerations`: Explicit Kubernetes tolerations field per node pool entry in the NODE_POOLS YAML document.

---

#### ai-dial-admin-evaluation-framework-backend `0.1.0-rc.0`

##### Breaking changes

**EvalSummary CSV export column-group separator changed from `:` to `::`**

Any downstream consumer that parses exported CSV headers by splitting on `:` must update to split on `::`. Example: `data:prompt` → `data::prompt`, `metric:Accuracy:score` → `metric::Accuracy::score`.

| Previous configuration | Required action |
|---|---|
| CSV header parser splits on single `:` (e.g. `data:prompt`, `metric:Accuracy:score`) | Update parser to split on `::` instead of `:` |

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
| Veo config contains `pubsub_topic` field | Remove `pubsub_topic` from Veo model configuration |

##### Config / Helm changes

- **Removed** `veo.pubsub_topic`: The `pubsub_topic` field has been removed from Veo model configuration.

---

#### ai-dial-adapter-openai `0.40.0`

##### Deprecated environment variables

| Variable | Description |
|---|---|
| `DIAL_USE_FILE_STORAGE` | DIAL_USE_FILE_STORAGE is deprecated. DIAL Storage is now enabled automatically when DIAL_URL is set. |

**Migration:** _DIAL_USE_FILE_STORAGE=True explicitly set_ → No action required; storage behavior is now automatic when DIAL_URL is set
**Migration:** _DIAL_USE_FILE_STORAGE=False or unset to disable storage_ → Verify new automatic behavior does not enable storage unexpectedly; remove DIAL_URL if storage should remain disabled

---

#### ai-dial-chat-themes `0.16.0`

##### Config / Helm changes

- **Added** `config.json / bg-inverted`: New 'bg-inverted' color property added to config.json theme configuration.

---
