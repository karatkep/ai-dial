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
   - ai-dial-chat: `0.46.0`
   - ai-dial-core: `0.44.4`
   - ai-dial-analytics-realtime: `0.24.1`
   - ai-dial-rag: `0.42.0`
   - ai-dial-log-parser: `0.3.0`
   - ai-dial-code-interpreter: `0.2.0`
   - ai-dial-app-controller: `0.4.0`
   - ai-dial-app-builder-python: `0.1.0`
   - ai-dial-quickapps-backend: `0.8.0`
   - ai-dial-mind-map-backend: `0.14.1`
   - ai-dial-mind-map-frontend: `0.13.0`
   - ai-dial-admin-backend: `0.17.0`
   - ai-dial-admin-frontend: `0.17.1`
   - ai-dial-admin-deployment-manager-backend: `0.17.0`

## Before upgrade

### General notes

- Please review the [Config changes](#config-changes) chapter carefully for each component that is used in your DIAL installation. Changes in components' configuration may be required.
- Please check if any image tag overrides (`image.tag`) are present and remove them if they are not required anymore.
- Please check and add `image.repository` to change the image location for `redis`, `postgresql`, `keycloak` and `keycloakConfigCli` components to start using alternative Docker registries (e.g. Amazon ECR Public Gallery) if required.

### Release-specific notes

#### ai-dial-adapter-bedrock `0.40.0`

## Features

* Converse API: support DIAL caching (#429)

## Other

* configure pre-push hooks by default (#430)
* bump idna from 3.7 to 3.15 (#426)
* bump dataaxiom/ghcr-cleanup-action from 1.0.16 to 1.1.0 in the github-actions group across 1 directory (#428)
* bump dataaxiom/ghcr-cleanup-action from 1.1.0 to 1.2.1 in the github-actions group across 1 directory (#432)
* bump the ai-dial-ci group with 4 updates (#427, #431, #435)

---

#### ai-dial-adapter-openai `0.40.0`

## BREAKING CHANGES

* deprecate DIAL_USE_FILE_STORAGE env variable (#445)

## Features

* Responses-to-Responses API: proxy endpoints for retrieving, deleting and cancelling of created response (#449)
* Whisper: improve errors messages (#451)

## Fixes

* Responses-to-Responses API: support string input (#453)
* support chunking strategy for GPT Transcribe models (#447)

## Other

* configure pre-push hooks by default (#459)
* bump idna from 3.7 to 3.15 (#456)
* bump urllib3 from 2.6.3 to 2.7.0 (#450)
* bump aidial-sdk from 0.36.0 to 0.37.0; bump aidial-adapter-anthropic from 0.8.0 to 0.10.0 (#462)
* fix typo in README that led to its incorrect rendering (#454)
* update documentation about Responses API deployments (#444)
* bump dataaxiom/ghcr-cleanup-action from 1.0.16 to 1.1.0 in the github-actions group across 1 directory (#458)
* bump dataaxiom/ghcr-cleanup-action from 1.1.0 to 1.2.1 in the github-actions group across 1 directory (#461)
* bump the ai-dial-ci group with 4 updates (#452, #457, #460, #465)

---

#### ai-dial-adapter-vertexai `0.36.0`

## Features

* support Mistral models (#454)
* WIF support for AWS container credential providers (#459)

## Fixes

* Veo: add generation stage to avoid timeout errors because of an idle response (#466)
* Veo: fix misfiring warning in case of content filtering (#461)
* Veo: remove pubsub_topic field from config (#458)

## Other

* bump aidial-sdk from 0.36.0 to 0.37.0; bump aidial-adapter-anthropic from 0.8.0 to 0.10.0 (#474)
* configure pre-push hooks by default (#470)
* remove unused project dependencies (#464)
* bump idna from 3.7 to 3.15 (#467)
* bump urllib3 from 2.6.3 to 2.7.0 (#460)
* bump dataaxiom/ghcr-cleanup-action (#469, #473)
* bump the ai-dial-ci group with 4 updates (#462, #468, #472, #477)

---

#### ai-dial-adapter-dial `0.15.0`

## Other

* configure pre-push hooks by default (#128)
* bump dataaxiom/ghcr-cleanup-action from 1.0.16 to 1.1.0 in the github-actions group across 1 directory (#127)
* bump dataaxiom/ghcr-cleanup-action from 1.1.0 to 1.2.1 in the github-actions group across 1 directory (#130)
* bump idna from 3.7 to 3.15 (#125)
* bump the ai-dial-ci group with 4 updates (#123, #126, #129, #131)

---

#### ai-dial-chat-themes `0.16.0`

## Features

* Add 'bg-inverted' property to config.json (#133)
* Update DIAL Chat and DIAL Admin favicon images in correct size and quality (#130)
* add default logo's and favicon for DIAL Admin and DIAL Chat (#129)
* add new color for neutral (#132)

## Other

* bump the ai-dial-ci group with 4 updates (#131)
* bump the ai-dial-ci group with 4 updates (#135)
* bump the ai-dial-ci group with 4 updates (#137)
* update nginx version (#134)

---

#### ai-dial-chat `0.46.0`

## Features

* Add error toast when a model or agent fails to be added to `installed_deployments.json` (Issue #6523) (#6522)
* Show visual indicator on the filter panel when filters are collapsed (Issue #6393) (#6493)
* [Preview] Align user messages to the end of the chat column via the `user-message-align-end` feature flag — uses logical CSS properties so RTL layout mirrors correctly (Issue #6775) (#6778)
* Add additional audio format support to the in-chat audio player (Issue #6472) (#6614)
* Switch applications to the `dial-app` toolset type (previously `dial-deployment`) and add MCP configuration view for applications that support the `connect-mcp` feature (Issue #6517) (#6742)
* Show an error message on an image attachment when the referenced file has been deleted (Issue #820) (#6658)
* Display token usage limits in the model detail view on the Marketplace (Issue #6536) (#6633)
* Render the instructions field as Markdown in the Quick App review window (Issue #6200) (#6698)
* Allow operators to configure the default preferred order of audio recording formats (Issue #6527) (#6532)
* Add a time-awareness setting to the Quick App editor (Issue #6516) (#6788)
* Include a `traceId` in error toast messages after failed API calls — surfaces the backend trace for easier incident lookup (Issue #5196) (#6662)
* Enable voice input while editing an existing user message (Issue #6466) (#6607)
* Use date-first format for exported conversation archive filenames to improve alphabetical sort order (Issue #6602) (#6663)
* Fix batch file deletion to correctly remove nested items when a parent folder is deleted (Issue #6702) (#6718)
* Hide the microphone button when the most recent agent message contains an error (Issue #6456) (#6696)
* Integrate the PDF highlighter into the chat attachment viewer (Issue #6625) (#6621)
* Support configurable logo and favicon for custom branding deployments (Issue #6600) (#6600)
* Add Quick App skills support (Issue #6513) (#6685)
* Update the publication request path title to reflect publish, unpublish, or mixed states (Issue #2150) (#6630)
* Rename Quick App 2.0 toolset field names to align with updated schema (Issue #6695) (#6714)
* [Overlay] Add floating panel toggle buttons for the conversations and prompts sidebars — rendered in the top corners when the DIAL header is hidden, controlled by the `conversations-panel-toggle` and `prompts-panel-toggle` feature flags (Issue #6710) (#6791)

## Fixes

* Fix auto-selection for nested files and folders (Issue #4484) (#6586)
* Align the downloaded files archive filename format with conversation and prompt export naming (Issue #5687) (#6535)
* Support AzureB2C CIAM tokens that emit `email` as a string instead of `emails` as an array, fixing an auth failure for CIAM-backed tenants (Issue #6594) (#6595)
* Correct logo source selection so the right logo variant loads for dark and light themes (Issue #6600) (#6616)
* Enforce UTF-8 byte-based limits for entity names — replaces the old 160-character limit to prevent overflows with multi-byte characters (e.g. Cyrillic) on S3/GCS/Azure/MinIO backends (Issue #3808) (#6613)
* Add checkboxes to the table attachment view (Issue #6482) (#6667)
* Restore the context menu in the select-folder modal (Issue #5737) (#6624)
* Include the environment name in exported files archive filenames (Issue #5687) (#6593)
* Add top margin to the modal header to prevent overlap (Issue #6520) (#6556)
* Add padding to conversation intro text (Issue #6512) (#6620)
* Add scroll to the application card in the Marketplace (Issue #6576) (#6578)
* Show a tooltip for long values in PDF attachments (Issue #6781) (#6784)
* Fix overly aggressive word-break in model descriptions (Issue #6538) (#6555)
* Replace the publication filter icon button with the correct icon style (Issue #6571, #1775) (#6577)
* Rename Quick App field labels to match the updated spec (Issue #6521) (#6634)
* Fix redirect logic for isolated-model routes (Issue #4985) (#6632)
* Update "Remove access" button appearance (Issues #6429, #6499) (#6669)
* Fix duplicate toolset action menu entries — deduplicate ordering (Issue #6540) (#6554, #6584)
* Copy user-attached review files to the review bucket when submitting a publication request (Issue #6387) (#6592)
* Fix Content Security Policy headers (#6651, `dix csp` commit)
* Fix 404 redirect when the completion URL inside the apps editor returns a 404 (Issue #6727) (#6794, #6825)
* Fix Overlay `setOverlayOptions` theme application so it correctly applies the theme after initialization (Issue #6721) (#6722)
* Fix shared files disappearing from folders after switching between File Manager tabs (Issue #6638) (#6648)
* Fix accepting your own share link triggering an error (Issue #6164) (#6709)
* Fix copy-paste of a space into a conversation starter field adding a blank starter (Issue #6353) (#6787)
* Fix token usage display UI issues in the application card (Issue #6536) (#6790)
* Fix auto-select requirements check (Issue #6463) (#6725)
* Fix cursor always jumping to end of field when editing user messages with voice input active (Issue #6466) (#6797)
* Fix File Manager change-path view (Issue #5606) (#6789)
* Fix hidden files being incorrectly blocked when attaching files (Issue #6260) (#6774)
* Fix multiple skills validation issues (Issues #6763, #6766, #6762, #6769, #6779, #6765) (#6786)
* Fix skills schema validation (Issues #6757, #6752) (#6808)
* Fix several microphone button UI inconsistencies — wrong visibility in replay mode and error state (Issues #6456, #6484) (#6759)
* Fix the "blinking" transition when editing a user message while switching from stop-recording to start-transcription (Issue #6466) (#6767)
* Fix modal tooltip positioning (Issue #6781) (#6793)
* Set `mcp` as the default transport for `dial-app` toolsets (Issue #6517) (#6804)
* Support legacy Quick App 2 field names alongside the new ones; remove form-only fields from the API payload (Issue #6695) (#6719)
* Fix byte-count vs. character-count mismatch in the entity name error message (Issue #6803) (#6818)
* Fix disappeared assistant message action buttons (Issue #6822) (#6830)
* Fix disappearing "unavailable model" hint when saving an app (Issue #5244) (#6771)
* Fix mobile: chat header action buttons now left-align correctly (Issue #6824) (#6831)
* Fix empty publication requests not loading (Issue #4919) (#6494)
* Fix empty value placeholder in the upload-device modal (Issue #1330) (#6626)
* Fix error toast when sharing an application with a public resource fails (Issue #5480) (#6636)
* Fix File Manager change-path panel design (Issue #5606) (#6611)
* Fix filter indicator padding (#6590)
* Fix grouped visualizer iframe rendering double on mount (#6528)
* Fix hover state styles on select components (Issue #6627) (#6668, #6678)
* Fix instructions field showing as single line in the Quick App review modal (Issue #6200) (#6760)
* Fix console error exposing internal DIAL core and bucket URLs (Issue #6684) (#6708)
* Fix microphone icon shown in Replay mode (Issue #6484) (#6673)
* Fix modal dividers (Issue #6840) (#6841)
* Fix modal size (Issue #6542) (#6581)
* Fix Monaco editor behaviour for empty folders (Issues #6222, #6223) (#6585)
* Fix new starter not saving when text contains only spaces (Issue #6353) (#6591)
* Fix stale file list not reflecting deletions without a page refresh (Issue #5706) (#6491)
* Fix PDF highlighter display issues (Issue #6625) (#6733)
* Fix prompt content height overflow (Issue #6764) (#6780)
* Fix Quick App review flow when the orchestrator deployment does not advertise `temperature` support (regression fix) (Issue #6637) (#6640)
* Fix reset of selected toolset not working (Issue #6464) (#6730)
* Fix folder-level publication rules not appearing when an admin updates a publishing request (Issue #5833) (#6690)
* Fix select component height (Issue #6691) (#6693, #6712)
* Fix selection highlight styles (Issue #5611) (#6717, #6731)
* Fix share limit exceeded error message (Issue #5291) (#6572)
* Fix model selector tooltip not showing correctly (Issue #6609) (#6713, #6735)
* Fix icon size for local custom icons in `IconButton` (Issue #6513) (#6738)
* Fix long start button text being clipped (Issue #6503) (#6518)
* Fix switcher toggle colors (Issue #6618) (#6622, #6653, #6749)
* Fix sub-folder search not returning results (Issue #6426) (#6433)
* Fix switch color (#6798)
* Fix app runner name being truncated in the UI (Issue #6676) (#6707)
* Fix application icon not updating when an admin edits a publication (Issue #6656) (#6674)
* Fix incorrect `isolatedModelId` sanitization when creating new conversations (Issue #4985) (#6654)
* Fix wrong 404 error surfaced from a completion URL (Issue #6220) (#6703)
* Hide the replay button for shared conversations (#6629)
* Remove colon from entity list titles (Issue #6543) (#6548)
* Remove N/A version display from model cards (Issue #6526) (#6579)
* Remove blurred icon outlines in Firefox (Issue #6649) (#6770)
* Fix attach-folder logic for nested selection (Issue #4484) (#6641)
* Fix button alignment in modal footer (Issue #6650) (#6724)
* Fix change-remove-access button appearance (Issues #6429, #6499) — follow-up to #6669 (#6675 ui-kit update)
* Fix user name resolution for AzureB2C `name` claim (Issue #6587) (#6589)
* Fix empty file folder deletion (Issue #6561) (#6597)
* Fix delete icon missing for empty files folder (Issue #6561)
* Open conversations sidebar by default when the `ShowConversationsSectionByDefault` feature is active in Overlay mode (Issue #6880) (#6883)

## Other

* Upgrade `next` to `16.2.6` to address high-severity advisories: GHSA-8h8q-6873-q5fj (DoS via Server Components), GHSA-267c-6grr-h53f and GHSA-26hh-7cqf-hhc6 (middleware/proxy bypass via segment-prefetch), GHSA-mg66-mrh9-m8jx (DoS via cache connection exhaustion), GHSA-492v-c6pp-mqqv (dynamic route parameter injection bypass), GHSA-c4j6-fc7j-m34r (SSRF via WebSocket upgrades), GHSA-36qx-fr4f-26g5 (i18n proxy bypass) (#6677)
* Bump `qs` and `express` to address stringify-related correctness issues (#6854)
* Bump `@epam/ai-dial-ui-kit` (#6872)
* Bump `tiktoken` (Issue #6506) (#6529)
* Update CI to use trusted publishing to npm registry (#6885)
* Bump vitest from 4.0.9 to 4.1.8 (https://github.com/epam/ai-dial-chat/pull/6977)

## Deployment Changes

### New environment variables

| Variable | Default | Description |
|---|---|---|
| `NEXT_PUBLIC_RESOURCE_MAX_SEGMENT_BYTES` | `255` | Maximum UTF-8 byte length of a single path segment in entity names (files, folders, conversations). Must be a positive integer less than 1024. Must be set at **build time** (Next.js public env). |

### Behavioral changes

> [!NOTE]
> Applications now use the `dial-app` toolset type internally (previously `dial-deployment` and `dial-mcp`). No operator action needed; models continue to use `dial-deployment`.

- **dial-app toolset type** — applications and toolset configuration view (Issue #6517) (#6742)
- **Entity name byte limits** — entity name validation now enforces UTF-8 byte budgets (max 255 bytes per path segment by default, configurable via `NEXT_PUBLIC_RESOURCE_MAX_SEGMENT_BYTES`) instead of the previous 160-character limit (Issue #3808) (#6613)

---

#### ai-dial-core `0.44.4`

## Other

* add debug messages for application listing API (#1616)

---

#### ai-dial-analytics-realtime `0.24.1`

## Other

* bump fastapi, starlette and pydantic (#264)
* bump transformers 5.0.0rc3 to 5.3.0 (#263)

---

#### ai-dial-rag `0.42.0`

## Other

* update libreoffice to 25.8.6 (#158)
* chore(ci): add release candidate branching (#154)
* bump aiohttp from 3.13.3 to 3.13.4 (#145)
* bump cryptography from 46.0.5 to 46.0.7 (#146)
* bump langsmith from 0.4.27 to 0.7.31 (#152)
* bump lxml from 5.3.0 to 6.1.0 (#156)
* bump nltk from 3.9.3 to 3.9.4 (#137)
* bump pillow from 12.1.1 to 12.2.0 (#148)
* bump pygments from 2.18.0 to 2.20.0 (#141)
* bump pypdf from 6.9.1 to 6.10.2 (#153)
* bump python-dotenv from 1.1.0 to 1.2.2 (#155)
* bump python-multipart from 0.0.22 to 0.0.26 (#150)
* bump requests from 2.32.5 to 2.33.0 (#135)
* bump the ai-dial-ci group with 4 updates (#159) (#161)

---

#### ai-dial-log-parser `0.3.0`

## Fixes

* do not fail if root of assembled_response is a list (#21)

## Other

* add .ort.yml (#22)
* update discord link (#19)

---

#### ai-dial-code-interpreter `0.2.0`

## Changes
* add session id verification (#12)
* add /health endpoint
* compile py files to speed up startup time
* do not write the rest of compiled files at runtime
* fix waiting for client to be ready and executing code under a lock
* update dependencies

---

#### ai-dial-app-controller `0.4.0`

## Fixes

* security fixes (#43)
* update k8s client 22 -> 25 (#55)

## Other

* deploy to the new dev (#60)
* fix copy/paste in readme (#62)
* refresh CI (#56)
* sync ci according to ai-dial-ci (#45)
* other dependency updates

---

#### ai-dial-app-builder-python `0.1.0`

Initial version

---

#### ai-dial-quickapps-backend `0.8.0`

## Features

* New `dial-app` toolset — given a DIAL deployment id, routes transparently to either MCP (when the deployment advertises `features.mcp == true`) or chat completion (otherwise); a `transport: "auto" | "mcp" | "chat-completion"` field (default `"auto"`) lets admins pin the choice #215 (#243)
* [Preview] New DIAL files toolset — exposes `list_files`, `read_file_lines`, `search_in_file`, `write_file`, `edit_file`, `delete_file`, `copy_file`, `move_file` against an `agent_home_dir` (default `files/{appdata}/`); per-app opt-in via `features.dial_files` with optional `enabled_tools` allowlist #256 (#257)
* Per-app overrides for predefined tool and toolset templates via a new optional `override` field carrying a JSON Merge Patch (RFC 7396) — ChatHub variants can swap a deployment, tweak a tool description, or disable a single tool per-app without forking templates globally or duplicating inline JSON #14 #273 (#269)
* Layered predefined default application configuration for the QuickApps builder — operators ship a `default_configuration.json` at each predefined content layer (built-in + `PREDEFINED_EXTRA_PATHS`), shallow-merged in layer order, exposed to the UI via the new `GET /v1/configuration-support/default-configuration` endpoint with `tool_sets` resolved server-side #120 (#323)
* Generic synthetic tool-call injector (v2) — replaces the bespoke read-files plumbing with a reusable mechanism for injecting synthetic tool calls into message history #235 (#255)
* [Preview] Config-driven synthetic tool-call injection (hooks) — declare `(ASSISTANT/tool_calls, TOOL)` injections in the app manifest's new `hooks` array without writing Python; `on_request_start` event with `always` / `append_if_changed` frequency (#275)
* Default orchestrator deployment via the new `DEFAULT_ORCHESTRATOR_DEPLOYMENT_ID` env variable — applied at runtime when an app manifest omits `orchestrator.deployment`, and surfaced as the JSON-schema `default` so DIAL Core can pre-fill new manifests #315 (#316)
* Per-app `features.stage_display.level` — `error` / `info` (default) / `debug` thresholds for which tool-execution stages surface in the DIAL UI; `debug` reveals synthetic/system stages for manifest authors; deprecates `display.stage.show = false` (still honored at `info` with a warning) #293 (#307)
* Pass orchestrator deployment's native configuration tools alongside QuickApps tools — enables Gemini 3+ native function-calling features (e.g. Google Search, code execution) when the orchestrator deployment declares them #104 (#301)
* Type-scoped DIAL Core routes for `/v1/configuration-support/*` — co-located with the QuickApps application type via `dial:applicationTypeRoutes` instead of a detached global route, and `dial:attachmentPaths` lets DIAL Core enforce ACL on prompt URLs in `skills/validate` request bodies #318 (#319)
* Trust custom corporate root CAs via the new `USE_SYSTEM_CA_CERTS` opt-in — merges `*.crt` files mounted under `/certificates/` with the Alpine system bundle at container startup and exports `SSL_CERT_FILE` so outbound `httpx` calls succeed behind TLS-intercepting proxies #335 (#336)
* Lenient skill frontmatter parsing aligned with the [Agent Skills spec](https://agentskills.io/client-implementation/adding-skills-support) — tolerates BOM, CRLF, leading whitespace, trailing fences, and unquoted-colon descriptions; cosmetic violations become warnings instead of skipping the skill; affects both predefined and DIAL-prompt-sourced skills #313 (#314)
* Configurable agent file-download size limit — replaces the hardcoded 10 MiB cap in `DialFileService` with `DEFAULT_FILE_LOADING_SIZE_LIMIT` (deployment-wide) and `features.file_loading.size_limit` (per-app override) (#274)
* Graduate Time Awareness feature to GA — `features.timestamp` is no longer gated by `ENABLE_PREVIEW_FEATURES` #288 (#290)
* Graduate DIAL Prompt Skills feature to GA — the `skills` config field and `DialPromptSkillsModule` are no longer gated by `ENABLE_PREVIEW_FEATURES` #289 (#291)

## Fixes

* Apply the configured fallback strategy when an MCP tool result carries `IsError: true` — previously the error path skipped the fallback and surfaced the raw failure to the model #328 (#329)
* Propagate DIAL field-level markers (`dial:resource`, `dial:file`, `format`) to deprecated legacy-alias properties — manifests stored under a legacy key were validating but DIAL Core's schema-driven auto-share couldn't key off the missing markers, surfacing as `403 Forbidden` on referenced resources #311 (#312)
* Render Agent skill stage content as a fenced code block and drop the redundant parameters chunk — embedded `#`/`##` headings in skill bodies no longer render as oversized bold text when the stage is expanded #320 (#322)
* Enrich synthetic tool results with metadata — synthetic tool messages now flow through the same `ToolCallResultEnricher` pipeline as real tool results, so message state (e.g. timestamp) is attached consistently #296 (#297)
* Restore OTEL `trace_id` / `span_id` / `resource.service.name` / `trace_sampled` in QuickApps logs via a new `OtelAwareFormatter` that matches `adapter-openai`'s wire format when correlation is on and renders nothing otherwise #284 (#285)
* Preserve millisecond precision in `TimestampMetadata.response_timestamp` — Pydantic's default `datetime.isoformat()` was dropping fractional digits on whole-second boundaries, leaving downstream consumers (UI, analytics, persistence) with inconsistent sub-second precision #278 (#281)
* Align integration test modules after recent refactors (#276)

## Other

* Normalize DIAL deployment-id field naming — `DialDeploymentConfig.name` → `deployment_id` and `DialMCPToolSet.dial_id` → `deployment_id`; legacy keys keep validating via `validation_alias`, and the published JSON schema retains deprecated siblings under an `anyOf` so existing manifests stay valid #286 (#287)
* Refresh integration coverage for the current model lineup — add Claude 4.6 tests, update Claude 4.5 model name, add ChatHub configs for the new models (#277); remove GPT-4.1 tool cache files and Claude 3.7/4 references (#283) (#294)
* Enhance issue templates and add a `🛠️ Tech debt` template (#292)
* Drop venv-activation hints from `CLAUDE.md` — `make` targets already run via `poetry run` (#272)
* Bump `authlib` from 1.6.11 to 1.6.12 — picks up a fix for redirecting to an unvalidated `redirect_uri` on `InvalidScopeError` in OIDC implicit/hybrid grants (#299)
* Bump `idna` from 3.11 to 3.15 — picks up CVE-2026-45409 mitigation hardening from 3.14 (#321)
* Bump `python-multipart` from 0.0.26 to 0.0.27 (#280)
* Bump `urllib3` from 2.6.3 to 2.7.0 (#295)

## Deployment Changes

### New environment variables

| Variable                             | Default    | Description                                                                                                                                                                                                                                     |
|--------------------------------------|------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `DEFAULT_ORCHESTRATOR_DEPLOYMENT_ID` | —          | Default DIAL deployment id used as the orchestrator model when a QuickApp manifest omits `orchestrator.deployment`. Also surfaces as the JSON-schema `default` for that field so DIAL Core can pre-fill new manifests. Apps can override per-app. |
| `DEFAULT_FILE_LOADING_SIZE_LIMIT`    | `10485760` | Deployment-wide cap (bytes, `> 0`; default 10 MiB) on files the agent downloads. Overridden per-app by `features.file_loading.size_limit` in the manifest.                                                                                       |
| `USE_SYSTEM_CA_CERTS`                | unset      | When set to `1`, merges every `*.crt` file under `/certificates/` with the Alpine system CA bundle at container startup and exports `SSL_CERT_FILE` to the merged path so outbound HTTP calls trust private/corporate root CAs. Opt-in; unset keeps existing behaviour. |

### Behavioral changes

> [!NOTE]
> Two preview-gated features have graduated to GA and are now active regardless of `ENABLE_PREVIEW_FEATURES`:
>
> - **Time Awareness** — `features.timestamp` (#290)
> - **DIAL Prompt Skills** — the `skills` config field and `DialPromptSkillsModule` (#291)

### DIAL Configuration changes

> [!IMPORTANT]
> Operators must update DIAL Core's configuration when upgrading to this release. The `/v1/configuration-support/*` endpoints are no longer served via a global DIAL Core `routes` entry — they are declared on the QuickApps application type itself.
>
> **Required migration** (#319):
>
> 1. **Remove** any `quick_apps2`-style entry from DIAL Core's global `routes` block (it will be ignored from now on).
> 2. **Add** the new `dial:applicationTypeRoutes` block to the QuickApps entry under `applicationTypeSchemas` — apply the schema snippet from [PR #319](https://github.com/epam/ai-dial-quickapps-backend/pull/319) verbatim.

### Schema deprecations

> [!CAUTION]
> Still accepted in app manifests, but will be removed in future versions (#287).

| Legacy key                | Replacement     | Affected config model    |
|---------------------------|-----------------|--------------------------|
| `name` (deployment field) | `deployment_id` | `DialDeploymentConfig`   |
| `dial_id`                 | `deployment_id` | `DialMCPToolSet`         |

---

#### ai-dial-mind-map-backend `0.14.1`

## Features

* support public_url (#76)

---

#### ai-dial-mind-map-frontend `0.13.0`

## Features

* enable withCredentials for document loading in PdfContent component (#87)
* enhance PDF URL resolution with getReferenceUrl utility
* support public URLs for file sources and reference links (#83)

## Other

* bump @tootallnate/once from 2.0.0 to 2.0.1 (#76)
* bump dataaxiom/ghcr-cleanup-action from 1.2.1 to 1.2.2 in the github-actions group (#85)
* bump js-cookie from 3.0.5 to 3.0.7 (#75)
* bump protobufjs, @opentelemetry/auto-instrumentations-node and @opentelemetry/sdk-node (#71)
* bump shell-quote from 1.8.3 to 1.8.4 (#86)
* bump the ai-dial-ci group with 4 updates (#79)
* bump the ai-dial-ci group with 4 updates (#84)
* bump the ai-dial-ci group with 4 updates (#88)

---

#### ai-dial-admin-backend `0.17.0`

## UPGRADE TO NEW RELEASE ##
**Please review [upgrade plan](https://github.com/epam/ai-dial-admin-backend/blob/0.17.0/docs/upgrade-plans/0.17.0.md) before new release installation.**

## INFRASTRUCTURE CHANGELOG ##
**Please review [infrastructure changelog](https://github.com/epam/ai-dial-admin-backend/blob/0.17.0/docs/INFRA-CHANGELOG.md) before new release installation.**

## BREAKING CHANGES (configuration)

* #783 Removed deprecated `allowedRoles` config property and changed default `config.rest.security.default.roles-mapping` from `{}` to `{"ConfigAdmin":["FULL_ADMIN"],"admin":["FULL_ADMIN"]}` (#943)
See [Changed Security & RBAC](https://github.com/epam/ai-dial-admin-backend/blob/0.17.0/docs/INFRA-CHANGELOG.md#security--rbac), [Removed Security & RBAC](https://github.com/epam/ai-dial-admin-backend/blob/0.17.0/docs/INFRA-CHANGELOG.md#security--rbac-1)
* #773 Removed default page size for analytics queries: `METRICS_INFLUX2_DEFAULT_PAGE_SIZE` and `METRICS_INFLUX3_DEFAULT_PAGE_SIZE` env vars are no longer supported (#984)
See [Removed Observability](https://github.com/epam/ai-dial-admin-backend/blob/0.17.0/docs/INFRA-CHANGELOG.md#observability)

## Features

* #940 Supported several OAuth token endpoint auth methods in toolset: `HTTP Basic Authentication`, `HTTP Request Body`, `None (clientSecret is not provided)` (#996)
* #986 Supported `in` filter operator for historical queries: `getRevisions`, `getAuditActivities` (#990)
* #895 Included uniqueness conflict validation into import results for toolsets, applications, and files instead of failing the request (#949, #980)
* #896 Added "Conversation" as a managed asset (#965)
* #908 Transferred tools retrieval to the new `/v1/toolset/{path}/tools` Core endpoint (#991)
* #1011 Supported `id` for model upstreams (#1012)
* Added new `/api/v1/deployments` endpoint to get Core deployments, supports `interface_types` and `deployment_types` filters in query params (#982)

## Fixes

* #1003 Handled `null` version in path for "Conversation" approvals (#1002)
* #1004 Added `source` field to the applications listing response (#1005)
* #1014 Adjusted "Sync with core" functionality for Routes to prevent "Out of sync" status (#1016)
* #1019 Prevented duplicate IDs between interceptors and deployments (#1020, #1021)
* epam/ai-dial-admin-frontend#3451 Allowed `STRING`/`UUID` type coercion in query-language filters (#989)
* Fixed docs for release 17 (#1018)

## Other

* improve config filtering against Core version (#951)
* bump org.apache.tomcat.embed:tomcat-embed-core from 11.0.21 to 11.0.22 (#983)
* bump the ai-dial-ci group with 4 updates (#972, #992, #1009)
* bump the github-actions group across 1 directory with 2 updates (#973, #993)
* add documentation for 0.17.0 release (#1008)

---

#### ai-dial-admin-frontend `0.17.1`

## Fixes

* [Approvals > Application Publications] Fallback entity icon fails to load due to malformed URL encoding  (Issue #3530) (#3532)

---

#### ai-dial-admin-deployment-manager-backend `0.17.0`

## BREAKING CHANGES

* #285 Replace node pool label-key/capacity config with explicit Kubernetes scheduling primitives; 'NODE_POOLS' is now a YAML document and adds 'NODE_POOL_DEFAULT' / 'NODE_POOL_DEFAULT_MODEL' create-time defaults (#319)
* #328 Remove deprecated 'config.rest.security.default.allowedRoles' and 'providers.*.allowed-roles'; migrate to 'roles-mapping' (#330)

## Features

* #285 Node Pool Configuration is now generally available, with explicit 'nodeSelector' / 'affinity' / 'tolerations' primitives per pool and create-time default stamping (#319)
* #320 Add API-key authentication via DIAL Core, validating the 'Api-Key' header against Core's '/v1/user/info' alongside the existing JWT/OIDC flow (#321)
* #286 Add per-resource revision rollback for deployments, image definitions, and the global image-build domain whitelist (#327)
* #87 Auto-detect HuggingFace text-classification models and emit a chained KServe predictor + transformer 'InferenceService' (#334)
* #324 Support Git Dockerfile source for non-MCP image definitions (Application, Adapter, Interceptor) (#325)

## Fixes

* #78 Support Docker Hub and bearer-auth registries (ACR/GHCR/GAR/ECR) on MCP LOCAL transport builds (#339)
* #294 Detect builder-push container failures early so failed builds flip to 'BUILD_FAILED' in seconds instead of after the full retry budget (#337)
* #286 Align rollback revision lookup with snapshot semantics so a gap revision number resolves to the most recent applicable earlier state (#338)

## Docs

* Add a warning note regarding H2 usage (#326)
* Update mcp-proxy information and remove stale files (#335)
* Update release notes for release 17 (#340)

## Other

* Improve Claude Code and spec-kit configuration, including per-layer 'CLAUDE.md' files and the 'specs/' index (#311)
* Bump dependencies to secure versions (#318, #329)
* Bump the ai-dial-ci group with 4 updates (#322, #331, #341, #349)
* Bump the github-actions group (#314, #323, #332, #342)

## Deployment Changes

This release includes **many critical and high-priority changes**. Please review the [full upgrade guide](https://github.com/epam/ai-dial-admin-deployment-manager-backend/blob/0.17.0/docs/upgrade-plans/0.17.0.md) before proceeding.

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
