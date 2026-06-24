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

**NODE_POOLS config format changed from label-key/capacity to explicit Kubernetes scheduling primitives; new env vars NODE_POOL_DEFAULT and NODE_POOL_DEFAULT_MODEL added**

The NODE_POOLS environment variable is now a YAML document and must be restructured to use explicit 'nodeSelector', 'affinity', and 'tolerations' primitives per pool instead of the previous label-key/capacity config. Two new env vars, NODE_POOL_DEFAULT and NODE_POOL_DEFAULT_MODEL, are introduced as create-time defaults.

| Previous configuration | Required action |
|---|---|
| NODE_POOLS configured with label-key/capacity format | Rewrite NODE_POOLS as a YAML document using 'nodeSelector', 'affinity', and/or 'tolerations' primitives per pool. Review the full upgrade guide for schema details. |
| NODE_POOL_DEFAULT and NODE_POOL_DEFAULT_MODEL not set | Evaluate and configure NODE_POOL_DEFAULT and NODE_POOL_DEFAULT_MODEL if create-time default pool stamping behavior is required. |

**Removed deprecated config properties 'config.rest.security.default.allowedRoles' and 'providers.*.allowed-roles'; must migrate to 'roles-mapping'**

The previously deprecated configuration properties 'config.rest.security.default.allowedRoles' and 'providers.*.allowed-roles' have been removed entirely. Deployments still using these properties will break. Migration to the 'roles-mapping' configuration is required.

| Previous configuration | Required action |
|---|---|
| 'config.rest.security.default.allowedRoles' present in config | Remove 'config.rest.security.default.allowedRoles' and configure equivalent access control via 'roles-mapping'. |
| 'providers.*.allowed-roles' present in config | Remove all 'providers.*.allowed-roles' entries and configure equivalent access control via 'roles-mapping'. |

##### Environment variables with changed defaults

| Variable | Old default | New default | Description |
|---|---|---|---|
| `NODE_POOLS` | `label-key/capacity format` | `YAML document with explicit 'nodeSelector' / 'affinity' / 'tolerations' primitives per pool` | The NODE_POOLS variable now requires a YAML document format with explicit Kubernetes scheduling primitives instead of the previous label-key/capacity structure. |

##### New environment variables

| Variable | Default | Required | Description |
|---|---|---|---|
| `NODE_POOL_DEFAULT` | — | No | Specifies the default node pool applied at resource create time as part of the new explicit Kubernetes scheduling primitives node pool configuration. |
| `NODE_POOL_DEFAULT_MODEL` | — | No | Specifies the default model node pool applied at resource create time as part of the new explicit Kubernetes scheduling primitives node pool configuration. |

##### Config / Helm changes

- **Removed** `config.rest.security.default.allowedRoles`: Previously deprecated property has been fully removed. Access control must now be configured via 'roles-mapping'.
- **Removed** `providers.*.allowed-roles`: Previously deprecated property has been fully removed. Access control must now be configured via 'roles-mapping'.
- **Added** `roles-mapping`: Replacement configuration for the removed 'config.rest.security.default.allowedRoles' and 'providers.*.allowed-roles' properties. Required to maintain access control behavior after upgrade.

---

#### ai-dial-adapter-vertexai `0.36.0`

##### Breaking changes

**Veo: `pubsub_topic` field removed from config**

The `pubsub_topic` field has been removed from the Veo model configuration. Any deployment config that includes this field must be updated to remove it.

| Previous configuration | Required action |
|---|---|
| Veo config includes `pubsub_topic` field | Remove the `pubsub_topic` field from the Veo model configuration |

##### Config / Helm changes

- **Removed** `veo.pubsub_topic`: The `pubsub_topic` field has been removed from the Veo model configuration.

---

#### ai-dial-admin-evaluation-framework-backend `0.1.0-rc.0`

##### Breaking changes

**CSV column group delimiter changed from `:` to `::`**

Any existing CSV datasets or integrations that used the single colon `:` as a column group delimiter must be updated to use `::` instead. Existing data may be parsed incorrectly if not migrated.

| Previous configuration | Required action |
|---|---|
| CSV files using `:` as column group delimiter | Update all CSV dataset files to use `::` as the column group delimiter before or after upgrading |

---

#### ai-dial-adapter-openai `0.40.0`

##### Deprecated environment variables

| Variable | Description |
|---|---|
| `DIAL_USE_FILE_STORAGE` | DIAL_USE_FILE_STORAGE is deprecated. Based on the deprecation pattern, DIAL Storage is likely now enabled automatically when DIAL_URL is set. |

**Migration:** _DIAL_USE_FILE_STORAGE explicitly set_ → Plan to remove DIAL_USE_FILE_STORAGE; verify storage behavior with DIAL_URL alone

---

#### ai-dial-chat-themes `0.16.0`

##### Config / Helm changes

- **Added** `config.json / bg-inverted`: New 'bg-inverted' color property added to config.json theme configuration.

---
