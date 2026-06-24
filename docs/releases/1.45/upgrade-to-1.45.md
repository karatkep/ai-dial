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
   - ai-dial-admin-evaluation-framework-backend: `-`

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

**NODE_POOLS config format replaced with explicit Kubernetes scheduling primitives; new env vars NODE_POOL_DEFAULT and NODE_POOL_DEFAULT_MODEL added**

The previous node pool label-key/capacity configuration format has been replaced. NODE_POOLS is now a YAML document and must be reformatted to use explicit 'nodeSelector', 'affinity', and 'tolerations' primitives per pool. Two new create-time default env vars NODE_POOL_DEFAULT and NODE_POOL_DEFAULT_MODEL are introduced.

| Previous configuration | Required action |
|---|---|
| NODE_POOLS configured with label-key/capacity format | Rewrite NODE_POOLS as a YAML document using explicit nodeSelector/affinity/tolerations primitives per pool. Set NODE_POOL_DEFAULT and NODE_POOL_DEFAULT_MODEL as needed for create-time defaults. |

**Removed deprecated config properties 'config.rest.security.default.allowedRoles' and 'providers.*.allowed-roles'; must migrate to 'roles-mapping'**

The previously deprecated 'config.rest.security.default.allowedRoles' and 'providers.*.allowed-roles' configuration properties have been removed entirely. Deployments using these properties must migrate to the 'roles-mapping' configuration.

| Previous configuration | Required action |
|---|---|
| 'config.rest.security.default.allowedRoles' set in config | Remove this property and configure access control via 'roles-mapping' instead. |
| 'providers.*.allowed-roles' set in config for one or more providers | Remove these properties and configure access control via 'roles-mapping' instead. |

##### New environment variables

| Variable | Default | Required | Description |
|---|---|---|---|
| `NODE_POOL_DEFAULT` | — | No | Specifies the default node pool to stamp at create time when using the new YAML-document NODE_POOLS configuration. |
| `NODE_POOL_DEFAULT_MODEL` | — | No | Specifies the default model node pool to stamp at create time when using the new YAML-document NODE_POOLS configuration. |

##### Config / Helm changes

- **Removed** `config.rest.security.default.allowedRoles`: Previously deprecated property fully removed. Must migrate to 'roles-mapping'.
- **Removed** `providers.*.allowed-roles`: Previously deprecated property fully removed from all provider configs. Must migrate to 'roles-mapping'.

---

#### ai-dial-adapter-vertexai `0.36.0`

##### Breaking changes

**Veo: pubsub_topic field removed from config**

The pubsub_topic field has been removed from the Veo configuration. Any deployment configs referencing this field will need to be updated.

| Previous configuration | Required action |
|---|---|
| Veo config contains pubsub_topic field | Remove pubsub_topic from Veo config; the field is no longer recognized |

##### Config / Helm changes

- **Removed** `veo.pubsub_topic`: The pubsub_topic field has been removed from the Veo model configuration.

---

#### ai-dial-adapter-openai `0.40.0`

##### Deprecated environment variables

| Variable | Description |
|---|---|
| `DIAL_USE_FILE_STORAGE` | DIAL_USE_FILE_STORAGE is deprecated. No replacement is explicitly mentioned in the release notes. |

**Migration:** _DIAL_USE_FILE_STORAGE is set_ → Plan to remove this env var; check release notes or README for updated file storage configuration guidance

---

#### ai-dial-chat-themes `0.16.0`

##### Config / Helm changes

- **Added** `config.json / bg-inverted`: New 'bg-inverted' property added to config.json for theming.

---
