# Instructions

## Versions

1. Helm chart versions:
   - dial: `6.4.0`
   - dial-core: `5.2.1`
   - dial-extension: `3.1.1`
   - dial-admin: `0.14.0`
2. Main components versions:
   - ai-dial-adapter-bedrock: `0.40.0`
   - ai-dial-adapter-openai: `0.40.0`
   - ai-dial-adapter-vertexai: `0.36.0`
   - ai-dial-adapter-dial: `0.15.0`
   - ai-dial-chat-themes: `0.16.0`
   - ai-dial-chat: `0.46.0`
   - ai-dial-core: `0.44.1`
   - ai-dial-analytics-realtime: `0.24.0`
   - ai-dial-rag: `0.42.0`
   - ai-dial-log-parser: `0.3.0`
   - ai-dial-code-interpreter: `0.2.0`
   - ai-dial-app-controller: `0.4.0`
   - ai-dial-app-builder-python: `0.1.0`
   - ai-dial-quickapps-backend: `0.8.0`
   - ai-dial-mind-map-backend: `0.14.0`
   - ai-dial-mind-map-frontend: `0.12.0`
   - ai-dial-admin-backend: `0.17.0`
   - ai-dial-admin-frontend: `0.17.1`
   - ai-dial-admin-deployment-manager-backend: `0.17.0`

## Before upgrade

### General notes

- Please review the [Config changes](#config-changes) chapter carefully for each component that is used in your DIAL installation. Changes in components' configuration may be required.
- Please check if any image tag overrides (`image.tag`) are present and remove them if they are not required anymore.
- Please check and add `image.repository` to change the image location for `redis`, `postgresql`, `keycloak` and `keycloakConfigCli` components to start using alternative Docker registries (e.g. Amazon ECR Public Gallery) if required.

### Release-specific notes

<!-- TODO: add release-specific upgrade notes -->

## Config changes

### ai-dial-adapter-bedrock

<!-- TODO: add config changes for ai-dial-adapter-bedrock -->

### ai-dial-adapter-openai

<!-- TODO: add config changes for ai-dial-adapter-openai -->

### ai-dial-adapter-vertexai

<!-- TODO: add config changes for ai-dial-adapter-vertexai -->

### ai-dial-adapter-dial

<!-- TODO: add config changes for ai-dial-adapter-dial -->

### ai-dial-chat-themes

<!-- TODO: add config changes for ai-dial-chat-themes -->

### ai-dial-chat

<!-- TODO: add config changes for ai-dial-chat -->

### ai-dial-core

<!-- TODO: add config changes for ai-dial-core -->

### ai-dial-analytics-realtime

<!-- TODO: add config changes for ai-dial-analytics-realtime -->

### ai-dial-rag

<!-- TODO: add config changes for ai-dial-rag -->

### ai-dial-log-parser

<!-- TODO: add config changes for ai-dial-log-parser -->

### ai-dial-code-interpreter

<!-- TODO: add config changes for ai-dial-code-interpreter -->

### ai-dial-app-controller

<!-- TODO: add config changes for ai-dial-app-controller -->

### ai-dial-app-builder-python

<!-- TODO: add config changes for ai-dial-app-builder-python -->

### ai-dial-quickapps-backend

<!-- TODO: add config changes for ai-dial-quickapps-backend -->

### ai-dial-mind-map-backend

<!-- TODO: add config changes for ai-dial-mind-map-backend -->

### ai-dial-mind-map-frontend

<!-- TODO: add config changes for ai-dial-mind-map-frontend -->

### ai-dial-admin-backend

<!-- TODO: add config changes for ai-dial-admin-backend -->

### ai-dial-admin-frontend

<!-- TODO: add config changes for ai-dial-admin-frontend -->

### ai-dial-admin-deployment-manager-backend

<!-- TODO: add config changes for ai-dial-admin-deployment-manager-backend -->
