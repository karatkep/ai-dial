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

**NODE_POOLS config replaced: label-key/capacity schema removed, now uses explicit Kubernetes scheduling primitives; new env vars NODE_POOL_DEFAULT and NODE_POOL_DEFAULT_MODEL added**

The NODE_POOLS configuration is now a YAML document using explicit 'nodeSelector', 'affinity', and 'tolerations' primitives per pool. The previous label-key/capacity config shape is no longer supported. Two new create-time default fields NODE_POOL_DEFAULT and NODE_POOL_DEFAULT_MODEL are introduced.

| Previous configuration | Required action |
|---|---|
| NODE_POOLS configured with label-key/capacity schema | Rewrite NODE_POOLS as a YAML document with explicit nodeSelector/affinity/tolerations per pool; set NODE_POOL_DEFAULT and NODE_POOL_DEFAULT_MODEL as appropriate |

**Removed deprecated config properties 'config.rest.security.default.allowedRoles' and 'providers.*.allowed-roles'; must migrate to 'roles-mapping'**

The previously deprecated config keys 'config.rest.security.default.allowedRoles' and 'providers.*.allowed-roles' have been fully removed. Deployments still using these keys must migrate to the 'roles-mapping' configuration.

| Previous configuration | Required action |
|---|---|
| 'config.rest.security.default.allowedRoles' set in config | Remove the property and configure equivalent access control via 'roles-mapping' |
| 'providers.*.allowed-roles' set in config | Remove the property and configure equivalent access control via 'roles-mapping' |

##### New environment variables

| Variable | Default | Required | Description |
|---|---|---|---|
| `NODE_POOL_DEFAULT` | — | No | Specifies the default node pool to stamp at create-time for deployments when no pool is explicitly selected. |
| `NODE_POOL_DEFAULT_MODEL` | — | No | Specifies the default node pool to stamp at create-time for model deployments when no pool is explicitly selected. |

##### Config / Helm changes

- **Removed** `config.rest.security.default.allowedRoles`: Previously deprecated property for allowed roles has been fully removed. Must migrate to 'roles-mapping'.
- **Removed** `providers.*.allowed-roles`: Previously deprecated per-provider allowed-roles property has been fully removed. Must migrate to 'roles-mapping'.
- **Added** `NODE_POOLS[*].nodeSelector`: Explicit Kubernetes nodeSelector primitive per node pool, replacing the previous label-key/capacity schema.
- **Added** `NODE_POOLS[*].affinity`: Explicit Kubernetes affinity primitive per node pool, replacing the previous label-key/capacity schema.
- **Added** `NODE_POOLS[*].tolerations`: Explicit Kubernetes tolerations primitive per node pool, replacing the previous label-key/capacity schema.
- **Added** `roles-mapping`: Replacement for the removed 'config.rest.security.default.allowedRoles' and 'providers.*.allowed-roles' config properties for access control.

---

#### ai-dial-adapter-vertexai `0.36.0`

##### Breaking changes

**Veo: `pubsub_topic` field removed from config**

The `pubsub_topic` field has been removed from the Veo model configuration. Any deployment config that includes this field must be updated to remove it.

| Previous configuration | Required action |
|---|---|
| Veo config contains `pubsub_topic` field | Remove `pubsub_topic` from the Veo model config before upgrading |

##### Config / Helm changes

- **Removed** `veo.pubsub_topic`: The `pubsub_topic` field has been removed from the Veo model configuration.

---

#### ai-dial-admin-evaluation-framework-backend `0.1.0-rc.0`

##### Breaking changes

**CSV column group delimiter changed from `:` to `::`**

The delimiter used to separate column groups in CSV files has changed from a single colon to a double colon. Any existing CSV datasets or integrations relying on the old delimiter format will be misparsed after upgrade.

| Previous configuration | Required action |
|---|---|
| CSV files using `:` as column group delimiter | Update all CSV dataset files to use `::` as the column group delimiter before or after upgrading |

---

#### ai-dial-adapter-openai `0.40.0`

##### Deprecated environment variables

| Variable | Description |
|---|---|
| `DIAL_USE_FILE_STORAGE` | DIAL_USE_FILE_STORAGE is deprecated. DIAL Storage is now enabled automatically when DIAL_URL is set. |

**Migration:** _DIAL_USE_FILE_STORAGE=True explicitly set_ → Remove DIAL_USE_FILE_STORAGE; storage will be enabled automatically when DIAL_URL is set
**Migration:** _DIAL_USE_FILE_STORAGE unset or False_ → Verify DIAL_URL is not set if storage should remain disabled; otherwise remove the variable

---

#### ai-dial-chat-themes `0.16.0`

##### Config / Helm changes

- **Added** `config.json/bg-inverted`: New 'bg-inverted' property added to config.json for theme configuration.

---
