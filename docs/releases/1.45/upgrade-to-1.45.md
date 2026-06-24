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
   - ai-dial-quickapps-backend: `0.8.1`
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

#### ai-dial-admin-deployment-manager-backend `0.17.0`

> [!CAUTION]
> This release includes high-priority changes. Please review the [full upgrade guide](https://github.com/epam/ai-dial-admin-deployment-manager-backend/blob/0.17.0/docs/upgrade-plans/0.17.0.md) before proceeding.

##### Breaking changes

**NODE_POOLS config format changed from label-key/capacity to explicit Kubernetes scheduling primitives (YAML document); new NODE_POOL_DEFAULT and NODE_POOL_DEFAULT_MODEL create-time defaults added**

The NODE_POOLS environment variable/config is now a YAML document and must be reformatted to use explicit 'nodeSelector', 'affinity', and 'tolerations' primitives per pool. Two new create-time default fields NODE_POOL_DEFAULT and NODE_POOL_DEFAULT_MODEL are introduced.

| Previous configuration | Required action |
|---|---|
| NODE_POOLS configured with label-key/capacity format | Rewrite NODE_POOLS as a YAML document using explicit 'nodeSelector', 'affinity', and 'tolerations' per pool; set NODE_POOL_DEFAULT and NODE_POOL_DEFAULT_MODEL as needed |

**Removed deprecated config properties 'config.rest.security.default.allowedRoles' and 'providers.*.allowed-roles'; must migrate to 'roles-mapping'**

The previously deprecated 'config.rest.security.default.allowedRoles' and 'providers.*.allowed-roles' config keys have been removed. Deployments still using these keys must migrate to the 'roles-mapping' configuration.

| Previous configuration | Required action |
|---|---|
| 'config.rest.security.default.allowedRoles' or 'providers.*.allowed-roles' present in configuration | Remove these keys and configure access control using 'roles-mapping' instead |

##### New environment variables

| Variable | Default | Required | Description |
|---|---|---|---|
| `NODE_POOL_DEFAULT` | — | No | Create-time default node pool to stamp onto new deployments when no pool is explicitly specified. |
| `NODE_POOL_DEFAULT_MODEL` | — | No | Create-time default node pool for model deployments when no pool is explicitly specified. |

##### Config / Helm changes

- **Removed** `config.rest.security.default.allowedRoles`: Previously deprecated property for allowed roles has been removed. Use 'roles-mapping' instead.
- **Removed** `providers.*.allowed-roles`: Previously deprecated per-provider allowed-roles property has been removed. Use 'roles-mapping' instead.
- **Added** `node_pools[*].nodeSelector`: Explicit Kubernetes nodeSelector primitive per node pool, replacing the previous label-key/capacity format.
- **Added** `node_pools[*].affinity`: Explicit Kubernetes affinity primitive per node pool, replacing the previous label-key/capacity format.
- **Added** `node_pools[*].tolerations`: Explicit Kubernetes tolerations primitive per node pool, replacing the previous label-key/capacity format.

---

#### ai-dial-adapter-vertexai `0.36.0`

##### Breaking changes

**Veo: `pubsub_topic` field removed from config**

The `pubsub_topic` field has been removed from the Veo model configuration. Any deployment config that includes this field must be updated to remove it before or during upgrade.

| Previous configuration | Required action |
|---|---|
| Veo model config contains `pubsub_topic` field | Remove the `pubsub_topic` field from the Veo model configuration |

##### Config / Helm changes

- **Removed** `veo.pubsub_topic`: The `pubsub_topic` field has been removed from the Veo model configuration.

---

#### ai-dial-adapter-openai `0.40.0`

##### Deprecated environment variables

| Variable | Description |
|---|---|
| `DIAL_USE_FILE_STORAGE` | DIAL_USE_FILE_STORAGE is deprecated. No replacement explicitly mentioned in release notes. |

**Migration:** _DIAL_USE_FILE_STORAGE is set_ → Plan to remove this env var; check release notes or README for updated file storage configuration guidance

---

#### ai-dial-chat-themes `0.16.0`

##### Config / Helm changes

- **Added** `config.json / bg-inverted`: New 'bg-inverted' property added to config.json theme configuration.

---
