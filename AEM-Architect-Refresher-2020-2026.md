# AEM Architect Refresher 2020 → 2026

As of Nov 18, 2026

## How to use this guide

The single biggest change since 2020: AEM is now a cloud-native, continuously updated product (AEM as a Cloud Service, "AEMaaCS"), with Edge Delivery Services and Universal Editor as the new default for building sites. On-Prem and Adobe Managed Services (AMS) still exist, now on AEM 6.5 LTS.

Each module follows the same pattern: **2020 (what you knew)** → **Now (2026)** → **Architect view** → **Code / config sample** → **Interview angle** → **References**. Each module ends with a \*\*Deep dive (handbook)\*\* section: step-by-step setup, complete config files and code, runbooks, and expert-level answers. Read Module 1 and the cheat sheet below first; then go module by module, overview first, deep dive second. Module 14 is a rehearsal set of interview questions.

### 2020 vs 2026 cheat sheet

| Area | 2020 (what you remember) | 2026 (what to say now) |
| --- | --- | --- |
| Product line | AEM 6.4 / 6.5 on-prem or AMS; AEMaaCS just launched (Jan 2020) | AEMaaCS is the default; AEM 6.5 LTS for on-prem/AMS; Edge Delivery Services (EDS) for high-performance sites |
| Upgrades | Big-bang upgrades every 2-3 years | AEMaaCS: Adobe pushes updates continuously (no upgrade projects). 6.5 LTS: service packs |
| Infrastructure | Fixed author/publish VMs, you size them | Containers on Kubernetes, auto-scaling publish, Adobe-managed CDN (Fastly) included |
| Repository | Everything mutable; CRXDE on prod | Immutable `/apps` + `/libs` (code only via pipeline); mutable `/content`, `/conf`, `/var` |
| Deployment | Package Manager / Jenkins / CRX upload | Cloud Manager pipelines only (full-stack, front-end, web-tier, config); RDE for fast dev loops |
| OSGi config | `sling:OsgiConfig` nodes, many custom run modes | `.cfg.json` files; only author/publish + dev/stage/prod run modes; env variables and secrets |
| Repo setup | Content packages with users, ACLs | `repoinit` scripts for service users, ACLs, folders |
| Replication | Replication agents | Sling Content Distribution via Adobe pipeline (no agents to configure); Preview tier |
| Front end | Clientlibs inside `ui.apps`, static or editable templates | `ui.frontend` webpack build, front-end pipeline, Site Themes; Core Components + Style System everywhere |
| Authoring | Page Editor (Touch UI) | Page Editor still works, but Universal Editor and document-based authoring (Google Docs / SharePoint / Document Authoring) are the new path |
| Headless | Content Services, Sling Model exporter, early GraphQL | Content Fragments + GraphQL persisted queries + OpenAPI delivery APIs; SPA Editor deprecated for new projects in favour of Universal Editor |
| Site delivery | Dispatcher + your CDN, Lighthouse 60-80 typical | EDS: static-like delivery from the edge, Lighthouse 100 is the target; classic Dispatcher still used for AEMaaCS Sites |
| Security | Dispatcher filters, your WAF | Traffic filter rules + WAF declared in `cdn.yaml`; Adobe IMS for author login |
| Assets | DAM Update Asset workflow, ImageMagick | Asset microservices (cloud processing); Assets view; Content Hub; Dynamic Media with OpenAPI |
| Forms | Foundation adaptive forms, AEM Forms JEE | Adaptive Forms Core Components, headless forms, forms on EDS, Communications APIs |
| Java | Java 8 / 11 | Java 17 / 21 (AEMaaCS runtime on Java 21; 6.5 LTS supports Java 17/21) |
| Monitoring | Your own Splunk/AppDynamics agents | Cloud Manager logs, log forwarding via config pipeline, Developer Console |
| AI | Smart Tags (Sensei) | GenAI: Generate Variations, AI Assistant, Sites Optimizer, agentic AI in AEM |

## Change Catalogue: everything that changed in AEM, 2020 → 2026

This catalogue lists **143 changes** in 12 areas, one line each, with the deployment model it applies to and the module that explains it in depth. Use it as the master checklist: if you can explain every row, you are up to date.

**Applies to**: *Cloud* = AEM as a Cloud Service only; *LTS* = AEM 6.5 LTS on-prem/AMS; *All* = every model; *EDS* = Edge Delivery Services.

| Area | Changes | Rows |
| --- | --- | --- |
| A. Platform and deployment models | 18 | 1-18 |
| B. DevOps and Cloud Manager | 17 | 19-35 |
| C. Backend coding | 18 | 36-53 |
| D. Front end and authoring | 15 | 54-68 |
| E. Edge Delivery Services | 13 | 69-81 |
| F. Headless | 6 | 82-87 |
| G. Dispatcher, CDN and security | 11 | 88-98 |
| H. Assets | 11 | 99-109 |
| I. Forms | 12 | 110-121 |
| J. Integrations and extensibility | 9 | 122-130 |
| K. Migration tooling | 5 | 131-135 |
| L. Operations and AI | 8 | 136-143 |

### A. Platform and deployment models (architecture)

| # | Change | Applies to | What it means | See |
| --- | --- | --- | --- | --- |
| 1 | AEM as a Cloud Service became generally available (2020) | Cloud | Cloud-native AEM operated by Adobe; the strategic default | M1 |
| 2 | Continuous updates replace upgrades (daily maintenance, monthly features, prerelease channel) | Cloud | No upgrade projects; regression testing becomes continuous | M2.4 |
| 3 | AEM 6.5 LTS released (Mar 2025) with Java 17/21 and refreshed libraries | LTS | The path for clients who stay on-prem or AMS | M1, M12.5 |
| 4 | Classic AEM 6.5 support end dates (core Feb 2027, extended Feb 2028) | LTS | Clients must plan LTS upgrade or cloud move | M1 |
| 5 | Containerized architecture on Kubernetes with auto-scaling publish | Cloud | No server sizing; design for disposable pods | M2.1 |
| 6 | Preview tier added beside author and publish | Cloud | Review content on a publish-like URL before go-live | M2.1 |
| 7 | Golden master + Sling Content Distribution replace replication agents | Cloud | No agents or queues to configure; no reverse replication | M2.6 |
| 8 | Immutable `/apps` and `/libs` (composite node store) | Cloud | Code only via pipeline; no CRXDE edits, no overlays | M2.2, M2.6 |
| 9 | Shared cloud blob store with direct binary access | Cloud | Binaries never pass through the JVM | M2.1, M9.5 |
| 10 | Adobe-managed CDN (Fastly) included | Cloud | CDN becomes part of your design and config | M8 |
| 11 | Program/environment model: prod+stage, dev, RDE, sandbox, specialized testing | Cloud | New vocabulary for planning environments | M3.8 |
| 12 | Additional publish regions (multi-region delivery) | Cloud | Lower latency and regional resilience as an add-on | M13 |
| 13 | Advanced Networking: flexible egress, dedicated egress IP, VPN | Cloud | How AEM reaches IP-restricted back ends | M2.3, M8.5 |
| 14 | Self-service restore and Content Copy tool | Cloud | Restore points and prod→lower copies from Cloud Manager | M3.6 |
| 15 | Sandbox programs that hibernate | Cloud | Cheap trial/POC environments | M13.3 |
| 16 | Cloud Manager for AMS (classic) vs Cloud Manager for AEMaaCS | All | AMS uses Cloud Manager too, but with VM-based deployment | M1.2 |
| 17 | Metered licensing (content requests, storage, add-ons) | Cloud | Caching and bot control affect cost | M1.5 |
| 18 | Java 21 runtime; Java 25 runtime announced (2027) | Cloud | Keep builds and plugins current | M3.3 |

### B. DevOps and Cloud Manager

| # | Change | Applies to | What it means | See |
| --- | --- | --- | --- | --- |
| 19 | Cloud Manager is the only deployment path | Cloud | No Package Manager/Jenkins deploys of code | M3 |
| 20 | Full-stack production and non-production pipelines | Cloud | Build → stage → tests → approval → prod | M3.1, M3.8 |
| 21 | Front-end pipeline | Cloud | Deploy CSS/JS themes in minutes | M3.1, M5.4 |
| 22 | Web-tier config pipeline | Cloud | Deploy Dispatcher config alone | M3.1 |
| 23 | Config pipeline (CDN rules, WAF, log forwarding, maintenance, alerts) | Cloud | Operations as code in YAML | M3.1, M8.3 |
| 24 | Code quality gates (SonarQube + Adobe rules + OakPAL) | Cloud | Critical issues block deployments | M3.2, M3.8 |
| 25 | Experience audit (Lighthouse) in the pipeline | Cloud | Performance measured on every release | M3.2 |
| 26 | Custom functional and UI tests (Docker-based) gate production | Cloud | Automated tests are mandatory in practice | M3.8 |
| 27 | Bring-your-own Git (GitHub, GitLab, Bitbucket, Azure DevOps) | Cloud | Pipelines from the client's repo; PR checks | M3.3, M3.8 |
| 28 | Environment variables and secrets (`$[env:]`, `$[secret:]`) | Cloud | No secrets in code; per-environment values | M3.3 |
| 29 | Pipeline (build-time) variables | Cloud | Private Maven repo credentials etc. | M3.8 |
| 30 | Rapid Development Environments (RDE) | Cloud | Deploy to a cloud env in seconds | M3.4 |
| 31 | AEM SDK and `aem-sdk-api` dependency replace the uber-jar | Cloud | Local dev matches the cloud release | M3.5 |
| 32 | Dispatcher SDK tools (`validate.sh`, Docker run) | Cloud | Validate Dispatcher locally exactly as the pipeline does | M3.5, M8.1 |
| 33 | AEM Analyser Maven plugin | Cloud | Cloud build checks run locally | M3.5 |
| 34 | Developer Console | Cloud | Read-only view of bundles, configs, queries, CRX | M3.6 |
| 35 | Adobe I/O CLI plugins (Cloud Manager, RDE, migration) | Cloud | Scriptable operations | M3.8 |

### C. Backend coding

| # | Change | Applies to | What it means | See |
| --- | --- | --- | --- | --- |
| 36 | Archetype restructured: `ui.apps.structure`, `ui.config`, `ui.frontend`, `all`, `it.tests`, `ui.tests` | All | Standard module layout everyone expects | M4.1 |
| 37 | Strict package types (`application` / `content` / `container`) | Cloud | Mixed packages fail the build | M4.1 |
| 38 | OSGi configs as `.cfg.json`; XML `sling:OsgiConfig` unsupported | Cloud (recommended for LTS) | New config file format | M4.2 |
| 39 | Run modes limited to author/publish + dev/stage/prod | Cloud | Use env vars, secrets, CAC instead of custom run modes | M4.2 |
| 40 | Repoinit for service users, groups, ACLs, paths | Cloud (also LTS) | No users/ACLs in content packages | M4.3 |
| 41 | Oak indexes as code: Lucene only, `-custom-n` naming, reindex before switch | Cloud | Index changes deploy safely via pipeline | M4.6 |
| 42 | Sling Jobs + `scheduler.runOn=SINGLE` for background work | Cloud | Avoid duplicate execution across pods | M4.5 |
| 43 | Stateless code: no local file persistence, no sticky sessions | Cloud | Pods are disposable | M2.6 |
| 44 | Limits on long-running requests and heavy traversal queries | Cloud | Make heavy work asynchronous; index every query | M2.6, M4.9 |
| 45 | Removed/unexported APIs (e.g. Guava, Commons Collections 3) | Cloud, LTS | Refactor old dependencies | M4.7 |
| 46 | Modern Java (11/17/21): records, text blocks, switch expressions, `var` | Cloud, LTS | Cleaner code; Java 8 builds unsupported | M4.7 |
| 47 | `Replicator` API backed by Content Distribution; publish to Preview | Cloud | Same API, new mechanism | M4.7 |
| 48 | Workflows on author only, kept short; transient workflows | Cloud | No heavy processing in workflows | M4.7 |
| 49 | Outbound HTTP through Advanced Networking proxy (JVM proxy settings, `AEM_*` env vars) | Cloud | Integration code must honour the proxy | M4.9 |
| 50 | OAuth Server-to-Server replaces JWT service credentials | All Adobe APIs | Update integration auth | M7.3 |
| 51 | Context-Aware Configuration as the standard for per-site settings | All | Multi-brand without run modes | M4.9 |
| 52 | Extending Core Components via delegation (`@Via ResourceSuperType`) as standard | All | Never copy Core Components | M4.4 |
| 53 | Unit testing with AEM Mocks + coverage gate | Cloud | 50% coverage default threshold | M4.9 |

### D. Front end and authoring

| # | Change | Applies to | What it means | See |
| --- | --- | --- | --- | --- |
| 54 | Core Components matured (Image v3, Teaser, Embed, Tabs, Accordion, Content Fragment, etc.) | All | Build on Core, extend via proxies | M5.1 |
| 55 | Web-optimized image delivery (WebP/AVIF, resizing) | Cloud | Faster images without Dynamic Media | M5.7 |
| 56 | Style System as the default for variants | All | One component, many looks | M5.2 |
| 57 | Static templates legacy; editable templates + policies only | All | Template Editor everywhere | M5.1, M5.7 |
| 58 | `ui.frontend` webpack/TypeScript build + clientlib generator | All | Modern front-end tooling | M5.3, M5.7 |
| 59 | Site Templates and Themes | Cloud | Create sites from templates; theme projects | M5.4 |
| 60 | Adobe Client Data Layer built into Core Components | All | Standard analytics data model | M5.7 |
| 61 | Universal Editor (framework-agnostic visual editor) | Cloud | Edits HTL, React, Next.js, EDS pages | M5.5, M7.4 |
| 62 | SPA Editor deprecated for new projects (2025.1.0) | Cloud (6.5 later SPs) | Use Universal Editor for new headless work | M5.5 |
| 63 | New Content Fragment Editor and Content Fragments console | Cloud | Faster structured authoring | M7.1 |
| 64 | New admin UIs (Assets view, Experience Cloud unified shell) | Cloud | Different look and navigation for authors | M9.1 |
| 65 | Accessibility becomes a legal requirement (EU EAA from June 2025, WCAG 2.2) | All | Accessibility testing in every project | M5.7 |
| 66 | Core Web Vitals focus (INP replaced FID in 2024) | All | Performance targets change | M5.7 |
| 67 | Machine translation built in; CF translation with references | Cloud | Cheaper translation flows | M11.7 |
| 68 | MSM rollouts as background jobs | Cloud | Large rollouts don't block authors | M11.7 |

### E. Edge Delivery Services

| # | Change | Applies to | What it means | See |
| --- | --- | --- | --- | --- |
| 69 | Edge Delivery Services GA (2023, ex-Franklin/Helix) | EDS | New delivery architecture with Lighthouse 100 target | M6 |
| 70 | Document-based authoring (Google Docs / SharePoint) | EDS | Authors write in Word/Docs | M6.2 |
| 71 | Document Authoring (da.live) | EDS | Adobe's browser-based doc editor | M6.2 |
| 72 | Universal Editor authoring on AEM ("xwalk") | EDS + Cloud | WYSIWYG with MSM, workflows, translation | M6.2, M6.8 |
| 73 | Blocks in vanilla JS/CSS, no build, eager/lazy/delayed loading | EDS | New front-end development model | M6.3, M6.8 |
| 74 | Code Sync GitHub app, branch previews (`aem.page` / `aem.live`) | EDS | Every branch is an environment | M6.1 |
| 75 | Sidekick browser extension for preview/publish | EDS | New author workflow | M6.1 |
| 76 | Spreadsheets as JSON, query index, sitemap, redirects, headers config | EDS | Data and SEO handled by config | M6.5, M6.8 |
| 77 | Built-in RUM (operational telemetry) | EDS | Real-user CWV for every page | M6.4 |
| 78 | Built-in experimentation | EDS | Simple A/B tests without Target | M6.5 |
| 79 | Admin API for automation | EDS | Script preview/publish/status | M6.8 |
| 80 | EDS sites onboarded in Cloud Manager (domains, CDN) | EDS + Cloud | Single place to manage go-live | M6.5 |
| 81 | Forms on EDS and Commerce storefront on EDS | EDS | Same performance model for forms and commerce | M6.5, M10, M11.3 |

### F. Headless

| # | Change | Applies to | What it means | See |
| --- | --- | --- | --- | --- |
| 82 | GraphQL API for Content Fragments GA with persisted queries | Cloud, 6.5 SPs | Cacheable, secure headless delivery | M7.2 |
| 83 | Richer CF models (references, validation, enums, UI tabs) | Cloud | Better structured content | M7.1, M7.7 |
| 84 | OpenAPI-based delivery and management APIs (Sites, Assets, Forms) | Cloud | Versioned, documented APIs | M7.3 |
| 85 | Assets HTTP API support for Content Fragments deprecated | Cloud | Move CF integrations to new APIs | M7.3 |
| 86 | AEM Headless client SDKs (JS, Java, React) | All | Faster headless app development | M7.2, M7.7 |
| 87 | Universal Editor replaces SPA Editor for in-context headless editing | Cloud | New hybrid authoring pattern | M7.4 |

### G. Dispatcher, CDN and security

| # | Change | Applies to | What it means | See |
| --- | --- | --- | --- | --- |
| 88 | Dispatcher cloud format, immutable files, flexible mode | Cloud | Fixed structure validated by Adobe | M8.1, M8.7 |
| 89 | Automatic Dispatcher invalidation (no flush agents) | Cloud | One less thing to configure | M8.1 |
| 90 | CDN configuration as code (`cdn.yaml`) | Cloud | Edge rules in Git, deployed in minutes | M8.3, M8.7 |
| 91 | Traffic filter rules, rate limiting and WAF flags (WAF licensed) | Cloud | Bot and attack protection at the edge | M8.3 |
| 92 | Edge redirects, origin selectors, request/response transformations, error pages | Cloud | Logic moves from Apache to the CDN | M8.3, M8.7 |
| 93 | CDN purge API with purge keys | Cloud | Instant invalidation for urgent changes | M8.7 |
| 94 | Bring-your-own CDN with `X-AEM-Edge-Key` | Cloud | Keep Akamai/Cloudflare in front | M8.2 |
| 95 | Adobe IMS login for authors (federated SSO) | Cloud | No local/LDAP author users | M8.4 |
| 96 | Publish-side SAML and OIDC handlers; user sync across pods | Cloud | Customer login on a scaled-out publish tier | M8.4 |
| 97 | IP allowlists, custom domains and SSL managed in Cloud Manager | Cloud | Self-service network settings | M8.5 |
| 98 | Enhanced Security / WAF-DDoS Protection add-ons | Cloud | Licensed security tiers | M8.3 |

### H. Assets

| # | Change | Applies to | What it means | See |
| --- | --- | --- | --- | --- |
| 99 | Asset microservices replace DAM Update Asset processing | Cloud | Renditions and metadata in cloud services | M9.1, M9.5 |
| 100 | Asset Compute custom workers (App Builder) | Cloud | Custom renditions without JVM code | M9.2, M9.5 |
| 101 | Post-processing workflows per folder | Cloud | Light customization after processing | M9.1 |
| 102 | Direct binary upload (`aem-upload`) | Cloud | Fast, scalable uploads | M9.2 |
| 103 | Assets view (ex Assets Essentials) and Admin view | Cloud | Simplified UI for marketers | M9.1 |
| 104 | Content Hub for brand-safe asset distribution | Cloud | Self-service portal for approved assets | M9.1 |
| 105 | Dynamic Media with OpenAPI; Dynamic Media Prime/Ultimate | Cloud | Deliver approved assets anywhere via URNs | M9.1, M9.5 |
| 106 | Asset Selector micro-frontend | Cloud | Pick AEM assets inside any app | M9.5 |
| 107 | Bulk Import from S3/Azure | Cloud | Large ingestion without custom scripts | M9.1 |
| 108 | AI for assets: AI metadata, Firefly generative edits, content credentials | Cloud | Faster tagging and variant creation | M9.1 |
| 109 | Deeper Workfront and Creative Cloud/Express integration | Cloud | Content supply chain end to end | M9.5 |

### I. Forms

| # | Change | Applies to | What it means | See |
| --- | --- | --- | --- | --- |
| 110 | AEM Forms as a Cloud Service | Cloud | Forms without on-prem servers | M10.1 |
| 111 | Adaptive Forms Core Components | Cloud, 6.5 | Modern, accessible, themeable forms | M10.1 |
| 112 | Headless adaptive forms (JSON + renderers) | Cloud | Same form in web, mobile and SPA | M10.2 |
| 113 | Forms on EDS, authored in Universal Editor (custom submit actions in 2026) | EDS + Cloud | Fast public forms | M10.1, M10.5 |
| 114 | Communications APIs, Forms Service APIs, document conversion APIs (2026) | Cloud | Document generation as APIs | M10.1, M10.5 |
| 115 | New Interactive Communication Editor with FDM data binding (2026) | Cloud | Personalized correspondence authoring | M10.1 |
| 116 | Automated Forms Conversion Service | Cloud | PDF to adaptive form | M10.1 |
| 117 | Captcha choices: reCAPTCHA Enterprise, hCaptcha, Turnstile | Cloud | Bot protection options | M10.1 |
| 118 | New submit targets: Power Automate, SharePoint, OneDrive, Azure Blob, Workfront Fusion, AEP | Cloud | Integrations without custom code | M10.5 |
| 119 | Forms Manager versioning (2026) | Cloud | Version history for forms, fragments, themes | M10.1 |
| 120 | AI form authoring by chat; Agentic Form Filling (early access 2026) | Cloud | Faster creation and conversational completion | M10.1, M13.4 |
| 121 | Forms JEE on 6.5 LTS with Java 17/21 and JBoss EAP 8 | LTS | JEE stays on-prem/AMS only | M10, M12.5 |

### J. Integrations and extensibility

| # | Change | Applies to | What it means | See |
| --- | --- | --- | --- | --- |
| 122 | App Builder (serverless on Adobe I/O Runtime) | All (mainly Cloud) | Integration logic outside AEM | M11.1, M11.7 |
| 123 | Adobe I/O Events for AEM Sites and Assets | Cloud | Event-driven integrations | M11.7 |
| 124 | UI Extensibility for CF console/editor, Universal Editor, Assets view | Cloud | Extend the UI without `/libs` overlays | M11.7 |
| 125 | API Mesh (GraphQL federation) | All | One API for AEM + Commerce + others | M11.7 |
| 126 | Adobe Web SDK replaces at.js/AppMeasurement for new builds | All | One data collection library | M11.2, M11.7 |
| 127 | AEP, Real-Time CDP and Journey Optimizer use AEM content | All | Personalization and journeys with AEM content | M11.2 |
| 128 | CIF on Cloud Service; Commerce storefront on EDS (drop-ins) | Cloud, EDS | Two commerce patterns | M11.3 |
| 129 | GenStudio for Performance Marketing on top of AEM Assets | Cloud | AI campaign variants from approved assets | M11.2 |
| 130 | Product line-up shifts: Screens and Guides on cloud; Communities not on cloud | Cloud | Know what's available where | M11.5 |

### K. Migration tooling

| # | Change | Applies to | What it means | See |
| --- | --- | --- | --- | --- |
| 131 | Best Practices Analyzer (BPA) | 6.x → Cloud | Finds cloud incompatibilities | M12.2 |
| 132 | Cloud Acceleration Manager (CAM) | 6.x → Cloud | Guides the whole migration | M12.2 |
| 133 | Content Transfer Tool with top-ups and pre-copy | 6.x → Cloud | Repeatable content migration | M12.7 |
| 134 | Repository Modernizer, Index Converter, Dispatcher Converter (aio plugin) | 6.x → Cloud | Automated code restructuring | M12.2 |
| 135 | AEM Modernize Tools (templates, components, policies) | All | Move from static/foundation to editable/Core | M12.2 |

### L. Operations and AI

| # | Change | Applies to | What it means | See |
| --- | --- | --- | --- | --- |
| 136 | Log forwarding (Splunk, Datadog, Elastic, Azure Blob, HTTPS) via config | Cloud | Centralized observability | M13.2 |
| 137 | CDN logs available and forwardable | Cloud | Edge-level troubleshooting and security analysis | M13.6 |
| 138 | Maintenance tasks configured as code | Cloud | Version/audit purge policies in Git | M13.3 |
| 139 | Proactive notifications and alerts (Cloud Manager, config-based alert rules) | Cloud | Faster incident response | M13.2 |
| 140 | AEM Sites Optimizer (agentic: identify, suggest, auto-fix with PRs) | Cloud, EDS | Continuous SEO/CWV/accessibility fixes | M13.4 |
| 141 | Adobe LLM Optimizer and Optimize at Edge (GEO) | Cloud, EDS | Visibility in AI answers | M13.4 |
| 142 | Generate Variations (copy and image variants) | Cloud, EDS | Faster experimentation content | M13.4 |
| 143 | AI Assistant and agents across AEM | Cloud | Natural-language help and actions for users | M13.4 |

## Module 1: The big picture (2020 → 2026)

AEM now ships in three forms: AEM as a Cloud Service (the strategic default), AEM 6.5 LTS (on-prem or AMS), and Edge Delivery Services (a delivery and authoring layer that comes with AEM Sites on Cloud Service). Every architecture answer in an interview should start by naming which of these you are designing for and why.

### 1.1 Timeline of what changed (newest first, approximate)

| When | Change | Why you care |
| --- | --- | --- |
| 2025-2026 | Agentic AI in AEM (AI Assistant, Sites Optimizer, content and optimization agents); LLM Optimizer for AI-search visibility | Clients ask about "AI-ready" content and GEO (generative engine optimization). Also announced: AEMaaCS runtime moves to Java 25 (planned May 2027) |
| 2025 | AEM 6.5 LTS GA on 7 Mar 2025 (Java 17/21, refreshed platform libraries); LTS SP1 Sep 2025, SP3 Aug 2026. Classic AEM 6.5: core support to 28 Feb 2027, extended to 28 Feb 2028 | On-prem/AMS clients now upgrade 6.5 → 6.5 LTS instead of waiting for "AEM 7" |
| 2025 | SPA Editor deprecated for new projects (AEMaaCS release 2025.1.0); Universal Editor is the recommended visual editor for headless and EDS | Changes your headless architecture recommendation |
| 2024-2025 | OpenAPI-based AEM APIs (Sites Content Fragment APIs, Assets Author API, Dynamic Media with OpenAPI); Content Hub; Document Authoring (da.live) | New integration surface, replaces many custom servlets |
| 2024 | Universal Editor authoring for Edge Delivery Services ("xwalk"), Forms on EDS, Commerce storefront on EDS | EDS becomes usable for enterprise authors, not only Google Docs users |
| 2023 | Edge Delivery Services GA (formerly Project Franklin / Helix); Universal Editor GA; new Content Fragment editor | The new default for high-performance marketing sites |
| 2022-2023 | Config pipeline, CDN config as code (`cdn.yaml`), traffic filter rules and WAF, Rapid Development Environments (RDE), Preview tier | Ops and security move into Git |
| 2021-2022 | AEM Forms as a Cloud Service, Adaptive Forms Core Components, Screens and Commerce (CIF) on Cloud Service; front-end pipeline | Forms and commerce clients can move to the cloud |
| 2020 | AEM as a Cloud Service GA (Sites + Assets), Cloud Manager mandatory, asset microservices | Your starting point |
| 2019 | AEM 6.5 released | Last "numbered" on-prem release before LTS |

### 1.2 The three deployment models today

| Dimension | On-Premise (6.5 / 6.5 LTS) | Adobe Managed Services (AMS) | AEM as a Cloud Service |
| --- | --- | --- | --- |
| Who runs infra | Client | Adobe (dedicated VMs on AWS/Azure) | Adobe (shared Kubernetes, containers) |
| Version | Fixed; you apply service packs | Fixed; AMS applies SPs on request | Always current; daily/monthly releases pushed by Adobe |
| Scaling | Manual (add publishers) | Manual (AMS ticket) | Automatic for publish |
| CDN | Bring your own | AMS CDN or BYO | Adobe-managed Fastly CDN included; BYO CDN in front allowed |
| Deploy | Any tool | Cloud Manager for AMS (classic) | Cloud Manager pipelines only |
| Customization | Anything, including overlays of `/libs` | Anything | Restricted: no `/libs` changes, immutable code, APIs only |
| Best fit | Regulated / air-gapped, heavy customization, AEM Forms JEE | Clients who want 6.5 but no ops team | Most new projects and migrations |

### 1.3 Architect view

- **Default recommendation in 2026**: AEMaaCS for authoring and repository; EDS for the public site where performance and speed of build matter; classic AEM Sites (Dispatcher + HTL) where you need deep in-context authoring of complex component-heavy pages or reuse an existing codebase.
- **When 6.5 LTS still wins**: data residency not covered by Adobe regions, air-gapped networks, AEM Forms JEE (Process Management/Workbench), heavy `/libs` overlays the client won't refactor, or a contract/timing constraint.
- **Key mindset shift**: in the cloud you design for *statelessness*, *immutability* and *APIs*, not for servers. No sticky sessions, no writing files to disk, no long-running jobs on one pod, no reliance on a specific publish instance.

### 1.4 Interview angle

**Q: "What is the difference between AMS and AEM as a Cloud Service?"** AMS is AEM 6.5 (or 6.5 LTS) hosted and operated by Adobe on dedicated VMs; it is still a versioned product you upgrade. AEMaaCS is a cloud-native, continuously updated SaaS-like service: containerized, auto-scaling, immutable code via Cloud Manager, Adobe-managed CDN, and no upgrade projects ever again.

**Q: "Is AEM 6.5 dead?"** No. Adobe released AEM 6.5 LTS in 2025 for clients who must stay on-prem or AMS. But new features (EDS, Universal Editor, OpenAPI APIs, GenAI) land first, or only, on Cloud Service.

### 1.5 Deep dive: how to frame platform decisions like an architect

**Weighted decision matrix** (use it live in an interview; scores 1-5, weights agreed with the client):

| Criterion | Weight | AEMaaCS classic | AEMaaCS + EDS | Headless (AEM + Next.js) | 6.5 LTS (AMS) |
| --- | --- | --- | --- | --- | --- |
| Time to market | 20% | 3 | 5 | 3 | 2 |
| Core Web Vitals / performance | 15% | 3 | 5 | 4 | 3 |
| Author experience (WYSIWYG, governance) | 20% | 5 | 4 | 3 | 5 |
| Total cost of ownership (ops, upgrades) | 15% | 4 | 5 | 3 | 2 |
| Reuse of existing code/skills | 15% | 4 | 2 | 2 | 5 |
| Flexibility for app-like experiences | 15% | 2 | 3 | 5 | 3 |

The numbers are illustrative; the point is to show you make the trade-off explicit and tie it to business weights.

**What Cloud Service changes in the commercial and operating model** (often asked by client leadership):

- Licensing is metered (content requests / page views, storage, authors, add-ons such as Forms, Screens, Dynamic Media, WAF-DDoS Protection, additional publish regions, Enhanced Security). Caching and bot filtering directly reduce counted requests.
- No upgrade projects: budget moves from "upgrade every 3 years" to "continuous small regression testing". You need automated tests because Adobe updates underneath you.
- Roles change: no AEM sysadmins; you need a **Cloud Manager Deployment Manager**, a **Business Owner** (approves prod deploys), DevOps for CDN/config pipeline, and an IMS/Admin Console admin.
- Environment set per program: 1 production + stage pair, several development environments, RDEs, optional specialized test environments; extra publish regions for latency and DR.

**6.5 vs 6.5 LTS: what an LTS upgrade really involves**

- Platform: Java 17/21 instead of 8/11; refreshed Oak, Sling, Felix and third-party libraries; deprecated APIs removed.
- Your code: recompile for Java 17/21, remove use of removed libraries (e.g. Guava, older Commons), update bundle imports, retest custom Oak queries and indexes, update build plugins.
- Forms JEE: new JBoss EAP 8 stack; plan it as a separate upgrade track.
- Approach: clone prod → upgrade on a copy → run regression + performance tests → rehearse the cutover → production upgrade in a maintenance window.

**Expert-level interview answer: "Should this client go to Cloud Service now?"** Answer with conditions, not a yes: *yes if* the customization inventory (BPA) is mostly cloud-compatible, integrations can be reached via egress IP/VPN, data residency fits Adobe regions, and the client accepts continuous updates; *not yet if* they depend on JEE Process Management, unsupported `/libs` overlays they won't refactor, or regulatory hosting constraints. Then propose an interim step (6.5 LTS) with a cloud-readiness backlog.

### References

- [AEM as a Cloud Service introduction](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/overview/introduction)
- [AEMaaCS architecture](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/overview/architecture)
- [AEMaaCS current release notes](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/release-notes/release-notes/release-notes-current)
- [Deprecated and removed features](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/release-notes/deprecated-removed-features)
- [AEM 6.5 LTS documentation](https://experienceleague.adobe.com/en/docs/experience-manager-65-lts)

## Module 2: AEM as a Cloud Service architecture

AEMaaCS runs AEM as containers on Adobe's Kubernetes clusters: one author cluster, an auto-scaling publish farm, an optional preview tier, a shared cloud data store for binaries, and a Fastly CDN in front. You never touch a server; you ship code through Cloud Manager and configure the rest as code.

### 2.1 Environments and tiers

- **Program** = a client's container in Cloud Manager. **Environments** inside it: Production + Stage (always a pair), Development (one or more), Rapid Development Environments (RDE), and Sandbox programs for trials.
- **Tiers per environment**: Author, Publish, and **Preview** (a publish-like tier where authors check content before go-live, via "Publish to Preview").
- **Author**: a cluster of author pods sharing one repository (document node store on MongoDB-like storage). Scales vertically/horizontally by Adobe, no clustering config for you.
- **Publish**: many identical publish pods, each with its own repository (segment node store) built from a "golden master". Content is distributed to all of them through Adobe's pipeline service (Sling Content Distribution). Pods come and go any time.
- **Binaries**: stored once in cloud blob storage (shared data store). Assets upload and download go **directly to blob storage** (direct binary access), not through the JVM.
- **Dispatcher**: still Apache + Dispatcher module, but runs in the same pod as each publish instance and is configured via your Git repo.
- **CDN**: Adobe-managed Fastly. You can place your own CDN (Akamai, Cloudflare) in front, using the `X-Forwarded-Host` and `X-AEM-Edge-Key` headers.

### 2.2 Immutable vs mutable repository

| Path | Type | How it changes |
| --- | --- | --- |
| `/apps`, `/libs` | Immutable (part of the image) | Only via Cloud Manager deployment. No CRXDE edits, no package install at runtime |
| `/content`, `/conf`, `/var`, `/etc`, `/home`, `/oak:index` | Mutable | Authors, workflows, packages in `ui.content`, repoinit |

Consequence: a code package (`ui.apps`) must contain **only** `/apps`; content and config go into `ui.content` / `ui.config`. Mixed packages fail the build. This is why the Maven archetype has separate modules (Module 4).

### 2.3 What is gone or different vs 6.5

| 6.5 habit | AEMaaCS reality |
| --- | --- |
| Replication agents, reverse replication | Sling Content Distribution, configured by Adobe; no reverse replication (use an external store or API) |
| CRXDE Lite / Package Manager on prod | Read-only CRXDE in Developer Console; packages only through pipeline (content packages still via Package Manager on author) |
| Custom run modes (`publish.uk`) | Only `author`, `publish`, `dev`, `stage`, `prod` (and combinations) |
| Overlays of `/libs` (e.g. modify a core UI) | Not allowed; use extension points, UI Extensibility (App Builder) |
| DAM Update Asset workflow for renditions | Asset microservices (cloud processing) + post-processing workflows |
| Classic UI | Removed |
| File system writes (e.g. `/tmp` caches) | Ephemeral; any pod can restart; use repository, blob, or external services |
| Long-running jobs, schedulers on every publish | Use Sling Jobs, run on author / leader; avoid heavy processing on publish |
| Upgrades and SPs | Automatic; your build must pass Adobe's quality gates to stay deployable |
| IP-allowlisting outbound | Advanced Networking (flexible egress, dedicated egress IP, VPN) |

### 2.4 Release model you must explain

- **Maintenance releases**: Adobe pushes small updates (often daily) with zero downtime using rolling (blue/green) deployment.
- **Feature releases**: monthly, announced in release notes; features first appear in the **prerelease channel** for testing.
- Your pipeline must keep passing against the latest AEM SDK. If a customer build breaks, Adobe can't update that environment, so keep dependencies current.
- Rolling deploys mean **old and new code run side by side** for a while: make index changes and content structure changes backward compatible.

### 2.5 Interview angle

**Q: "Why can't I install a code package via Package Manager on AEMaaCS?"** Because `/apps` and `/libs` are immutable and baked into the container image at deploy time. That gives identical, reproducible pods that can scale and be replaced at any time. Code only arrives through a Cloud Manager pipeline.

**Q: "How does content get to the publish tier?"** Author publishes → Sling Content Distribution sends it to Adobe's pipeline service → every publish pod pulls and applies it. New pods start from a golden master and catch up. You do not manage agents or queues.

**Q: "How do you handle user-generated data (e.g. form posts, comments) on publish?"** Not by writing to the publish repository (no reverse replication, and pods are disposable). Use an external store (database, CRM, Adobe Experience Platform), AEM Forms submit actions, or an App Builder action.

### 2.6 Deep dive: how AEMaaCS works internally

**How immutability is implemented: the composite node store.** Each AEM pod mounts its repository as a *composite* of two node stores: a read-only store containing `/apps` and `/libs` (built into the container image at deploy time) and the read-write store for everything else. A write to `/apps` therefore fails at the repository level, not just by policy. This is also why code and content packages must be separated: the code part is applied at image build time, the content part is installed on a running instance.

**Author vs publish storage**

| Tier | Node store | Binaries | Scaling |
| --- | --- | --- | --- |
| Author | Document node store (shared database), all author pods share it | Shared cloud blob store | Several pods behind a load balancer; Adobe scales |
| Publish | Segment node store (local, per pod) | Same shared cloud blob store (no copy of binaries per pod) | Horizontal auto-scaling; pods created from the golden master |

**Content distribution step by step**

1. Author activates (Manage Publication, Quick Publish, workflow, or `Replicator` API).
2. Sling Content Distribution builds a content package (metadata only; binaries are already in the shared blob store) and sends it to Adobe's pipeline service (a managed message queue).
3. Every publish pod (and the golden master) consumes the queue and imports the package in order.
4. Dispatcher cache for affected paths is invalidated on each pod.
5. A new publish pod is cloned from the golden master and replays queued items it hasn't seen, so it joins fully up to date.

Architecture consequences: publishing very large trees produces large queues (use tree activation via workflow/job, off-peak); the Distribution console in the Developer Console shows queue state; publishing is eventually consistent across pods (milliseconds to seconds).

**Deployments in detail (rolling / blue-green)**

1. Cloud Manager builds your code and merges it with the current AEM release into a new image.
2. If index definitions changed, new indexes are built in the background on a clone first.
3. Author: new pods start with the new image; repoinit runs; mutable content packages install; traffic switches; old pods stop.
4. Publish: a new golden master is prepared, new publish pods start from it, traffic moves gradually, old pods drain.
5. For a short period **old and new code run side by side** against the same author repository: content structure changes must be backward compatible (add new properties, migrate later; don't rename in place).

**Repoinit and mutable content timing.** Repoinit runs on every startup and is idempotent; mutable content packages (`ui.content`) install on author and publish during deployment. Use `filter.xml` modes carefully: `mode="merge"` or `"update"` for structure you don't want to overwrite on each deploy, because authors may have changed it.

**Statelessness rules (the checklist interviewers like)**

- No in-memory session state relied on across requests (no sticky sessions guarantee).
- No local file writes expected to persist; `/tmp` is per pod and ephemeral.
- Caches: in-memory caches per pod are fine if they tolerate inconsistency; otherwise external cache (e.g. Redis via App Builder/managed service) or the repository.
- Background jobs: Sling Jobs (cluster-aware), and keep them idempotent because pods can be killed during a deployment.
- Health: long-running requests (> \~60 s) get cut; make heavy operations asynchronous.

**Expert-level interview answer: "What happens when Adobe pushes an update during peak traffic?"** Updates are rolling: new pods with the new image join and pass health checks before old pods drain. Publish traffic is served by the CDN cache and remaining pods throughout. Risk lies in *your* code: backward-incompatible content changes and non-idempotent jobs. Mitigate with custom functional/UI tests in the pipeline (Adobe runs them before promoting) and with the prerelease channel to test upcoming features early.

### References

- [AEMaaCS architecture](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/overview/architecture)
- [Development guidelines for AEMaaCS](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/implementing/developing/development-guidelines)
- [Deploying to AEMaaCS (immutable/mutable)](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/implementing/deploying/overview)
- [Advanced networking](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/security/advanced-networking)

## Module 3: Cloud Manager, CI/CD, RDE and the local SDK

Cloud Manager is the only way code and configuration reach an AEMaaCS environment. It builds your Git repo, runs Adobe's quality gates, and deploys with zero downtime. Since 2020 it gained split pipelines (front-end, web tier, config), bring-your-own Git, environment variables and secrets, and Rapid Development Environments.

### 3.1 Pipeline types

| Pipeline | Deploys | When to use |
| --- | --- | --- |
| **Full-stack** (production or non-production) | Java bundles, `ui.apps`, `ui.content`, `ui.config`, dispatcher config, front-end clientlibs | Default; the production pipeline goes Build → Stage → tests → approval → Prod |
| **Front-end** | Only the static theme (CSS/JS) built from a `/ui.frontend`-style folder or Site Theme | Front-end team ships CSS/JS in minutes without a full build; needs Site Templates or the front-end module pattern |
| **Web tier config** | Only the `dispatcher/` folder | Change rewrite rules or cache config without redeploying AEM |
| **Config** | YAML files in a `config/` folder: CDN rules, traffic filters/WAF, log forwarding, maintenance tasks, alerts | Ops-as-code; deploys in minutes |
| **Code quality only** | Nothing | Run Sonar-style scans on a feature branch |

### 3.2 Quality gates (be ready to list them)

1. **Code quality**: SonarQube with Adobe custom rules (e.g. no `ResourceResolver` leaks, no `/libs` overlays, no `Thread.sleep`, OakPAL checks on package structure and index definitions). Ratings for security, reliability, maintainability, coverage (default 50% unit coverage threshold).
2. **Security tests** on stage: health checks Adobe runs against the deployed environment.
3. **Product functional tests** (Adobe's own) and **custom functional tests** (`it.tests`, Java) and **custom UI tests** (`ui.tests`, Cypress/Selenium/Playwright in a Docker image).
4. **Experience audit**: Lighthouse scores for configured pages on stage.
5. **Important vs critical issues**: critical fail the pipeline; important ones can be overridden by a Deployment Manager.

### 3.3 Git, build JDK and secrets

- **Repos**: Adobe-hosted Git, or bring your own (GitHub, GitLab, Bitbucket, Azure DevOps) connected to Cloud Manager. Typical setup at Infosys-scale clients: client GitHub/Azure DevOps as source of truth, Cloud Manager pipelines triggered from it.
- **Java version**: chosen with `.cloudmanager/java-version` in the repo root. Build with 21 (preferred), 17 or 11; Java 8 is not supported. Java 17/21 builds run on the Java 21 runtime. Adobe has announced a move to Java 25 (planned 2027), so keep plugins current.
- **Environment variables and secrets**: defined per environment in Cloud Manager, referenced in OSGi config. No more hard-coded API keys in run-mode folders.

```text
# .cloudmanager/java-version
21
```

```json
// ui.config/.../osgiconfig/config/com.mysite.core.services.impl.WeatherServiceImpl.cfg.json
{
  "apiUrl": "$[env:WEATHER_API_URL;default=https://api.example.com]",
  "apiKey": "$[secret:WEATHER_API_KEY]"
}
```

### 3.4 Rapid Development Environments (RDE)

- An RDE is a cloud environment where you push bundles, packages, OSGi configs and dispatcher config **in seconds** with the Adobe I/O CLI, skipping the full pipeline.
- Use it to test cloud-only behaviour (CDN, IMS login, asset microservices) that the local SDK can't simulate.
- Not for production content or load testing; resets are expected.

```bash
aio plugins:install @adobe/aio-cli-plugin-aem-rde
aio aem:rde:install core/target/mysite.core-1.0.0-SNAPSHOT.jar
aio aem:rde:install all/target/mysite.all-1.0.0-SNAPSHOT.zip
aio aem:rde:status
```

### 3.5 Local development with the AEM SDK

- **AEM SDK** (download from Software Distribution): QuickStart jar (author/publish), Dispatcher tools (Docker-based), Java API jar (`aem-sdk-api`). Update it monthly to match Cloud Service.
- Run author locally: `java -jar aem-author-p4502.jar`. Use JDK 21 locally.
- **Dispatcher tools**: `validate.sh` checks your config exactly like the pipeline; `docker_run.sh` runs Apache + Dispatcher locally against your local publish.
- Maven dependency is a single BOM-like artifact instead of the 2020 uber-jar:

```xml
<dependency>
  <groupId>com.adobe.aem</groupId>
  <artifactId>aem-sdk-api</artifactId>
  <version>${aem.sdk.api}</version>
  <scope>provided</scope>
</dependency>
```

- The **AEM Analyser Maven plugin** (`aemanalyser-maven-plugin`) runs during your local build to catch the same issues the cloud build catches (bundle imports, repoinit errors, content-package type mistakes).

### 3.6 Developer Console and Cloud Manager tools

- **Developer Console** (per environment): view OSGi bundles, components, configs, Sling status, run queries, check CRX in read-only mode, and test content distribution.
- **Logs**: download from Cloud Manager or tail with `aio cloudmanager:tail-log`; forward to Splunk/Datadog/Elastic with the config pipeline (Module 13).
- **Content Copy tool**: copy content from prod to stage/dev from Cloud Manager.
- **Self-service restore**: restore content to a point in time from Cloud Manager.

### 3.7 Interview angle

**Q: "Your front-end team waits 60+ minutes for every CSS change. What do you do?"** Introduce the front-end pipeline (or move the site to EDS). Separate the theme into its own folder, let Cloud Manager deploy only static assets, and keep the full-stack pipeline for Java and HTL.

**Q: "How do you keep secrets out of the repo?"** Cloud Manager secret environment variables referenced in `.cfg.json` with `$[secret:NAME]`, different values per environment, rotated without code changes.

**Q: "Pipeline fails on a code-quality 'critical' issue the business says to ignore."** Criticals can't be overridden by design; fix it. Only 'important' issues can be approved by the Deployment Manager, and you document the decision.

### 3.8 Deep dive (handbook): setting up Cloud Manager end to end

**Step 1: Roles in Admin Console (IMS)**

| Role (product profile) | Can do |
| --- | --- |
| Business Owner | Create programs, approve production deployments |
| Deployment Manager | Create/run pipelines, manage environments, override "important" quality issues |
| Program Manager | View, schedule, approve; no pipeline config |
| Developer | Access Git credentials, Developer Console, logs |
| AEM Administrators / Users (per environment) | Log in to that environment's author with admin or user rights |

Create the profiles per environment in Admin Console, assign the client's IdP groups, and document who approves prod. This is the first thing to set up because it blocks everyone.

**Step 2: Program and environments**

1. Create the program (Sites, Assets, Forms solutions; region; add-ons such as additional publish region, Enhanced Security).
2. Add Production + Stage, then Development environments (e.g. `dev`, `qa`) and RDEs.
3. Configure IP allowlists for author/preview (client VPN ranges), custom domains, SSL certificates.
4. Request Advanced Networking (choose ONE type per program: flexible egress, dedicated egress IP, or VPN) before integration testing starts.

**Step 3: Repository strategy**

- Option A: Adobe-hosted Git as the pipeline source; client Git mirrored into it (`git push adobe main`) from the client's CI (GitHub Actions/Azure DevOps).
- Option B (preferred now): connect the client's GitHub/GitLab/Bitbucket/Azure DevOps directly as a private repository in Cloud Manager; pipelines can then run on PRs (code-quality checks on pull requests).
- Branching: `develop` → dev pipeline (on commit), `release/*` → stage/prod pipeline (manual), feature branches → code-quality pipeline or RDE.

**Step 4: The `.cloudmanager` folder (complete example)**

```text
.cloudmanager/
  java-version            # 21
  maven/settings.xml      # private Maven repos (e.g. Nexus/Artifactory)
```

```xml
<!-- .cloudmanager/maven/settings.xml -->
<settings>
  <servers>
    <server>
      <id>myco-nexus</id>
      <username>${env.CUSTOM_NEXUS_USER}</username>
      <password>${env.CUSTOM_NEXUS_PASSWORD}</password>
    </server>
  </servers>
  <profiles>
    <profile>
      <id>myco-nexus</id>
      <activation><property><name>env.CM_BUILD</name></property></activation>
      <repositories>
        <repository>
          <id>myco-nexus</id>
          <url>https://nexus.myco.com/repository/maven-public/</url>
        </repository>
      </repositories>
    </profile>
  </profiles>
</settings>
```

`CUSTOM_NEXUS_PASSWORD` is a **pipeline variable** marked secret (build-time), set via Cloud Manager UI or CLI:

```bash
aio cloudmanager:set-pipeline-variables <pipelineId> \
  --variable CUSTOM_NEXUS_USER builduser \
  --secret CUSTOM_NEXUS_PASSWORD '***'
```

The nexus host must be reachable from Cloud Manager's build network (public, or allowlisting Adobe build egress per docs).

**Step 5: Environment variables vs pipeline variables**

| Type | Scope | Used in | Example |
| --- | --- | --- | --- |
| Pipeline variable (build) | One pipeline, build time | Maven/npm build | Private repo credentials |
| Environment variable | One environment, runtime | OSGi `$[env:...]`, `$[secret:...]`; config pipeline `${{...}}` | API URLs, keys, log levels |

Values are applied without a code deploy (the environment restarts services). Limit: names in `UPPER_SNAKE_CASE`; secrets never shown after save.

**Step 6: Production pipeline phases (what you'll see in the UI)**

1. Validation (pipeline config, environment state).
2. Build & unit tests (Maven `clean install` in Adobe's build container; `CM_BUILD=true` env var is set).
3. Code scanning (Sonar + Adobe rules + OakPAL); artifact preparation (image build).
4. Deploy to Stage.
5. Stage testing: product functional tests, custom functional tests (`it.tests`), custom UI tests (`ui.tests`), experience audit (Lighthouse on configured paths).
6. Approval gate (Business Owner / Deployment Manager) — can be skipped/scheduled.
7. Deploy to Production (rolling).

A prod pipeline always deploys the same artifact that was tested on stage. Emergency fix path: "execution mode" with emergency/skip-tests options exists but is audited; use only per agreed runbook.

**Step 7: Custom UI tests contract (`ui.tests`)**

- Module builds a Docker image (via `ui.tests` Maven profile) containing the test runner (Cypress, Selenium/WebdriverIO, Playwright).
- Cloud Manager injects env vars such as `AEM_AUTHOR_URL`, `AEM_AUTHOR_USERNAME`, `AEM_AUTHOR_PASSWORD`, `AEM_PUBLISH_URL`, `REPORTS_PATH`; results must be JUnit XML in `REPORTS_PATH`.
- Opt in by adding a `testing.properties` file with `ui-tests.version=1` to the build output.
- Keep tests under \~30 minutes; they gate every production deploy.

**Step 8: Code quality rules you will hit most often**

| Rule (example) | Why | Fix |
| --- | --- | --- |
| ResourceResolver not closed | Leaks sessions | try-with-resources |
| Use of `Thread.sleep` / `System.out` | Blocks, no logs | Async/jobs, SLF4J |
| HTTP request without timeout | Thread exhaustion | Set connect/read timeouts |
| Use of deprecated/internal APIs | Breaks on updates | Use public APIs |
| Mixed package / `/libs` content | Immutable violation | Split packages |
| Custom index not Lucene / wrong name | Deploy failure | `-custom-n` Lucene index |
| Overlapping OSGi configs | Ambiguity | One config per PID per run mode |
| Coverage < threshold (50%) | Quality gate | Unit tests with AEM Mocks |

**Step 9: Day-2 Cloud Manager CLI cheat sheet**

```bash
npm i -g @adobe/aio-cli
aio plugins:install @adobe/aio-cli-plugin-cloudmanager
aio login
aio cloudmanager:list-programs
aio cloudmanager:program:list-environments --programId 12345
aio cloudmanager:pipeline:start-execution <pipelineId>
aio cloudmanager:environment:tail-log <envId> author aemerror
aio cloudmanager:environment:download-logs <envId> publish httpdaccess 1
aio cloudmanager:environment:set-variables <envId> --variable LOG_LEVEL DEBUG
```

**Expert-level interview answer: "Design CI/CD for 6 scrum teams on one AEM program."** Trunk-based or short-lived branches in the client's GitHub connected to Cloud Manager; PR checks with the code-quality pipeline + local `aemanalyser`; each team gets an RDE for fast loops; shared `dev` pipeline on merge; nightly stage deploy with full UI tests; weekly prod release train with Business Owner approval; front-end pipeline for theme changes and web-tier pipeline for dispatcher; config pipeline owned by the platform team. Feature toggles via environment variables/CAC to decouple deploy from release.

### References

- [CI/CD pipelines in Cloud Manager](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/implementing/using-cloud-manager/cicd-pipelines/introduction-ci-cd-pipelines)
- [Cloud Manager build environment (Java versions)](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/implementing/using-cloud-manager/create-application-project/build-environment-details)
- [Front-end pipeline](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/implementing/using-cloud-manager/front-end-pipeline)
- [Config pipeline](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/implementing/using-cloud-manager/config-pipeline)
- [Environment variables](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/implementing/using-cloud-manager/environment-variables)
- [Rapid Development Environments](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/implementing/developing/rapid-development-environments)
- [AEM as a Cloud Service SDK](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/implementing/developing/aem-as-a-cloud-service-sdk)

## Module 4: Backend coding changes

The Java/OSGi/Sling/HTL core you know is still the same. What changed is *where things live* (strict package separation), *how the repository is initialized* (repoinit instead of packages), *how config is written* (`.cfg.json`), and *what you are allowed to do at runtime* (stateless, cluster-aware, no long-running work on publish).

### 4.1 The Maven archetype project structure

Generate projects with the latest AEM Project Archetype (check the GitHub releases page for the current version):

```bash
mvn -B org.apache.maven.plugins:maven-archetype-plugin:3.2.1:generate \
  -D archetypeGroupId=com.adobe.aem \
  -D archetypeArtifactId=aem-project-archetype \
  -D archetypeVersion=<latest> \
  -D appTitle="My Site" -D appId=mysite -D groupId=com.mysite \
  -D aemVersion=cloud -D frontendModule=general -D includeDispatcherConfig=y
```

| Module | Contains | Package type |
| --- | --- | --- |
| `core` | Java: Sling Models, services, servlets, schedulers | OSGi bundle |
| `ui.apps` | `/apps`: components, templates' code, clientlibs | `application` |
| `ui.apps.structure` | Declares repository roots so FileVault validation knows them | (build only) |
| `ui.config` | OSGi configs `/apps/mysite/osgiconfig/config.<runmode>` | `container` |
| `ui.content` | `/content`, `/conf` (templates, policies), sample pages | `content` |
| `ui.frontend` | Webpack/TypeScript/Sass source; output is generated into clientlibs | npm build |
| `ui.tests`, `it.tests` | UI tests (Cypress/Selenium) and HTTP integration tests | test images |
| `dispatcher` | Apache + Dispatcher config in the Cloud Service format | zip |
| `all` | Single deployable package embedding everything | `container` |

**Rule**: a package is either `application` (only `/apps`) or `content` (only mutable paths). Mixed = build failure. Custom Oak indexes go in `ui.apps` under `/oak:index` (allowed exception).

### 4.2 OSGi configuration: `.cfg.json` and run modes

```text
ui.config/src/main/content/jcr_root/apps/mysite/osgiconfig/
  config/                    # all environments, author + publish
  config.author/
  config.publish/
  config.publish.prod/
  config.author.dev/
```

```json
// org.apache.sling.commons.log.LogManager.factory.config~mysite.cfg.json
{
  "org.apache.sling.commons.log.names": ["com.mysite"],
  "org.apache.sling.commons.log.level": "$[env:LOG_LEVEL;default=INFO]",
  "org.apache.sling.commons.log.file": "logs/error.log",
  "org.apache.sling.commons.log.additiv": "false"
}
```

Old XML `sling:OsgiConfig` nodes and custom run modes (e.g. `publish.uk`) are not supported on Cloud Service. Use a site-specific value in the config itself or context-aware configuration (CAC) for per-site settings.

### 4.3 Repoinit: service users, ACLs, folders

In 2020 you shipped `/home/users/system/...` in a content package. Now everything structural is declared in a repoinit script, executed at startup on every pod:

```json
// ui.config/.../config/org.apache.sling.jcr.repoinit.RepositoryInitializer~mysite.cfg.json
{
  "scripts": [
    "create path (sling:OrderedFolder) /content/dam/mysite",
    "create service user mysite-content-reader with path system/mysite",
    "set ACL for mysite-content-reader\n  allow jcr:read on /content/mysite\n  allow jcr:read on /content/dam/mysite\nend",
    "create group mysite-authors\nset ACL for mysite-authors\n  allow jcr:all on /content/mysite\nend"
  ]
}
```

```json
// org.apache.sling.serviceusermapping.impl.ServiceUserMapperImpl.amended~mysite.cfg.json
{
  "user.mapping": ["com.mysite.core:content-reader=[mysite-content-reader]"]
}
```

```java
Map<String, Object> auth = Map.of(ResourceResolverFactory.SUBSERVICE, "content-reader");
try (ResourceResolver rr = resolverFactory.getServiceResourceResolver(auth)) {
    Resource page = rr.getResource("/content/mysite/en");
    // ...
} // auto-closed: no leaks (a Sonar critical rule)
```

### 4.4 Sling Models and exporters (still the core pattern)

```java
@Model(adaptables = SlingHttpServletRequest.class,
       adapters = {Teaser.class, ComponentExporter.class},
       resourceType = TeaserImpl.RESOURCE_TYPE,
       defaultInjectionStrategy = DefaultInjectionStrategy.OPTIONAL)
@Exporter(name = ExporterConstants.SLING_MODEL_EXPORTER_NAME,
          extensions = ExporterConstants.SLING_MODEL_EXTENSION)
public class TeaserImpl implements Teaser {
    static final String RESOURCE_TYPE = "mysite/components/teaser";

    @ValueMapValue private String title;
    @ValueMapValue private String linkURL;
    @OSGiService private LinkService linkService;

    @Override public String getTitle() { return title; }
    @Override public String getLink() { return linkService.externalize(linkURL); }
    @Override public String getExportedType() { return RESOURCE_TYPE; }
}
```

Best practice since 2020: **extend Core Components via delegation** (Sling Model `@Self @Via(type = ResourceSuperType.class)`), never copy them; and use the Core Components' JSON export for hybrid/headless pages.

```java
@Model(adaptables = SlingHttpServletRequest.class, adapters = Title.class,
       resourceType = "mysite/components/title")
public class CustomTitle implements Title {
    @Self @Via(type = ResourceSuperType.class)
    private Title delegate;          // core component's model
    @Override public String getText() { return delegate.getText().toUpperCase(); }
    @Override public String getType() { return delegate.getType(); }
}
```

### 4.5 Background work: Sling Jobs over plain schedulers

Publish pods are many and disposable; a plain `@Scheduled` Runnable runs on *every* pod. Use a scheduler only to enqueue, and a **Sling Job** to do the work (guaranteed once, retried, cluster-aware). Run heavy work on author.

```java
@Component(service = Runnable.class, property = {
    "scheduler.expression=0 0 2 * * ?",
    "scheduler.concurrent:Boolean=false",
    "scheduler.runOn=SINGLE"})
public class NightlySyncScheduler implements Runnable {
    @Reference private JobManager jobManager;
    public void run() { jobManager.addJob("mysite/sync/products", Map.of()); }
}

@Component(service = JobConsumer.class,
           property = JobConsumer.PROPERTY_TOPICS + "=mysite/sync/products")
public class ProductSyncJob implements JobConsumer {
    public JobResult process(Job job) {
        // call external API, write to /content or /var ...
        return JobResult.OK; // FAILED -> retried
    }
}
```

### 4.6 Oak indexes as code

- Index definitions live in `ui.apps` (or a dedicated index package) under `/oak:index`.
- **Naming convention**: customizing an OOTB index = `damAssetLucene-<productVersion>-custom-<n>` (e.g. `damAssetLucene-11-custom-1`); a fully custom index = `mysite.productIndex-1-custom-1`. Each change bumps the `custom-n` number; Cloud Manager builds the new index *before* switching traffic (blue/green).
- Only Lucene indexes (`type=lucene`, `compatVersion=2`, `async`). No property-type indexes for customers.
- Use the **Oak Index Definition Generator** and query explain tool in the Developer Console to design them.

```xml
<!-- /oak:index/mysite.articleIndex-1-custom-1/.content.xml (simplified) -->
<jcr:root jcr:primaryType="oak:QueryIndexDefinition" type="lucene" async="[async,nrt]"
    compatVersion="{Long}2" evaluatePathRestrictions="{Boolean}true"
    includedPaths="[/content/mysite]" queryPaths="[/content/mysite]">
  <indexRules jcr:primaryType="nt:unstructured">
    <cq:PageContent jcr:primaryType="nt:unstructured">
      <properties jcr:primaryType="nt:unstructured">
        <articleType jcr:primaryType="nt:unstructured" name="articleType" propertyIndex="{Boolean}true"/>
      </properties>
    </cq:PageContent>
  </indexRules>
</jcr:root>
```

### 4.7 Other coding changes worth mentioning

- **Java 17/21 features** are allowed: `record`, `switch` expressions, text blocks, `var`, pattern matching.
- **Replication API**: `Replicator` still works on Cloud Service (backed by Sling Content Distribution); you can also publish to the Preview tier with `ReplicationOptions`/agent id `preview`.
- **HTTP clients**: use Apache HttpClient 4 or Java's `HttpClient`, with outbound traffic routed through Advanced Networking proxies (`AEM_PROXY_HOST`) when the target needs an allowlisted IP.
- **Servlets**: register by resource type (not by path) to keep ACLs and Dispatcher caching sane.
- **Workflows**: still supported, but on author only, and keep them short; DAM processing is done by asset microservices plus *post-processing workflows*.
- **Removed/deprecated APIs**: Guava, Commons Collections 3, Apache Commons Lang 2 are not exported; Sonar and the AEM analyser flag them. Same cleanup applies when moving to 6.5 LTS.
- **Content-aware configuration (CAC)** is the standard for per-site settings (multi-tenant, multi-brand).

### 4.8 Interview angle

**Q: "Where do you create a service user in AEMaaCS?"** In a repoinit script in `ui.config`, plus a `ServiceUserMapperImpl.amended` config. Never in a content package, never manually.

**Q: "A scheduler sends the same email 6 times a night. Why?"** It runs on every publish pod (and pods change). Run it on author with `scheduler.runOn=SINGLE`, or enqueue a Sling Job that runs exactly once.

**Q: "How do you add a property to the OOTB asset search index?"** Create `damAssetLucene-<v>-custom-1` with the full definition plus your property, deploy via pipeline; Cloud Manager reindexes before switching.

### 4.9 Deep dive (handbook): complete backend patterns

**A. OSGi service with typed configuration (`@Designate`)**

```java
@ObjectClassDefinition(name = "My Site - Weather Service")
public @interface WeatherConfig {
    @AttributeDefinition(name = "API URL") String apiUrl() default "https://api.example.com";
    @AttributeDefinition(name = "API key") String apiKey();
    @AttributeDefinition(name = "Timeout ms") int timeout() default 5000;
}

@Component(service = WeatherService.class)
@Designate(ocd = WeatherConfig.class)
public class WeatherServiceImpl implements WeatherService {
    private static final Logger LOG = LoggerFactory.getLogger(WeatherServiceImpl.class);
    private volatile WeatherConfig config;
    private HttpClient client;

    @Activate @Modified
    protected void activate(WeatherConfig config) {
        this.config = config;
        this.client = HttpClient.newBuilder()
            .connectTimeout(Duration.ofMillis(config.timeout()))
            .build();                       // honours JVM proxy settings set by Advanced Networking
    }

    @Override
    public Optional<String> forecast(String city) {
        var req = HttpRequest.newBuilder(URI.create(config.apiUrl() + "/forecast?city="
                + URLEncoder.encode(city, StandardCharsets.UTF_8)))
            .timeout(Duration.ofMillis(config.timeout()))
            .header("x-api-key", config.apiKey())
            .GET().build();
        try {
            var res = client.send(req, HttpResponse.BodyHandlers.ofString());
            return res.statusCode() == 200 ? Optional.of(res.body()) : Optional.empty();
        } catch (IOException e) {
            LOG.warn("Weather API failed for {}", city, e);
            return Optional.empty();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            return Optional.empty();
        }
    }
}
```

The PID is the class name: config file `com.mysite.core.services.impl.WeatherServiceImpl.cfg.json` (values from env vars/secrets as in 3.3).

**Outbound networking notes (Advanced Networking):** for HTTP/HTTPS, AEM sets the standard JVM proxy properties, so clients that honour them route through the egress proxy automatically; Apache HttpClient needs `useSystemProperties()`. Proxy host/port are also exposed as env vars (e.g. `AEM_HTTP_PROXY_HOST`, `AEM_HTTP_PROXY_PORT`, `AEM_HTTPS_PROXY_HOST`, `AEM_HTTPS_PROXY_PORT`); non-HTTP protocols (SMTP, databases) use `AEM_PROXY_HOST` with port forwarding rules declared in the network configuration.

**B. Servlet bound to a resource type (preferred over path)**

```java
@Component(service = Servlet.class)
@SlingServletResourceTypes(
    resourceTypes = "mysite/components/page",
    selectors = "weather",
    extensions = "json",
    methods = HttpConstants.METHOD_GET)
public class WeatherServlet extends SlingSafeMethodsServlet {
    @Reference private transient WeatherService weather;

    @Override
    protected void doGet(SlingHttpServletRequest req, SlingHttpServletResponse res) throws IOException {
        String city = req.getResource().getValueMap().get("city", "London");
        res.setContentType("application/json");
        res.setHeader("Cache-Control", "max-age=300");      // let Dispatcher/CDN cache it
        res.getWriter().write(weather.forecast(city).orElse("{}"));
    }
}
```

Called as `/content/mysite/en/home.weather.json`: inherits page ACLs, is cacheable, and Dispatcher filters only need to allow the `weather` selector.

**C. Sling filter (e.g. adding security headers or logging)**

```java
@Component(service = Filter.class, property = {
    EngineConstants.SLING_FILTER_SCOPE + "=" + EngineConstants.FILTER_SCOPE_REQUEST,
    EngineConstants.SLING_FILTER_PATTERN + "=/content/mysite/.*",
    Constants.SERVICE_RANKING + ":Integer=-700"})
public class RequestTimingFilter implements Filter {
    private static final Logger LOG = LoggerFactory.getLogger(RequestTimingFilter.class);
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
            throws IOException, ServletException {
        long start = System.nanoTime();
        try { chain.doFilter(req, res); }
        finally {
            long ms = (System.nanoTime() - start) / 1_000_000;
            if (ms > 1000) LOG.warn("Slow request {} took {} ms",
                ((SlingHttpServletRequest) req).getRequestURI(), ms);
        }
    }
    public void init(FilterConfig c) {} public void destroy() {}
}
```

**D. Context-Aware Configuration (per-site settings)**

```java
@Configuration(label = "Site Settings")
public @interface SiteSettings {
    String gtmId() default "";
    String supportEmail() default "support@example.com";
    boolean enableChat() default false;
}

// In a Sling Model
@SlingObject private Resource resource;
public String getSupportEmail() {
    ConfigurationBuilder cb = resource.adaptTo(ConfigurationBuilder.class);
    return cb.as(SiteSettings.class).supportEmail();
}
```

The site root points to its config with `sling:configRef=/conf/mysite-uk`; values are stored in `/conf/mysite-uk/sling:configs/com.mysite.core.config.SiteSettings` and inherited from `/conf/mysite` (multi-brand pattern without custom run modes). Register the class with the `Sling-ContextAware-Configuration-Classes` bundle header (the archetype's bnd config does this when you list it).

**E. Reacting to content changes: `ResourceChangeListener`**

```java
@Component(service = ResourceChangeListener.class, property = {
    ResourceChangeListener.PATHS + "=glob:/content/mysite/**",
    ResourceChangeListener.CHANGES + "=ADDED",
    ResourceChangeListener.CHANGES + "=CHANGED"})
public class PageChangeListener implements ResourceChangeListener {
    @Reference private JobManager jobManager;
    public void onChange(List<ResourceChange> changes) {
        changes.stream().map(ResourceChange::getPath).distinct()
            .forEach(p -> jobManager.addJob("mysite/index/page", Map.of("path", p)));
    }
}
```

Keep listeners tiny: hand the work to a Sling Job. Remember it fires on author (and on publish when content arrives); register with a run-mode-specific config or check `SlingSettingsService` if needed.

**F. Custom workflow process step**

```java
@Component(service = WorkflowProcess.class,
           property = "process.label=My Site - Notify Legal")
public class NotifyLegalStep implements WorkflowProcess {
    @Reference private EmailService email;
    public void execute(WorkItem item, WorkflowSession session, MetaDataMap args) throws WorkflowException {
        String payload = item.getWorkflowData().getPayload().toString();
        String to = args.get("PROCESS_ARGS", "legal@example.com");
        email.send(to, "Review needed", "Please review " + payload);
    }
}
```

Workflow models live in `/conf/global/settings/workflow/models` (editable) and are deployed via `ui.content`; runtime models are in `/var/workflow/models`. Purge workflow instances regularly (maintenance task).

**G. Queries that don't kill the repository**

- Always have an index for the query (check with Developer Console → Queries → Explain). Traversal warnings in `error.log` = missing index.
- Prefer JCR-SQL2 with a path restriction and a node type that matches the index rule.
- Set limits; with QueryBuilder use `p.guessTotal=true` and `p.limit`.
- Oak aborts queries that read too many nodes (default limit \~100,000) on Cloud Service: design for it.

```java
String q = "SELECT * FROM [cq:PageContent] AS c "
         + "WHERE ISDESCENDANTNODE(c, '/content/mysite') AND c.[articleType] = 'news' "
         + "ORDER BY c.[jcr:created] DESC";
Iterator<Resource> it = resolver.findResources(q, Query.JCR_SQL2);
// read only the first N results
```

**H. Unit tests with AEM Mocks (JUnit 5)**

```java
@ExtendWith(AemContextExtension.class)
class TeaserImplTest {
    private final AemContext ctx = new AemContext(ResourceResolverType.JCR_MOCK);

    @BeforeEach
    void setUp() {
        ctx.registerService(LinkService.class, url -> "https://www.example.com" + url);
        ctx.addModelsForClasses(TeaserImpl.class);
        ctx.create().resource("/content/teaser",
            "sling:resourceType", "mysite/components/teaser",
            "title", "Hello", "linkURL", "/en/home");
        ctx.currentResource("/content/teaser");
    }

    @Test
    void returnsTitleAndExternalLink() {
        Teaser t = ctx.request().adaptTo(Teaser.class);
        assertEquals("Hello", t.getTitle());
        assertEquals("https://www.example.com/en/home", t.getLink());
    }
}
```

Dependency: `io.wcm:io.wcm.testing.aem-mock.junit5` (test scope). Coverage from these tests feeds the 50% gate.

**I. Logging properly**

- One logger config per package in `ui.config` (example in 4.2); `DEBUG` only on dev via env var.
- Never log PII or secrets; Cloud Service logs may be forwarded to many systems.
- Use parameterized logging (`LOG.info("x {}", y)`), no string concatenation.

**Expert-level interview answer: "Review this code: a Sling Model opens a service ResourceResolver in `@PostConstruct` and stores it in a field."** It leaks a session per request (never closed), runs with service-user rights for every visitor (privilege issue), and is not thread-safe if cached. Fix: use the request's resolver for reads the visitor is allowed to do; if elevated access is truly needed, open the service resolver in try-with-resources inside an OSGi service method and return plain data.

### References

- [AEM Project Archetype (GitHub)](https://github.com/adobe/aem-project-archetype)
- [Core WCM Components (GitHub)](https://github.com/adobe/aem-core-wcm-components)
- [Configuring OSGi for AEMaaCS](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/implementing/deploying/configuring-osgi)
- [Repoinit](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/implementing/deploying/sling-feature-model/repoinit)
- [Content search and indexing](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/operations/indexing)
- [WKND reference project (GitHub)](https://github.com/adobe/aem-guides-wknd)

## Module 5: Front-end changes

Front end in AEM now has three valid tracks: (1) classic HTL + Core Components + `ui.frontend` webpack build; (2) Site Templates + Themes deployed via the front-end pipeline; (3) Edge Delivery Services with plain JS/CSS blocks (Module 6). Headless SPAs (React/Next.js/Angular) consuming AEM APIs are the fourth, covered in Module 7.

### 5.1 Classic track: what changed since 2020

| Topic | 2020 | Now |
| --- | --- | --- |
| Components | Mix of foundation, custom and early Core Components | Core Components (v2+ models) for everything; proxy components in `/apps`; extend by delegation |
| Templates | Static templates still common | Editable templates + policies only (static templates are legacy; AEM Modernize Tools converts them) |
| Styling variants | Many component copies | Style System: authors pick CSS classes defined in policies; one component, many looks |
| Front-end build | Hand-written clientlibs | `ui.frontend` (webpack, TypeScript, Sass) → `aem-clientlib-generator` writes clientlibs into `ui.apps` |
| Images | Adaptive Image servlet | Core Image v3 with **web-optimized image delivery** (WebP/AVIF, resized by the CDN-side image service), lazy loading, `srcset` |
| Analytics data | Custom `digitalData` | **Adobe Client Data Layer** (ACDL) built into Core Components; feeds Adobe Tags (Launch) / Web SDK |
| Performance | Optional | Core Web Vitals (LCP, CLS, INP) measured in Cloud Manager's experience audit; Sites Optimizer suggests fixes |

### 5.2 Proxy component + Style System example

```xml
<!-- ui.apps/.../apps/mysite/components/teaser/.content.xml -->
<jcr:root xmlns:sling="http://sling.apache.org/jcr/sling/1.0"
    xmlns:cq="http://www.day.com/jcr/cq/1.0" xmlns:jcr="http://www.jcp.org/jcr/1.0"
    jcr:primaryType="cq:Component"
    jcr:title="Teaser"
    sling:resourceSuperType="core/wcm/components/teaser/v2/teaser"
    componentGroup="My Site - Content"/>
```

Style System classes are configured on the component **policy** (in `/conf/mysite/settings/wcm/policies`), e.g. `cmp-teaser--hero`, `cmp-teaser--dark`. The CSS lives in `ui.frontend` and targets those BEM classes. Authors pick the style in the page editor; no Java change.

```html
<!-- HTL still the same language; use Sling Models via data-sly-use -->
<sly data-sly-use.teaser="com.mysite.core.models.Teaser"
     data-sly-use.template="core/wcm/components/commons/v1/templates.html">
  <div class="cmp-teaser" data-sly-test="${teaser.title}">
    <h2 class="cmp-teaser__title">${teaser.title}</h2>
    <a class="cmp-teaser__link" href="${teaser.link}">${'Read more' @ i18n}</a>
  </div>
  <sly data-sly-call="${template.placeholder @ isEmpty=!teaser.title}"/>
</sly>
```

### 5.3 `ui.frontend` build

```bash
cd ui.frontend
npm ci
npm run dev      # webpack build + clientlib generation
npm run watch    # with aemsync: pushes changes to local AEM instantly
npm run start    # webpack dev server proxying a local AEM (static HTML mocks)
```

The Maven build runs `npm` through `frontend-maven-plugin`, so the Cloud Manager full-stack pipeline builds the front end too.

### 5.4 Site Templates, Themes and the front-end pipeline

- **Site Template**: a zip with sample pages, templates, policies and a theme; authors create a new site from it in the Sites console ("Create → Site from template"). Adobe's "Basic site template" is the starting point.
- **Theme**: a standalone npm project (CSS/JS). Front-end developers run it locally against a live AEM environment with a proxy (`npm run live`), then push to Git; the **front-end pipeline** builds it and deploys only the static files (to the CDN) in minutes.
- Good fit: marketing microsites, campaign sites, teams that want front-end independence without a full Java build.

### 5.5 Page Editor vs Universal Editor

- **Page Editor** (the Touch UI editor you know) is still the editor for classic HTL sites, with Template Editor, Style System, layout mode.
- **Universal Editor** is the new, framework-agnostic visual editor: it edits any HTML (HTL, React, Next.js, EDS) via `data-aue-*` attributes, and writes back to AEM. Recommended for all new headless and EDS work (details in Module 7).
- **SPA Editor** (React/Angular SDKs from 2019-2020) is deprecated for new projects since AEMaaCS 2025.1.0. Existing SPA Editor sites keep working with P1/P2 and security fixes only.

### 5.6 Interview angle

**Q: "Authors want 5 variations of a teaser. Five components?"** No: one proxy of the Core Teaser, five Style System classes in the policy, CSS in `ui.frontend`. Less code, consistent analytics and accessibility.

**Q: "Front-end team works in React and dislikes HTL. Options?"** (a) Headless React app with AEM GraphQL/OpenAPI + Universal Editor; (b) EDS with vanilla JS blocks; (c) keep HTL but give them a theme project + front-end pipeline. Choose based on authoring needs and performance targets.

### 5.7 Deep dive (handbook): building the classic front end properly

**A. Editable template anatomy** (in `/conf/mysite/settings/wcm/templates/<template>`)

| Node | Purpose |
| --- | --- |
| `structure` | Locked layout (header XF, footer XF, main container); authors can't remove it |
| `initial` | Content copied into every new page (unlocked components) |
| `policies` | Maps each component/container to a policy (allowed components, styles, settings) |
| `thumbnail.png`, `jcr:content` (status) | Template must be *enabled* to be used |
| Template type (`/conf/mysite/settings/wcm/template-types`) | Blueprint for templates, created by developers |

Policies live in `/conf/mysite/settings/wcm/policies/...` and are shared between templates. Allowed templates for a site are set via `cq:allowedTemplates` on the site root. Ship template types and initial templates via `ui.content`, let template authors (a power-user group) create the rest in Template Editor.

**B. Clientlibs: the full picture**

```text
ui.frontend/src/main/webpack/
  site/main.scss, main.ts
  components/teaser/_teaser.scss
ui.frontend/clientlib.config.js     # aem-clientlib-generator config
→ generates ui.apps/.../clientlibs/clientlib-site (css.txt, js.txt, resources/)
→ and clientlib-dependencies (vendor bundles)
```

```js
// ui.frontend/clientlib.config.js (shortened)
module.exports = {
  context: __dirname,
  clientLibRoot: '../ui.apps/src/main/content/jcr_root/apps/mysite/clientlibs',
  libs: [{
    name: 'clientlib-site',
    allowProxy: true,                 // serve via /etc.clientlibs (publish can't read /apps)
    categories: ['mysite.site'],
    dependencies: ['mysite.dependencies'],
    serializationFormat: 'xml',
    assets: { js: ['dist/clientlib-site/*.js'], css: ['dist/clientlib-site/*.css'],
              resources: { cwd: 'dist/clientlib-site', files: ['**/*.*'], flatten: false, ignore: ['**/*.js', '**/*.css'] } }
  }]
};
```

Loading: the Core **Page** component's page policy lists which clientlib categories load in `<head>` (CSS) and at the end of `<body>`, with *async/defer* options for JS. Versioned clientlib URLs (`.lc-<hash>-lc.min.js`, via the Versioned ClientLibs/"long cache" feature built into Cloud Service) let you cache them for a year at the CDN.

**C. Adobe Client Data Layer (ACDL)**

Core Components push component data and events automatically when the data layer is enabled via context-aware config (`/conf/mysite/sling:configs/com.adobe.cq.wcm.core.components.internal.DataLayerConfig` with `enabled=true`).

```js
// Listen in Adobe Tags or your own JS
window.adobeDataLayer = window.adobeDataLayer || [];
window.adobeDataLayer.push((dl) => {
  dl.addEventListener('cmp:click', (event) => {
    const id = event.eventInfo.path.split('.').pop();
    const component = dl.getState(event.eventInfo.path);
    console.log('clicked', id, component['dc:title']);
  });
});
// Push your own event
window.adobeDataLayer.push({ event: 'form:submitted', eventInfo: { formId: 'newsletter' } });
```

Custom components join the data layer by implementing `getData()` with `ComponentData` (via `DataLayerBuilder`) in the Sling Model and rendering `data-cmp-data-layer="${model.data.json}"` in HTL.

**D. Images: web-optimized delivery**

- Enable on the Image component policy ("Enable web optimized images"); Core Image v3 then renders `srcset` URLs served by AEM's image delivery (on the CDN side), converting to WebP/AVIF and resizing per width.
- Configure widths in the policy (e.g. 320, 640, 960, 1280, 1920); set `loading="lazy"` except the LCP hero (mark it as not lazy).
- Dynamic Media alternative: smart crop renditions and DM URLs (licensed).

**E. Responsive grid and layout**

- Breakpoints are defined in the template's `cq:responsive` node (phone, tablet, desktop widths); the Layout Container generates `aem-GridColumn--default--6` style classes.
- Include the `responsivegrid` LESS/SCSS (from the archetype) in your site CSS; designers often replace the grid with their design-system grid and keep AEM's class names.

**F. i18n and dictionaries**

- UI strings in HTL use `${'Read more' @ i18n}`; dictionaries in `/apps/mysite/i18n/<lang>.json` (code) or managed by authors in `/content/dictionaries` via the Translator UI (content, translatable via TIF).
- Locale comes from the page language (`jcr:language` on the language root).

**G. Accessibility and quality (expected by clients in 2026)**

- Target WCAG 2.2 AA (EU European Accessibility Act in force since June 2025 makes this a legal requirement for many sites).
- Core Components are built accessible; your CSS and custom components must keep focus states, contrast, ARIA labels.
- Automate: axe checks in `ui.tests`, Lighthouse in the experience audit, manual screen-reader testing on key journeys.

**H. Front-end pipeline theme (detailed workflow)**

1. Download the theme sources from the site (Sites console → site → "Download theme sources") or start from Adobe's standard site theme.
2. Configure `.env` with the AEM environment URL and site path, run `npm install` and `npm run live`: a local proxy serves pages from the cloud environment with your local CSS/JS.
3. Commit to the theme repo/folder; the **front-end pipeline** runs `npm run build` and deploys the `dist` output; the site's theme reference is updated so pages load the new version.
4. Combine with the full-stack pipeline: back end changes go through full-stack; theme changes go through front-end pipeline; don't put the same files in both.

**Expert-level interview answer: "LCP on the home page is 4.2 s on mobile. Walk me through it."** Measure with RUM/CrUX first, then Lighthouse. Typical AEM causes: hero image lazy-loaded or too large (fix: not lazy, `fetchpriority=high`, web-optimized widths), render-blocking CSS/JS in head (split critical CSS, defer JS, move Adobe Tags/Target to async with anti-flicker scope limited), web fonts (preload, `font-display: swap`), uncached HTML (CDN TTL, `stale-while-revalidate`), heavy third-party tags. Verify in the Cloud Manager experience audit and Sites Optimizer; if the site is marketing-led, propose EDS for the next phase.

### References

- [Core Components (GitHub)](https://github.com/adobe/aem-core-wcm-components)
- [Site Templates](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/sites/administering/site-creation/site-templates)
- [Front-end pipeline](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/implementing/using-cloud-manager/front-end-pipeline)
- [Page Editor and Universal Editor](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/implementing/developing/universal-editor/page-editor-universal-editor)
- [SPA Editor deprecation](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/implementing/developing/hybrid/spa-editor-deprecation)

## Module 6: Edge Delivery Services (EDS)

Edge Delivery Services is the biggest change since 2020 and the topic you are most likely to be grilled on. It is a separate, serverless delivery layer (formerly Project Helix / Franklin) that turns authored content into pre-rendered HTML served from the CDN edge, with plain JavaScript and CSS "blocks" instead of HTL components and Java. Target: Lighthouse 100 and very fast builds.

### 6.1 How it works

1. **Code** lives in a GitHub repo (start from `adobe/aem-boilerplate`, or `aem-boilerplate-xwalk` for Universal Editor authoring). The **AEM Code Sync** GitHub app syncs every push to the edge instantly. No build step, no Maven, no bundler.
2. **Content** comes from one of three authoring sources (6.2).
3. **Preview and publish**: authors use the AEM **Sidekick** browser extension (or the Universal Editor publish button) to preview and publish.
4. **Delivery**: the content is converted to semantic HTML (sections, blocks, default content) and cached at the edge. URLs: `https://<branch>--<repo>--<owner>.aem.page` (preview) and `.aem.live` (live). Production sits behind your own CDN or the Adobe-managed CDN via Cloud Manager.
5. **Branches are environments**: every Git branch gets its own preview URL, which makes feature reviews trivial.

### 6.2 Authoring options

| Option | Where authors write | Needs AEM author? | Good for |
| --- | --- | --- | --- |
| Document-based | Google Docs/Sheets or Microsoft Word/Excel (SharePoint) | No | Marketing teams, fast launches, simple governance |
| Document Authoring (DA, da.live) | Adobe's browser-based doc editor | No | Doc-style authoring without Google/Microsoft dependency; scales well |
| AEM authoring with Universal Editor ("xwalk") | AEM Sites console + Universal Editor (WYSIWYG) | Yes (AEMaaCS author) | Enterprises needing AEM workflows, MSM, translation, permissions, DAM integration |

Adobe's guidance on the default has shifted between versions of the docs; in an interview, explain the trade-off rather than claiming one is always right. Content mounting is defined by `fstab.yaml` (or the newer configuration service), e.g. AEM authoring mount: `https://<aem-author>/bin/franklin.delivery/<owner>/<repo>/main`.

### 6.3 Code: a block

A "block" is a table in a document (or a component in Universal Editor) that becomes a `div` with a class. The block's JS receives the DOM element and decorates it:

```text
/blocks/cards/cards.js
/blocks/cards/cards.css
/scripts/aem.js        # core library: loadBlock, decorate, createOptimizedPicture
/scripts/scripts.js    # page loading: eager / lazy / delayed phases
/styles/styles.css
```

```js
// blocks/cards/cards.js
import { createOptimizedPicture } from '../../scripts/aem.js';

export default function decorate(block) {
  const ul = document.createElement('ul');
  [...block.children].forEach((row) => {
    const li = document.createElement('li');
    while (row.firstElementChild) li.append(row.firstElementChild);
    [...li.children].forEach((div) => {
      div.className = div.querySelector('picture') ? 'cards-card-image' : 'cards-card-body';
    });
    ul.append(li);
  });
  ul.querySelectorAll('picture > img').forEach((img) =>
    img.closest('picture').replaceWith(
      createOptimizedPicture(img.src, img.alt, false, [{ width: '750' }])));
  block.replaceChildren(ul);
}
```

For Universal Editor (xwalk) projects you also define the block's authoring model in JSON (`component-definition.json`, `component-models.json`, `component-filters.json`), which tells the editor which fields to show.

### 6.4 The performance model ("keeping it 100")

- **Three-phase loading**: *Eager* (only what's needed for LCP: first section, hero image), *Lazy* (rest of page, blocks, header/footer), *Delayed* (3+ seconds later: martech, analytics, chat widgets, Adobe Tags).
- No frameworks, no build, minimal JS. Third-party scripts go in the delayed phase.
- **Real Use Monitoring (RUM)** is built in, collecting Core Web Vitals from real users.
- PSI/Lighthouse checks run on every pull request through the AEM Code Sync bot.

### 6.5 Other EDS building blocks

- **Spreadsheets as data**: a Google Sheet / Excel becomes a JSON endpoint (e.g. `/redirects.json`, `/placeholders.json`, `/query-index.json`) used for redirects, i18n strings, navigation, lists.
- **Indexing**: `helix-query.yaml` defines a query index of pages (title, image, date) for listing blocks and search.
- **Metadata**: bulk metadata sheet sets meta tags per URL pattern.
- **Forms on EDS**: forms block, or Adaptive Forms authored in Universal Editor and rendered on EDS (Module 10).
- **Commerce storefront on EDS**: Adobe Commerce drop-in components (cart, checkout, PDP) on EDS (Module 11).
- **Experimentation**: built-in A/B testing via metadata (`experiment`) and RUM, without Target, for simple cases.
- **Assets**: images from AEM Assets / Dynamic Media with OpenAPI can be referenced from EDS pages.
- **Cloud Manager integration**: EDS sites can be onboarded in Cloud Manager, including domain, SSL and CDN config.

### 6.6 EDS vs classic AEM Sites (decision table)

| Need | Lean towards |
| --- | --- |
| Lighthouse 100, fast time-to-market, marketing site | EDS |
| Front-end team with plain JS skills; no Java team | EDS |
| Heavy server-side logic per request, personalized authenticated portal | Classic AEM (Dispatcher + HTL) or headless + app server |
| Complex enterprise authoring: MSM live copies, translation, workflows, granular permissions | EDS with Universal Editor (AEM authoring) or classic |
| Existing large HTL codebase, big component library | Classic now; migrate incrementally (EDS for new sections) |
| Closed user groups / logged-in content | Classic, or EDS with edge auth patterns (more design work) |

### 6.7 Interview angle

**Q: "Explain EDS to a client CTO in two minutes."** Content is authored in documents or AEM, code lives in GitHub with zero build, and pages are pre-rendered and served from the edge. It gives near-perfect Core Web Vitals, very low hosting complexity, and branch-based previews. AEM still provides authoring governance when you choose Universal Editor authoring.

**Q: "Can EDS and classic AEM coexist on one domain?"** Yes. Route paths at the CDN (origin selectors): `/blog/*` to EDS, the rest to AEM publish. This is the common incremental migration pattern.

**Q: "Where does personalization go?"** Adobe Target / Web SDK loaded in the right phase, EDS experimentation for simple tests, or edge-side logic. Keep LCP unaffected.

### 6.8 Deep dive (handbook): building an EDS project end to end

**Step 1: Create the project**

1. On GitHub: "Use this template" from `adobe/aem-boilerplate` (document authoring) or `adobe-rnd/aem-boilerplate-xwalk` (Universal Editor/AEM authoring).
2. Install the **AEM Code Sync** GitHub app on the repo.
3. Point content at the source in `fstab.yaml` (or via the configuration service for newer setups):

```yaml
# fstab.yaml — document-based (SharePoint) example
mountpoints:
  /: https://myco.sharepoint.com/sites/web/Shared%20Documents/website

# AEM authoring (xwalk) example
mountpoints:
  /:
    url: "https://author-p123-e456.adobeaemcloud.com/bin/franklin.delivery/myorg/mysite/main"
    type: "markup"
    suffix: ".html"
```

4. Local dev: `npm i -g @adobe/aem-cli` then `aem up` — serves your local code against the remote preview content at `http://localhost:3000`.
5. Install the **Sidekick** extension for authors; configure it for the project.

**Step 2: Understand the three "buses"**

| Bus | Holds | Updated by |
| --- | --- | --- |
| Code bus | JS, CSS, icons, config from GitHub | Code Sync app on every push |
| Content bus | Converted content (markdown → HTML) | Preview/publish from Sidekick, UE or Admin API |
| Media bus | Images/videos extracted from content, content-addressed (hash URLs) | Preview of content |

The HTML pipeline renders a page as `<header>`, `<main>` with `<div>` sections, blocks as `<div class="blockname variant">`, and `<footer>`; `head.html` from the repo is injected into every page.

**Step 3: Page lifecycle in `scripts/scripts.js`**

```js
import { decorateBlocks, decorateSections, decorateIcons, decorateButtons,
  loadSection, loadSections, loadHeader, loadFooter, loadCSS, waitForFirstImage,
  buildBlock, sampleRUM } from './aem.js';

function buildAutoBlocks(main) {
  // e.g. turn the first picture + h1 into a hero block automatically
  const h1 = main.querySelector('h1');
  const picture = main.querySelector('picture');
  if (h1 && picture && (h1.compareDocumentPosition(picture) & Node.DOCUMENT_POSITION_PRECEDING)) {
    const section = document.createElement('div');
    section.append(buildBlock('hero', { elems: [picture, h1] }));
    main.prepend(section);
  }
}

export function decorateMain(main) {
  decorateButtons(main); decorateIcons(main);
  buildAutoBlocks(main); decorateSections(main); decorateBlocks(main);
}

async function loadEager(doc) {               // only what LCP needs
  document.documentElement.lang = 'en';
  const main = doc.querySelector('main');
  if (main) {
    decorateMain(main);
    document.body.classList.add('appear');
    await loadSection(main.querySelector('.section'), waitForFirstImage);
  }
}

async function loadLazy(doc) {                // rest of the page
  await loadSections(doc.querySelector('main'));
  loadHeader(doc.querySelector('header'));
  loadFooter(doc.querySelector('footer'));
  loadCSS(`${window.hlx.codeBasePath}/styles/lazy-styles.css`);
}

function loadDelayed() {                      // martech, chat, etc.
  window.setTimeout(() => import('./delayed.js'), 3000);
}

async function loadPage() {
  await loadEager(document);
  await loadLazy(document);
  loadDelayed();
}
loadPage();
```

(Function names follow the current boilerplate's `aem.js`; check the version in your repo.)

**Step 4: Authoring conventions authors must learn (document-based)**

- **Block** = a table whose first row is the block name: `Cards`, variants in brackets `Cards (dark, wide)` → `class="cards dark wide"`.
- **Section** = content between horizontal rules (`---`); **Section Metadata** table sets section styles.
- **Metadata** table at the bottom of the doc: Title, Description, Image, Template, Theme, Keywords.
- **Fragments**: reusable content (e.g. `/fragments/promo`) embedded with a fragment block.
- **Links** to YouTube, forms, etc. are auto-blocked into embeds.

**Step 5: Universal Editor (xwalk) block models**

```json
// component-definition.json (part)
{ "groups": [{ "title": "Blocks", "id": "blocks", "components": [{
  "title": "Hero", "id": "hero",
  "plugins": { "xwalk": { "page": {
    "resourceType": "core/franklin/components/block/v1/block",
    "template": { "name": "Hero", "model": "hero" } } } } }] }] }
```

```json
// component-models.json (part)
[{ "id": "hero", "fields": [
  { "component": "reference", "valueType": "string", "name": "image", "label": "Image", "multi": false },
  { "component": "text", "valueType": "string", "name": "imageAlt", "label": "Alt text" },
  { "component": "richtext", "valueType": "string", "name": "text", "label": "Text" }
] }]
```

`component-filters.json` restricts which blocks can be added where (e.g. only `cards-item` inside `cards`). Content is stored as AEM pages in `/content/mysite`, so MSM, translation, workflows and permissions all work.

**Step 6: Indexing, sitemap, redirects, headers**

```yaml
# helix-query.yaml
version: 1
indices:
  blog:
    include: ['/blog/**']
    exclude: ['/blog/drafts/**']
    target: /blog/query-index.json
    properties:
      title:
        select: head > meta[property="og:title"]
        value: attribute(el, "content")
      image:
        select: head > meta[property="og:image"]
        value: match(attribute(el, "content"), "https:\/\/[^/]+(/.*)")
      lastModified:
        select: none
        value: parseTimestamp(headers["last-modified"], "ddd, DD MMM YYYY hh:mm:ss GMT")
```

- **Sitemap**: `helix-sitemap.yaml` generates `sitemap.xml` from the index (multi-language supported with hreflang).
- **Redirects**: a `redirects` spreadsheet (Source, Destination) → 301s at the edge.
- **Headers**: custom HTTP headers (CSP, CORS) per path via the headers configuration.
- **robots.txt**: from the repo or config.

**Step 7: Automate with the Admin API**

```bash
# Preview, then publish a page programmatically (e.g. from a CMS migration script)
curl -X POST "https://admin.hlx.page/preview/myorg/mysite/main/blog/hello" -H "Authorization: token $TOKEN"
curl -X POST "https://admin.hlx.page/live/myorg/mysite/main/blog/hello"    -H "Authorization: token $TOKEN"
# Check status
curl "https://admin.hlx.page/status/myorg/mysite/main/blog/hello"
```

**Step 8: Production CDN and go-live**

- Either the Adobe-managed CDN (onboard the EDS site in Cloud Manager, add domain + certificate) or BYO CDN (Fastly, Akamai, Cloudflare, CloudFront) pointing to `main--mysite--myorg.aem.live` with the documented config.
- Configure **push invalidation** so publishing purges your CDN instantly (credentials for your CDN stored in the project config).
- Site authentication: protect preview/live with token-based auth if the site or a section is not public.
- Go-live checklist: redirects from the old site, sitemap submitted, canonical URLs, 404 page, analytics in delayed phase, RUM enabled, Lighthouse 100 on key templates on mobile.

**Step 9: Quality and team workflow**

- Each PR gets a branch preview URL (`https://<branch>--mysite--myorg.aem.page`) and an automatic PSI check from the bot; merge only when green.
- ESLint (airbnb-base) and Stylelint ship with the boilerplate; add unit tests for complex blocks.
- Block library: start from the Block Collection, keep blocks small and content-driven; avoid client-side frameworks.
- RUM explorer (operational telemetry) for CWV, 404s, top pages, experiment results.

**Expert-level interview answer: "How would you migrate a 2,000-page classic AEM site to EDS?"** Inventory templates/components → map them to blocks (aim for \~20-40 blocks) → choose authoring (UE xwalk if they rely on MSM/translation/workflows) → build blocks and auto-blocking → import content with the AEM importer/Admin API or, for xwalk, by restructuring pages to block components → route sections progressively at the CDN (origin selectors: migrated paths to EDS, rest to AEM publish) → redirects, SEO parity checks, RUM comparison → cut over remaining paths and decommission the old templates.

### References

- [aem.live documentation](https://www.aem.live/docs/)
- [Developer tutorial (aem.live)](https://www.aem.live/developer/tutorial)
- [Keeping it 100 (performance)](https://www.aem.live/developer/keeping-it-100)
- [Block Collection](https://www.aem.live/developer/block-collection)
- [Authoring with AEM Sites for EDS](https://www.aem.live/docs/aem-authoring)
- [EDS overview (Experience League)](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/edge-delivery/overview)
- [Document Authoring vs Universal Editor (Adobe perspective)](https://experienceleague.adobe.com/en/perspectives/adobe-document-authoring-and-aem-as-a-cloud-service-on-edge-delivery)
- [AEM boilerplate (GitHub)](https://github.com/adobe/aem-boilerplate) and [xwalk boilerplate](https://github.com/adobe-rnd/aem-boilerplate-xwalk)

## Module 7: Headless and hybrid (Content Fragments, GraphQL, OpenAPI, Universal Editor)

In 2020 "headless AEM" meant Sling Model JSON exporters and an early GraphQL beta. In 2026 it is a mature stack: structured Content Fragments, GraphQL with persisted queries (cached at the CDN), OpenAPI-based delivery and management APIs, and the Universal Editor for visual in-context editing of any front end.

### 7.1 Content Fragments: what's new

- **Content Fragment Models** define structure (text, number, date, enum, JSON, fragment reference, content reference, tab placeholders). Enabled per `/conf` configuration.
- **New Content Fragment Editor** and **Content Fragments console** (separate from Assets UI): faster, supports nested references, variations, version compare, and AI-assisted variations.
- **Fragment references** let you model graphs (Article → Author → Image).
- **Content Fragment component** (Core) renders a CF on an HTL page (hybrid).
- **Validation, UI schemas, and "fragment templates"** guide authors.

### 7.2 GraphQL with persisted queries

AEM exposes one GraphQL endpoint per configuration, generated from CF Models. For production always use **persisted queries**: they are stored in AEM, called with HTTP GET, and therefore cacheable by Dispatcher and CDN.

```graphql
# Persisted as /mysite/article-by-slug
query ArticleBySlug($slug: String!) {
  articleList(filter: { slug: { _expressions: [{ value: $slug }] } }) {
    items {
      _path
      title
      body { html }
      author { name  avatar { ... on ImageRef { _dynamicUrl } } }
    }
  }
}
```

```bash
# Persist (author), then publish the query
curl -u admin:admin -X PUT \
  -H "Content-Type: application/json" \
  https://author/graphql/persist.json/mysite/article-by-slug \
  --data '{"query": "..."}'

# Call it (publish / CDN-cached GET)
GET https://publish/graphql/execute.json/mysite/article-by-slug;slug=hello-world
```

```js
// React / Next.js using the AEM Headless client
import AEMHeadless from '@adobe/aem-headless-client-js';
const aem = new AEMHeadless({ serviceURL: process.env.AEM_HOST, endpoint: '/content/graphql/global/endpoint.json' });
const { data } = await aem.runPersistedQuery('mysite/article-by-slug', { slug: 'hello-world' });
```

Architect points: set cache headers per persisted query (`cache-control`, `surrogate-control`), use `_dynamicUrl` for web-optimized images, paginate with `first`/`after` (cursor), and allow-list origins with CORS + Dispatcher filters.

### 7.3 OpenAPI-based AEM APIs (2024 onward)

Adobe is moving from servlet-style APIs to versioned OpenAPI specs published at developer.adobe.com:

| API | Purpose |
| --- | --- |
| Content Fragment **Delivery** with OpenAPI | Read published CFs and models by ID/path from publish/preview, with reference hydration, cursor pagination; publish bucket can be public, preview needs a token |
| Sites (CF management) API | Create/update/publish fragments and models from integrations (replaces the deprecated Assets HTTP API for CFs) |
| Assets Author API, Dynamic Media with OpenAPI | Manage assets, deliver approved assets with URNs |
| Forms Communications APIs | Generate documents (Module 10) |

Authentication for server-to-server calls: **OAuth Server-to-Server** credentials from Adobe Developer Console (JWT credentials were deprecated in 2024-2025). AEM local technical accounts from the Developer Console still exist for older APIs.

### 7.4 Universal Editor

The Universal Editor (UE) is a visual editor hosted by Adobe that loads *your* front end (any framework, any host) in an iframe and edits content in AEM directly.

```html
<!-- In the page head: which AEM author the content comes from -->
<meta name="urn:adobe:aue:system:aemconnection"
      content="aem:https://author-p123-e456.adobeaemcloud.com">
<script src="https://universal-editor-service.adobe.io/cors.js" async></script>

<!-- Instrument any element (React, Next.js, HTL, EDS) -->
<article data-aue-resource="urn:aemconnection:/content/dam/mysite/articles/hello/jcr:content/data/master"
         data-aue-type="reference" data-aue-filter="cf" data-aue-label="Article">
  <h1 data-aue-prop="title" data-aue-type="text">Hello</h1>
  <div data-aue-prop="body" data-aue-type="richtext">...</div>
</article>
```

- Works with Content Fragments (headless), with AEM pages (EDS xwalk), and with custom backends via the UE service.
- Extensible with App Builder UI extensions (custom side panels, buttons).
- Replaces SPA Editor for new projects; Content Fragment Editor remains the form-based option.

### 7.5 Hybrid patterns

- **Remote SPA**: a Next.js/React app hosted on Vercel/Netlify/Azure, content from AEM via persisted queries, edited with Universal Editor.
- **Hybrid**: HTL pages with embedded React widgets reading the same GraphQL; good for gradual adoption.
- **Omnichannel**: same CFs to web, mobile apps, kiosks (AEM Screens), and email.

### 7.6 Interview angle

**Q: "GraphQL in AEM is slow under load. What do you do?"** Switch to persisted queries (GET, cacheable), set long CDN TTLs with purge on publish, reduce query depth/references, add Oak indexes for filter fields, paginate, and avoid ad-hoc POST queries from browsers.

**Q: "Headless or EDS or classic for a new site?"** Classic/EDS if marketers own pages and need WYSIWYG; headless if the experience is an app (complex state, logged-in, multi-channel) and the front-end team owns rendering. Universal Editor reduces the authoring gap of headless.

**Q: "How do you secure a headless API?"** Publish is public by default for persisted queries; restrict with CUG/permissions plus token auth for private content, CORS allow-list, Dispatcher filters for `/graphql/execute.json/*` only, and block the raw `/content/graphql/global/endpoint.json` POST on publish if not needed.

### 7.7 Deep dive (handbook): headless end to end

**Step 1: Configuration and models**

1. Tools → Configuration Browser → create `/conf/mysite` with *Content Fragment Models* and *GraphQL Persistent Queries* enabled.
2. Apply the config to the DAM folder (`/content/dam/mysite` → Properties → Cloud Configuration).
3. Create models. Modelling rules of thumb:

| Rule | Why |
| --- | --- |
| Model by *content*, not by page layout (Article, Author, Product, Offer) | Reuse across channels |
| Use fragment references for relations, keep depth ≤ 3 | Query performance and author clarity |
| Use enumerations and validation (regex, required, unique) | Data quality |
| Separate translatable from non-translatable fields | Translation cost |
| Add a `slug` and use it for lookups instead of paths | Stable URLs across restructures |
| Use variations only for channel-specific text (e.g. short vs long) | Not for languages (use language copies) |

4. Create the GraphQL endpoint for `/conf/mysite` (Tools → GraphQL). Endpoint URL: `/content/_cq_graphql/mysite/endpoint.json`.

**Step 2: GraphQL features you'll use**

```graphql
# Paginated list with filter, sort, locale and references
query Articles($cursor: String, $category: String) {
  articlePaginated(
    first: 10, after: $cursor, sort: "publishDate DESC",
    filter: { category: { _expressions: [{ value: $category, _operator: EQUALS }] } }
  ) {
    edges { cursor node { _path title slug publishDate
      heroImage { ... on ImageRef { _dynamicUrl width height } }
      author { name } } }
    pageInfo { endCursor hasNextPage }
  }
}

# Single item by path, a variation, with all references
query ArticleByPath($path: String!) {
  articleByPath(_path: $path, variation: "summary") {
    item { title body { html plaintext } }
    _references { ... on ImageRef { _path _dynamicUrl } }
  }
}
```

Language: put language copies under `/content/dam/mysite/en`, `/de`, and filter by path or use the locale argument on list queries.

**Step 3: Persist with cache headers**

```bash
curl -u "$USER:$PASS" -X PUT \
  "https://author-p123-e456.adobeaemcloud.com/graphql/persist.json/mysite/articles" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "query Articles($cursor:String,$category:String){ ... }",
    "cache-control":   { "max-age": 300 },
    "surrogate-control": { "max-age": 3600, "stale-while-revalidate": 86400, "stale-if-error": 86400 }
  }'
```

Publish the persisted query (it's content under `/conf`) via replication or package; manage queries in the GraphiQL IDE in AEM, which can also publish them. Call: `GET /graphql/execute.json/mysite/articles;category=news;cursor=...` (variables are matrix parameters, URL-encoded).

**Step 4: CORS and Dispatcher**

```json
// ui.config/.../config.publish/com.adobe.granite.cors.impl.CORSPolicyImpl~mysite-graphql.cfg.json
{
  "alloworigin": ["https://www.mysite.com"],
  "alloworiginregexp": ["http://localhost:.*"],
  "allowedpaths": ["/graphql/execute.json/.*", "/content/_cq_graphql/mysite/endpoint.json"],
  "supportedheaders": ["Origin", "Accept", "Content-Type", "Authorization"],
  "supportedmethods": ["GET", "HEAD", "OPTIONS"],
  "maxage:Integer": 1800,
  "supportscredentials": false
}
```

Dispatcher: allow `GET /graphql/execute.json/*`, pass the `Origin` header in `clientheaders.any`, cache the responses (and include `Origin` in cache variation if needed), deny POST to the endpoint on publish unless required.

**Step 5: Next.js app with on-demand revalidation**

```js
// lib/aem.js
import AEMHeadless from '@adobe/aem-headless-client-js';
export const aem = new AEMHeadless({
  serviceURL: process.env.AEM_PUBLISH_URL,
  endpoint: '/content/_cq_graphql/mysite/endpoint.json',
  fetch,
});

// app/articles/[slug]/page.js  (App Router)
import { aem } from '@/lib/aem';
export const revalidate = 3600; // ISR fallback
export default async function Article({ params }) {
  const { data } = await aem.runPersistedQuery('mysite/article-by-slug', { slug: params.slug });
  const a = data.articleList.items[0];
  return (
    <article data-aue-resource={`urn:aemconnection:${a._path}/jcr:content/data/master`}
             data-aue-type="reference" data-aue-label="Article">
      <h1 data-aue-prop="title" data-aue-type="text">{a.title}</h1>
      <div data-aue-prop="body" data-aue-type="richtext"
           dangerouslySetInnerHTML={{ __html: a.body.html }} />
    </article>
  );
}

// app/api/revalidate/route.js — called by an App Builder action on AEM "CF published" events
import { revalidatePath } from 'next/cache';
export async function POST(req) {
  if (req.headers.get('x-secret') !== process.env.REVALIDATE_SECRET) return new Response('no', { status: 401 });
  const { slug } = await req.json();
  revalidatePath(`/articles/${slug}`);
  return Response.json({ revalidated: true });
}
```

Flow: author publishes CF → AEM emits an I/O event → App Builder action calls `/api/revalidate` → page is regenerated in seconds; the CDN TTL covers the rest.

**Step 6: Preview and private content**

- Authors preview against the **Preview tier** (`preview-p123-e456...`) or author with a token; the headless app reads an env var to switch endpoints (`AEM_PREVIEW_URL`) in a draft-mode route.
- Private content: protect folders with CUG on publish, call with a bearer token from a technical account in server-side code only; never expose tokens to the browser; disable CDN caching for authenticated responses or vary by auth.

**Step 7: Universal Editor details**

- Add the connection meta tag and the UE `cors.js` only in the editing context (e.g. when `?aue` or a preview flag is present) to keep production pages clean.
- Define component definitions/models/filters JSON (same format as EDS xwalk, 6.8 step 5) so authors can add/reorder components, not just edit text.
- Authors open the app in UE from the AEM Sites/CF console; their IMS login is used to write to AEM.
- Extensions: custom properties panel, custom field types via UI Extensibility.

**Expert-level interview answer: "How do you avoid the N+1 query problem in headless AEM?"** Model references so one persisted query returns the whole view (with `_references` or nested fields), cap depth, use pagination with cursors, and cache per query at the CDN. For listing + detail, use two persisted queries rather than one per item. For extreme fan-out, precompute a JSON (e.g. an App Builder job writing to a CDN-cached endpoint or the EDS index pattern).

### References

- [GraphQL API for Content Fragments](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/headless/graphql-api/content-fragments)
- [Persisted queries](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/headless/graphql-api/persisted-queries)
- [AEM APIs for structured content](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/headless/apis-headless-and-content-fragments)
- [CF Delivery with OpenAPI reference](https://developer.adobe.com/experience-cloud/experience-manager-apis/api/stable/contentfragments/delivery/)
- [AEMaaCS API index](https://developer.adobe.com/experience-cloud/experience-manager-apis/)
- [Universal Editor introduction](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/implementing/developing/universal-editor/introduction)
- [Universal Editor developer overview](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/implementing/developing/universal-editor/developer-overview)
- [AEM Headless client for JS (GitHub)](https://github.com/adobe/aem-headless-client-js)

## Module 8: Dispatcher, CDN, caching and security

The caching chain on Cloud Service is **Browser → (optional customer CDN) → Adobe CDN (Fastly) → Dispatcher (Apache) → AEM publish**. In 2020 you tuned only Dispatcher and the client's CDN; now the Adobe CDN is part of your design and is configured as code through the config pipeline.

### 8.1 Dispatcher on Cloud Service

- Same module, but a **fixed folder structure** validated by Adobe. Some directives are immutable (Adobe-owned files you include but can't change).
- **Flexible mode** (recommended; enable by adding the file `opt-in/USE_SOURCES_DIRECTLY`): you can have many vhost/farm files, rewrites, and use the web-tier pipeline to deploy only dispatcher changes.
- **Cache invalidation** is automatic on publish (content distribution triggers flush); no flush agents to configure.
- Local validation with the SDK: `./bin/validate.sh ./dispatcher/src`, run with `./bin/docker_run.sh`.

```text
dispatcher/src/
  conf.d/
    available_vhosts/mysite.vhost
    enabled_vhosts/mysite.vhost -> ../available_vhosts/mysite.vhost
    rewrites/rewrite.rules
    variables/custom.vars
  conf.dispatcher.d/
    available_farms/mysite.farm
    enabled_farms/mysite.farm
    filters/filters.any
    cache/rules.any
    clientheaders/clientheaders.any
  opt-in/USE_SOURCES_DIRECTLY
```

```text
# filters.any: deny by default, then allow
/0001 { /type "deny"  /url "*" }
/0100 { /type "allow" /extension '(css|js|png|jpg|webp|svg|woff2|json)' /path "/content/*" }
/0101 { /type "allow" /method "GET" /url "/graphql/execute.json/*" }
/0102 { /type "allow" /extension "html" /path "/content/mysite/*" }
```

### 8.2 Adobe-managed CDN and TTL strategy

- The Adobe CDN caches based on `Cache-Control` / `Surrogate-Control` headers you set in Apache (vhost) or AEM. Defaults: HTML \~5 minutes, clientlibs long (versioned).
- Typical design: HTML `max-age=300` with `stale-while-revalidate`, versioned clientlibs `max-age=31536000, immutable`, persisted GraphQL queries `max-age` per query, DAM images with web-optimized delivery.
- **Purge**: CDN purge API with a purge key configured in `cdn.yaml`, triggered by an event or a Sling job when an urgent change must go live.
- **BYO CDN**: allowed; your CDN must send `X-Forwarded-Host` and an `X-AEM-Edge-Key` secret to the Adobe CDN. Avoid double caching surprises (set TTLs consistently).

### 8.3 CDN configuration as code (`config/cdn.yaml`)

Deployed via the **config pipeline**. One file can hold traffic filter rules (incl. WAF), rate limits, redirects, request/response transformations, origin selectors (route paths to EDS or another backend), custom error pages and purge keys.

```yaml
kind: "CDN"
version: "1"
metadata:
  envTypes: ["dev", "stage", "prod"]
data:
  trafficFilters:
    rules:
      - name: rate-limit-per-ip
        when: { reqProperty: tier, equals: publish }
        rateLimit: { limit: 100, window: 10, penalty: 300, groupBy: [ { reqProperty: clientIp } ] }
        action: block
      - name: block-admin-paths
        when: { reqProperty: path, like: "/system/*" }
        action: block
      - name: waf-attacks          # needs the WAF-DDoS Protection licence
        when: { reqProperty: tier, matches: "author|publish" }
        action:
          type: block
          wafFlags: [ SQLI, XSS, TRAVERSAL, CMDEXE, LOG4J-JNDI, BAD-USER-AGENTS ]
  redirects:
    rules:
      - name: old-campaign
        when: { reqProperty: path, equals: "/summer-sale" }
        action: { type: redirect, status: 301, location: "/offers/summer" }
  originSelectors:
    rules:
      - name: blog-to-eds
        when: { reqProperty: path, like: "/blog/*" }
        action: { type: selectOrigin, originName: eds }
    origins:
      - name: eds
        domain: main--mysite--myorg.aem.live
```

Start WAF/rate rules in `action: log` mode, watch CDN logs (forwarded to Splunk etc.), then switch to `block`.

### 8.4 Authentication and authorization

| Tier | 2020 | Now |
| --- | --- | --- |
| Author | Local users, LDAP, SAML | **Adobe IMS** (Admin Console, federated SSO via the client's IdP); groups synced from IMS; AEM groups for permissions |
| Publish (end users) | SAML handler, custom login | SAML 2.0 handler or OAuth/OIDC authentication handler; Closed User Groups (CUG); user/group data sync across publish pods (enable via Adobe) |
| APIs | Basic auth, local users | OAuth Server-to-Server (Developer Console), local technical accounts, bearer tokens for preview delivery |

### 8.5 Network and other security topics

- **Advanced Networking**: flexible port egress, **dedicated egress IP** (so a client's backend can allowlist AEM), or **VPN** to the client network.
- **IP allowlists** for author, preview and publish in Cloud Manager.
- **Custom domains and SSL certificates** managed in Cloud Manager (Adobe-managed DV certs or your own EV/OV).
- Penetration testing: allowed on your code, coordinated with Adobe; Adobe does platform security and patching.
- Headers: CSP, HSTS, X-Frame-Options in Apache vhost or CDN response transformations.

### 8.6 Interview angle

**Q: "Bots hit the site with 10,000 requests a second. Response?"** Traffic filter rules in `cdn.yaml`: rate limit by client IP, block known bad user agents and countries if allowed, enable WAF flags (licensed), cache aggressively so origin is protected. Deploy via the config pipeline in minutes, no code release.

**Q: "Marketing changed a price; it must be live in 30 seconds."** Dispatcher flush is automatic; CDN TTL bounds the delay. Use a short HTML TTL with stale-while-revalidate, or trigger a CDN purge for that URL on publish.

**Q: "Client wants to keep Akamai."** Fine: Akamai → Adobe CDN, with `X-Forwarded-Host` and `X-AEM-Edge-Key`. Agree who owns WAF and TTLs to avoid double rules.

### 8.7 Deep dive (handbook): caching and edge security, complete

**A. Farm file (shortened, cloud format)**

```text
/mysitepublishfarm {
  /clientheaders { $include "../clientheaders/clientheaders.any" }
  /virtualhosts  { $include "../virtualhosts/mysite.vhosts.any" }
  /renders       { $include "../renders/default_renders.any" }
  /filter        { $include "../filters/filters.any" }
  /cache {
    /docroot "${DOCROOT}"
    /statfileslevel "2"            # invalidation granularity: /content/mysite/<lang>
    /rules { $include "../cache/rules.any" }
    /invalidate { $include "../cache/default_invalidate.any" }
    /ignoreUrlParams {
      /0001 { /glob "*" /type "deny" }        # params make a request uncacheable...
      $include "../cache/marketing_query_parameters.any"   # ...except utm_*, gclid etc.
    }
    /headers { "Cache-Control" "Surrogate-Control" "Expires" "Last-Modified" "Content-Type" "Content-Disposition" }
    /gracePeriod "2"
    /enableTTL "1"                 # respect Cache-Control max-age from AEM
  }
}
```

**Key cache concepts to explain:**

| Concept | Meaning | Typical choice |
| --- | --- | --- |
| `statfileslevel` | Depth of `.stat` files; an activation invalidates only files below that level's folder | 2-3 for multi-site/multi-language, so publishing in `/en` doesn't flush `/de` |
| Auto-invalidation | `.html`, `.json` etc. older than the `.stat` file are re-fetched | Default on |
| `ignoreUrlParams` | Which params are ignored for cache key | Ignore marketing params, deny others |
| `/enableTTL` | Dispatcher expires files using HTTP headers | On, combined with CDN TTL |
| `gracePeriod` | Serve stale content shortly after invalidation | 2 s to absorb invalidation bursts |

**B. Cache headers in the vhost**

```apache
<LocationMatch "^/content/.*\.html$">
  Header set Cache-Control "max-age=300,stale-while-revalidate=43200,stale-if-error=43200"
  Header set Surrogate-Control "max-age=3600"
  Header set Age 0
</LocationMatch>
<LocationMatch "^/etc\.clientlibs/.*\.(?i:js|css|ttf|woff2)$">
  Header set Cache-Control "max-age=31536000,immutable"
</LocationMatch>
<LocationMatch "^/content/dam/.*\.(?i:jpeg|jpg|png|gif|svg|webp|avif)$">
  Header set Cache-Control "max-age=86400,stale-while-revalidate=43200"
</LocationMatch>
# Security headers
Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
Header always set X-Content-Type-Options "nosniff"
Header always set Referrer-Policy "strict-origin-when-cross-origin"
Header always set Content-Security-Policy "frame-ancestors 'self'"
```

`Cache-Control` max-age applies to the browser (and the CDN if no `Surrogate-Control`); `Surrogate-Control` applies only to the CDN and is stripped before the browser. That lets you cache longer at the edge (purged on publish) and shorter in browsers.

**C. URL design: short URLs and vanity URLs**

- Rewrite `/en/products.html` → `/content/mysite/en/products.html` in Apache (`RewriteRule ^/(.*)$ /content/mysite/$1 [PT,L]` with conditions), and the reverse mapping in Sling (`/etc/map` or resource resolver mapping config) so links are generated short.
- Prefer extensionless URLs only if your rewrite rules and cache keys are consistent.
- `sling:vanityPath` works, but at scale keep vanity/redirect maps at the CDN (`cdn.yaml` redirects or Apache `RewriteMap` from a file) for performance.

**D. Full `cdn.yaml` features (beyond 8.3)**

```yaml
kind: "CDN"
version: "1"
metadata:
  envTypes: ["prod"]
data:
  authentication:
    authenticators:
      - name: purge-auth
        type: purge
        purgeKey1: ${{CDN_PURGE_KEY_1}}
        purgeKey2: ${{CDN_PURGE_KEY_2}}     # two keys allow rotation
      - name: edge-auth
        type: edge
        edgeKey1: ${{CDN_EDGE_KEY_1}}
        edgeKey2: ${{CDN_EDGE_KEY_2}}       # for BYO CDN in front
    rules:
      - name: purge-auth-rule
        when: { reqProperty: tier, equals: "publish" }
        action: { type: authenticate, authenticator: purge-auth }
  requestTransformations:
    removeMarketingParams: true            # better cache hit ratio
    rules:
      - name: set-country-header
        actions:
          - { type: set, reqHeader: x-country, value: { reqProperty: clientCountry } }
  responseTransformations:
    rules:
      - name: frame-options
        actions:
          - { type: set, respHeader: X-Frame-Options, value: SAMEORIGIN }
```

Purge a URL after an urgent change:

```bash
curl -X PURGE -H "X-AEM-Purge-Key: $CDN_PURGE_KEY_1" "https://www.mysite.com/en/offers.html"
# purge everything (use sparingly): add header  X-AEM-Purge: all
```

**E. Authenticated and personalized content**

- **CUG** pages on publish: Dispatcher can cache them with **permission-sensitive caching** (`/auth_checker` calling a servlet that checks the user's access with a HEAD request); otherwise mark them `Cache-Control: private`.
- Personalization: prefer client-side (Target/Web SDK, or a JSON endpoint per user that is not cached) and keep the HTML shell fully cacheable. Sling Dynamic Include (SSI) is possible at Dispatcher level but adds complexity; avoid ESI at the Adobe CDN.
- Never cache responses with `Set-Cookie` for a logged-in user; ensure `Vary` is correct.

**F. Security hardening checklist (publish)**

- [ ] Dispatcher filters: deny by default; allow only needed selectors/extensions; deny `/crx`, `/system`, `/bin` (except needed servlets), `/libs/granite`, `*.infinity.json`, `*.tidy.json`, `*.query.json`, `*.sysview.xml`, `*.docview.xml`.
- [ ] No anonymous write access; no default credentials (Cloud Service enforces IMS on author).
- [ ] Traffic filter rules: rate limit per IP, block admin paths, WAF flags (licensed), geo-blocking if required.
- [ ] CSP header (start with report-only), HSTS, X-Content-Type-Options, Referrer-Policy.
- [ ] Secrets only in Cloud Manager secret variables.
- [ ] Regular review of Developer Console access and IMS groups.
- [ ] Pen test coordinated with Adobe; OWASP Top 10 review of custom servlets (input validation, XSS via HTL context `@ context='...'`).

**Expert-level interview answer: "Cache hit ratio at the CDN is 40%. How do you get to 90%?"** Analyse CDN logs by URL pattern and `cache` field (HIT/MISS/PASS). Common causes: unique query strings (marketing params → `removeMarketingParams`, ignoreUrlParams), `Set-Cookie` or `Cache-Control: private` on public pages, short or missing TTLs, `Vary` on too many headers, personalized HTML. Fix headers in Apache, move personalization client-side, add `Surrogate-Control` with longer edge TTL and purge on publish, and pre-warm key pages after deployments.

### References

- [Dispatcher in the cloud](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/implementing/dispatcher/disp-overview)
- [CDN in AEMaaCS](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/implementing/content-delivery/cdn)
- [Configuring traffic at the CDN](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/implementing/content-delivery/cdn-configuring-traffic)
- [Traffic filter rules including WAF](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/security/traffic-filter-rules-including-waf)
- [Advanced networking](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/security/advanced-networking)

## Module 9: AEM Assets evolution

Assets moved from "a DAM inside AEM that runs workflows on the JVM" to a cloud-native content supply chain: binaries go straight to blob storage, processing runs in Adobe's asset microservices, and approved assets are delivered anywhere through Dynamic Media with OpenAPI and Content Hub.

### 9.1 What changed

| Topic | 2020 (6.5) | Now (AEMaaCS) |
| --- | --- | --- |
| Upload | Through the AEM JVM (servlet) | **Direct binary upload** to cloud storage (`aem-upload` lib / Bulk Import); JVM only gets metadata |
| Renditions, metadata extraction | DAM Update Asset workflow, ImageMagick, FFmpeg on the server | **Asset microservices** (cloud processing profiles); custom processing via **Asset Compute** workers on App Builder |
| Custom post-processing | Edit DAM Update Asset | **Post-processing workflows** (configured per folder) run after microservices finish |
| UI | Assets admin (Touch UI) | **Assets view** (simplified, for marketers) + Admin view; new Content Fragments console split out |
| Distribution to marketers/partners | Brand Portal, Asset Share Commons | **Content Hub** (approved, brand-safe self-service portal) + Brand Portal still exists |
| Delivery | Dynamic Media Scene7 / classic | **Dynamic Media with OpenAPI** (approved assets via stable URNs, smart crop, format/quality params); Dynamic Media Prime/Ultimate packages |
| Picker in other apps | Custom | **Asset Selector** micro-frontend embeddable in any app (Workfront, Commerce, your React app) |
| AI | Smart Tags | Smart Tags, AI-generated metadata, Firefly-based generative edits, "Generate variations", content credentials |
| Bulk ingest | Custom scripts | **Bulk Import** tool from S3/Azure; Content Transfer Tool for migration |

### 9.2 Code samples

**Direct binary upload from Node** (for integrations and migrations):

```js
import DirectBinary from '@adobe/aem-upload';
const upload = new DirectBinary.DirectBinaryUpload();
const options = new DirectBinary.DirectBinaryUploadOptions()
  .withUrl('https://author-p123-e456.adobeaemcloud.com/content/dam/mysite/products')
  .withUploadFiles([{ fileName: 'shoe.jpg', fileSize: 102400, filePath: './shoe.jpg' }])
  .withHttpOptions({ headers: { Authorization: `Bearer ${token}` } });
const result = await upload.uploadFiles(options);
```

**Custom Asset Compute worker** (App Builder action that creates a custom rendition):

```js
const { worker, SourceCorruptError } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

exports.main = worker(async (source, rendition, params) => {
  const stats = await fs.stat(source.path);
  if (stats.size === 0) throw new SourceCorruptError('empty source');
  // e.g. call an external watermarking API, then write the result
  await fs.copyFile(source.path, rendition.path);
});
```

The worker is registered in a processing profile; AEM calls it after upload. No code runs inside the AEM JVM.

**Dynamic Media with OpenAPI delivery URL** (approved assets only):

```text
https://delivery-p123-e456.adobeaemcloud.com/adobe/assets/urn:aaid:aem:1a2b.../as/shoe.webp?width=800&quality=80
```

### 9.3 Architect view

- Never process binaries in Java on AEMaaCS; use microservices, Asset Compute or external services via events.
- Separate **Assets-only** programs are common: one AEM Assets program serving several Sites programs, EDS sites, Commerce and Workfront ("remote DAM"). Sites can reference remote approved assets via Dynamic Media with OpenAPI.
- Governance: metadata schemas, folder-level processing profiles, approval status (drives what's deliverable), rights/expiry, content credentials.
- Integration with **Workfront** (work management, review and approval) is the typical enterprise content supply chain design.

### 9.4 Interview angle

**Q: "Client has a custom DAM Update Asset step that adds a watermark. How do you migrate it?"** Replace it with a custom Asset Compute worker (App Builder) in a processing profile, or a post-processing workflow if it's metadata-only. Remove the workflow launcher customization.

**Q: "Ingest 5 million assets from an old DAM."** Bulk Import from S3/Azure (or Content Transfer Tool if source is AEM), in batches, with processing profiles tuned, metadata mapped via CSV, and off-peak scheduling. Plan for reprocessing time.

### 9.5 Deep dive (handbook): designing and running AEM Assets

**A. Upload and processing flow (know it step by step)**

1. Client calls `initiateUpload` on the target folder → AEM returns signed upload URIs for blob storage.
2. Client uploads binary parts **directly to blob storage** (parallel, resumable).
3. Client calls `completeUpload` → AEM creates the asset node (metadata only).
4. AEM sends a processing request to **asset microservices** with the folder's **processing profile**.
5. Microservices generate renditions (standard: thumbnails, web rendition; custom: sizes/formats you define; Asset Compute custom workers), extract metadata and XMP, run Smart Tags.
6. Renditions are written back to blob storage; AEM is notified → **post-processing workflow** (if configured for the folder) runs, e.g. set approval status, notify, copy metadata.
7. Asset is searchable; once **Approved**, it's deliverable via Dynamic Media with OpenAPI / Content Hub.

**B. Processing profiles**

| Profile type | Example |
| --- | --- |
| Standard | JPEG 1280px web rendition, PNG thumbnails, extract text from PDFs |
| Custom (Asset Compute) | Watermark, call external AI for tagging, convert PSD to custom format |
| Video | Dynamic Media video encoding profiles |
| Image profiles (DM) | Smart crop definitions (e.g. 1:1, 16:9, 4:5) |

Apply profiles to folders; subfolders inherit. Reprocess existing assets after profile changes (folder → Reprocess).

**C. Metadata governance**

- **Metadata schemas** (Tools → Assets → Metadata Schemas) per folder: required fields, cascading fields, dropdowns from tags.
- **Tag taxonomy** in `/content/cq:tags/mysite` owned by a librarian role; Smart Tags trained/curated.
- **Approval and rights**: `dam:status` approved/rejected; expiry date (`prism:expirationDate`) blocks use after expiry; licence fields.
- **Content credentials** (C2PA) for AI-generated assets.
- Folder structure: by brand/campaign/product, not by year; permissions by folder via IMS groups.

**D. Asset Compute worker development**

```bash
aio app init my-workers --template @adobe/generator-app-asset-compute
cd my-workers
aio app run          # Asset Compute devtool: upload a file, run the worker locally
aio app test         # unit tests with test files and expected renditions
aio app deploy       # deploy to I/O Runtime; reference the worker URL in a processing profile
```

Keep workers stateless and fast (they have time and memory limits); for long tasks, call an external service and let it write back via API.

**E. Delivery options compared**

| Option | When |
| --- | --- |
| Web-optimized image delivery (Core Image v3) | Sites pages on the same program; no extra licence |
| Dynamic Media (classic Scene7-based) | Existing DM customers: image presets, viewers, 360, video |
| Dynamic Media with OpenAPI | Approved assets delivered to any channel (EDS, headless, Commerce, third parties) with stable URNs, transformations by URL |
| Content Hub | Marketers/partners self-serve approved, on-brand assets with search and download |
| Brand Portal | Legacy distribution to agencies; Content Hub is the newer path |

**F. Integrations**

- **Asset Selector**: an embeddable micro-frontend (IMS-authenticated) to pick AEM assets inside other apps (custom React apps, Commerce admin, Workfront).
- **Adobe Express / Creative Cloud** connectors so designers save straight into AEM Assets.
- **Workfront**: assets linked to projects, review/approval in Workfront, approved files land in AEM folders with metadata mapped.
- **Events**: AEM Assets events (asset processed, metadata changed, deleted) via Adobe I/O Events to drive PIM/CDN/search sync.

**G. Performance and scale considerations**

- Millions of assets are fine; avoid >1,000 children per folder for usability, and design search with custom `damAssetLucene` index properties for frequently filtered metadata.
- Bulk imports should be throttled and scheduled; monitor processing queues in the Assets UI.
- Storage and downloads affect licence consumption; clean unused renditions and duplicates (duplicate detection can be enabled).

**Expert-level interview answer: "Design a central DAM for 8 brands and 3 agencies."** One AEM Assets program as the system of record; folder per brand with brand-specific metadata schemas and processing profiles; IMS groups per brand and agency (agencies get upload to WIP folders, not approved ones); approval workflow (or Workfront review) setting `Approved`; Content Hub for consumers; Dynamic Media with OpenAPI for delivery to Sites/EDS/Commerce/social tools; Asset Selector embedded in each channel tool; events to sync PIM; governance board for taxonomy; metrics on asset reuse.

### References

- [AEM Assets as a Cloud Service overview](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/overview)
- [Asset microservices overview](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-overview)
- [Integrate remote approved assets with Sites](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/dynamicmedia/dynamic-media-open-apis/integrate-remote-approved-assets-with-sites)
- [AEMaaCS API index (Assets APIs)](https://developer.adobe.com/experience-cloud/experience-manager-apis/)
- [App Builder overview](https://developer.adobe.com/app-builder/docs/overview/)

## Module 10: AEM Forms (2020 → 2026)

AEM Forms moved to the cloud (Forms as a Cloud Service), rebuilt its Adaptive Forms on **Core Components**, added **headless forms** and **forms on Edge Delivery Services** authored in Universal Editor, and turned document generation into **Communications APIs**. AEM Forms JEE (Process Management, Workbench) remains on-prem/AMS only, now on 6.5 LTS with Java 17/21 and JBoss EAP 8.

### 10.1 What changed

| Topic | 2020 | Now |
| --- | --- | --- |
| Deployment | AEM Forms 6.5 OSGi or JEE (on-prem/AMS) | Forms as a Cloud Service (OSGi capabilities); JEE only on 6.5 / 6.5 LTS |
| Form components | Foundation components, form templates | **Adaptive Forms Core Components** (open source, BEM, accessible, Style System, themes via front-end pipeline) |
| Rendering | Server-rendered only | Core (HTL), **Headless Adaptive Forms** (JSON spec + React/Angular renderers), **EDS forms** (forms block, Universal Editor authoring) |
| Authoring | Adaptive Form editor | AF editor, Universal Editor for EDS forms, AI-assisted authoring (generate forms from prompts/PDFs) |
| Logic | Rule editor + scripts | Visual Rule Editor + **custom functions** (JS with JSDoc annotations), direct API calls (EA) |
| Data integration | Form Data Model (FDM) over RDBMS, REST, SOAP, OData | FDM still central; plus submit to REST, email, Power Automate, Workfront Fusion, SharePoint/OneDrive, Azure Blob, Salesforce via FDM, AEP |
| Document of record / output | Output & Forms services on JEE | **Communications APIs** (OpenAPI): document generation, PDF manipulation (assemble, sign, protect), Forms Service APIs (render XDP, import/export data), document conversion (HTML→PDF, PDF→image) |
| Interactive Communications | IC editor (print/web) | New Interactive Communication Editor with FDM data binding and rule editor (2026) |
| Conversion | Manual | Automated Forms Conversion Service (PDF → adaptive form) |
| Spam/bot | reCAPTCHA | reCAPTCHA Enterprise, hCaptcha, Cloudflare Turnstile |
| Signatures | Adobe Sign | Adobe Acrobat Sign integration; DocuSign via custom |
| Versioning | Limited | Forms Manager versioning for forms, fragments, themes, XDP (2026) |

### 10.2 Code samples

**Custom function** used in the Visual Rule Editor (place in a client library referenced by the form, or `scripts/functions.js` in EDS forms). JSDoc annotations make it appear in the rule editor:

```js
/**
 * Calculates age in years from a date of birth
 * @name calculateAge
 * @param {string} dob date in YYYY-MM-DD
 * @return {number} age in years
 */
function calculateAge(dob) {
  const birth = new Date(dob);
  const now = new Date();
  let age = now.getFullYear() - birth.getFullYear();
  const m = now.getMonth() - birth.getMonth();
  if (m < 0 || (m === 0 && now.getDate() < birth.getDate())) age -= 1;
  return age;
}
```

**Headless Adaptive Form definition** (simplified; rendered by `@aemforms/af-react-renderer` in a React app or mobile):

```json
{
  "adaptiveform": "0.10.0",
  "items": [
    { "fieldType": "text-input", "name": "firstName", "label": { "value": "First name" }, "required": true },
    { "fieldType": "email", "name": "email", "label": { "value": "Email" } },
    { "fieldType": "date-input", "name": "dob", "label": { "value": "Date of birth" } },
    { "fieldType": "button", "name": "submit", "label": { "value": "Submit" },
      "events": { "click": "submitForm()" } }
  ]
}
```

**EDS forms**: in document-based authoring a form is defined in a spreadsheet (one row per field: Name, Type, Label, Mandatory, Options) and placed with a `Form` block; with Universal Editor authoring you drag Adaptive Form components onto an EDS page and choose a submit action (including custom submit actions, 2026).

### 10.3 Typical architectures

- **Customer onboarding (bank/insurance)**: Adaptive Form (Core or EDS) → prefill from CRM via FDM → save-and-resume (drafts) → Acrobat Sign → submit to an integration layer (API gateway / App Builder) → Document of Record PDF archived.
- **High-volume correspondence**: XDP templates + data from core systems → Communications APIs batch generation → print/email channel. On-prem JEE only if heavy Process Management orchestration is mandated.
- **Public-sector forms with performance targets**: Forms on EDS for Lighthouse 100, Universal Editor authoring, submit to REST.

### 10.4 Interview angle

**Q: "We have 300 foundation-component adaptive forms on 6.5. Migrate to cloud?"** Run BPA for Forms, move as-is (foundation forms still work on Cloud Service), then convert progressively to Core Components (conversion utility + manual review of scripts/rules); move JEE processes to AEM Workflows, App Builder or external BPM.

**Q: "Where do form submissions go on Cloud Service?"** Never stored in the publish repository long-term. Use a submit action to a system of record (REST, FDM, Power Automate, Azure Blob, AEP), or the Forms Portal draft/submission storage backed by an external database via FDM.

**Q: "JEE features without equivalent in the cloud?"** Process Management (long-lived processes, Workbench), some Output/Assembler deployments → Communications APIs, AEM Workflows, or external BPM.

### 10.5 Deep dive (handbook): building enterprise forms

**A. Choosing the form technology**

| Requirement | Choose |
| --- | --- |
| New form, AEM Sites (HTL) | Adaptive Form with **Core Components** |
| Max performance, public form on an EDS site | **Forms on EDS** (Universal Editor authoring) |
| Native mobile app or React SPA owns the UI | **Headless adaptive form** (JSON) + renderer |
| Existing XDP/PDF-based process, print fidelity | XFA/PDF forms via Forms Service APIs, or convert with Automated Forms Conversion |
| Simple contact form on EDS with docs authoring | EDS forms block defined in a spreadsheet |

**B. Form Data Model (FDM) step by step**

1. Create a **data source** cloud configuration: RESTful service (upload its OpenAPI/Swagger spec), SOAP (WSDL), OData (e.g. Dynamics), Salesforce, or RDBMS (JDBC datasource OSGi config; database must be reachable via Advanced Networking).
2. Create the FDM: add data model objects (e.g. `Customer`) and services (`getCustomer`, `updateCustomer`) from the data source.
3. Configure **read/write services** and bindings (e.g. `getCustomer` with argument bound to the user profile attribute or a request parameter).
4. Test services inside the FDM editor with sample input.
5. Build the form based on the FDM: fields bind to model properties (`bindRef`), so prefill and submit map automatically.
6. Use **Prefill service**: FDM read service, or a custom prefill service (OSGi) for complex logic.
7. Submit with "Submit using Form Data Model" to write back, or invoke an FDM service from a rule (e.g. look up an address by postcode).

Authentication to backends: OAuth 2.0, API key, basic, mutual TLS (certificates via keystore), with secrets in Cloud Manager variables where supported.

**C. Submit actions overview**

| Submit action | Use case |
| --- | --- |
| Submit to REST endpoint | Your API gateway / App Builder action (most common in enterprises) |
| Submit using Form Data Model | Write to CRM/DB through FDM services |
| Send email | Low-volume notifications (avoid PII in emails) |
| Invoke AEM Workflow | Approval routing, Document of Record, Sign steps on author |
| Microsoft Power Automate / SharePoint / OneDrive / Azure Blob | Microsoft-centric clients |
| Workfront Fusion | No-code integrations |
| Adobe Experience Platform / Marketo | Lead capture into marketing |
| Custom submit action | Your own server-side logic (Java) when none fit |

**D. Rules and logic**

- Visual Rule Editor: When/Then rules (show/hide, set value, validate, invoke service, navigate panels, dispatch events).
- Custom functions (10.2) for reusable logic; they are client-side JS, so never put secrets in them.
- Validation: client-side for UX, **server-side revalidation** on submit (Core Components forms support server-side validation of constraints) for security.
- Large forms: use panels/wizards, lazy loading of fragments, and **form fragments** for reusable sections (address, applicant details).

**E. Document of Record (DoR) and signing**

- DoR: PDF version of the submitted data, auto-generated from the form or from an XDP template; configured in form properties; stored or attached in the workflow.
- Acrobat Sign: configure the Sign cloud configuration (OAuth), enable signing in the form (signers, order, authentication like SMS/KBA), handle the callback to complete the process.

**F. Communications APIs (document generation) pattern**

1. Design XDP templates (Forms Designer) or use the new Interactive Communication editor with FDM binding.
2. Store templates in AEM (Forms & Documents) or cloud storage.
3. Call the API from server-side code with an OAuth Server-to-Server token: synchronous for single documents (e.g. a confirmation letter), batch for volume (input data in cloud storage, output written back).
4. Post-process: assemble, protect, sign, convert (HTML → PDF, PDF → image), then deliver via email/print/archive.

**G. Forms security and compliance checklist**

- [ ] No PII stored in AEM long-term; drafts only if required, with retention policy.
- [ ] Captcha (reCAPTCHA Enterprise, hCaptcha or Turnstile) on public forms; traffic filter rate limits on submit URLs.
- [ ] File attachments: size/type limits, virus scanning before persistence (EA scanner hook in 2026, or scan in your submit endpoint).
- [ ] Server-side validation; output encoding.
- [ ] Accessibility WCAG 2.2 AA (labels, error messages, keyboard navigation) — Core Components help.
- [ ] Analytics without PII (track step completion and abandonment only).

**H. Migrating Forms from 6.5 to Cloud Service**

- Run BPA, check Forms-specific findings (e.g. JEE dependencies, custom submit actions, workflows).
- Move forms, themes, fragments, FDMs and data source configs via packages/CTT; recreate cloud configurations (credentials) per environment.
- Re-test every FDM integration with new egress IPs.
- Convert Foundation forms to Core Components progressively (conversion utility + manual fix of scripts).
- Replace JEE-only features: Process Management → AEM Workflow/App Builder/BPM; Output/Assembler → Communications APIs.

**Expert-level interview answer: "Design a loan application journey for 1 million applications a year."** EDS-hosted or Core Components adaptive form with panels and save/resume; prefill from CRM via FDM after customer login (OIDC); address/credit lookups via FDM services from rules; attachments scanned; submit to the bank's API gateway (async processing via queue), Document of Record generated and archived in ECM; Acrobat Sign for consent; Communications API generates offer letters in batch; analytics on step drop-off; traffic filters + captcha; load test the submit API, not just AEM; no PII retained in AEM.

### References

- [AEM Forms as a Cloud Service introduction](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/forms/forms-overview/home)
- [Adaptive Forms Core Components (GitHub)](https://github.com/adobe/aem-core-forms-components)
- [Edge Delivery Services for AEM Forms](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/edge-delivery/build-forms/overview)
- [Forms on aem.live](https://www.aem.live/developer/forms)
- [Forms early access features](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/forms/forms-overview/early-access-ea-features)
- [AEMaaCS release notes (Forms section each month)](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/release-notes/release-notes/release-notes-current)

## Module 11: Integrations, extensibility and other AEM capabilities

The modern rule: **keep custom logic out of AEM where you can**. Extend AEM with Adobe App Builder (serverless on Adobe I/O Runtime), react to AEM events via Adobe I/O Events, extend the UI with UI Extensibility, and integrate the Experience Cloud via Web SDK / Adobe Experience Platform.

### 11.1 App Builder, I/O Events and UI Extensibility

| Capability | What it is | Use it for |
| --- | --- | --- |
| **App Builder** | Adobe-hosted serverless (I/O Runtime, Node.js actions) + React SPA hosting, with IMS auth | Integrations (ERP, PIM, CRM), heavy processing, custom admin apps, Asset Compute workers |
| **Adobe I/O Events for AEM** | AEM emits events (page/CF/asset published, modified, deleted) to Adobe I/O | Event-driven sync: push content to search (Algolia/Elastic), notify downstream systems, purge caches |
| **UI Extensibility** | Extension points in Content Fragment console/editor, Universal Editor, Assets view | Add buttons, side panels, custom fields (e.g. "Translate with DeepL", "Generate SEO description") without touching `/libs` |
| **API Mesh** | GraphQL federation on App Builder | Combine AEM + Commerce + PIM into one API for the front end |

```js
// App Builder action triggered by an AEM I/O event (e.g. a Content Fragment published)
const { Core } = require('@adobe/aio-sdk');

async function main(params) {
  const logger = Core.Logger('cf-published', { level: params.LOG_LEVEL || 'info' });
  const event = params.data || {};          // event payload from Adobe I/O Events
  logger.info(`Event type: ${params.type}, path: ${event.path}`);
  // 1. fetch the fragment via the CF Delivery OpenAPI
  // 2. push it to the search index / downstream system
  return { statusCode: 200, body: { ok: true } };
}
exports.main = main;
```

### 11.2 Experience Cloud integration

- **Data collection**: Adobe Web SDK (`alloy.js`) via Adobe Tags, fed by the Adobe Client Data Layer from Core Components; replaces AppMeasurement and at.js on new projects.
- **Adobe Target**: Experience Fragments and Content Fragments exported as offers; personalization via Web SDK; on EDS loaded with care for LCP.
- **Adobe Experience Platform / Real-Time CDP / Journey Optimizer**: AEM content (XFs, CFs) used in AJO emails and journeys; AEP audiences used for personalization.
- **Workfront**: project management, review/approval and asset handoff to AEM Assets (Workfront for AEM enhanced connector / native integration).
- **GenStudio for Performance Marketing**: generates on-brand campaign variants using AEM Assets as the source.

### 11.3 Commerce

- **CIF (Commerce Integration Framework)**: AEM add-on that connects AEM Sites to Adobe Commerce (or other platforms via GraphQL) with CIF Core Components; still supported on Cloud Service.
- **Adobe Commerce Storefront on EDS** (2024+): "drop-in" components (product details, cart, checkout, account) built for EDS; now Adobe's recommended storefront pattern for new builds.
- Product content enrichment: Commerce product data + AEM Content Fragments/Assets; Asset Selector inside Commerce admin.

### 11.4 Multi-site, multi-language (still core architect territory)

- **MSM (Blueprints, Live Copies, rollout configs)** unchanged in concept; on Cloud Service rollouts run as jobs; avoid huge synchronous rollouts.
- **Language copies + Translation Integration Framework**: connectors for Smartling, RWS/SDL, Lionbridge, XTM, etc. Machine translation built in (Adobe's Microsoft Translator based connector) on Cloud Service.
- **Content Fragments translation**: translatable fields flagged in models; Translation projects handle CFs and their references.
- **Launches** and **Sites Preview tier** for future-dated and review-before-publish content.
- **Experience Fragments** for headers/footers, campaign content and Target offers.
- **Context-aware configuration** for per-brand/per-market settings.

### 11.5 Other AEM products to know by name

| Product | Status 2026 | Note |
| --- | --- | --- |
| AEM Screens | On Cloud Service | Digital signage; player apps; content from Sites/Assets |
| AEM Guides | Cloud and on-prem | DITA-based technical documentation, multi-channel publishing |
| AEM Communities | Not on Cloud Service | Use third-party (Khoros, Discourse) or custom via App Builder |
| Brand Portal | Still available | Content Hub is the newer approach |
| Adobe Learning Manager, Campaign | Separate products | Integrate via APIs/XFs |

### 11.6 Interview angle

**Q: "Where would you put a nightly PIM import of 200k products?"** Not in an AEM scheduler. App Builder (or an existing integration platform like MuleSoft) pulls from PIM, writes Content Fragments via the Sites OpenAPI in batches, with retries and monitoring. AEM stays lean.

**Q: "How do you add a 'Generate SEO description with AI' button for authors?"** A UI Extensibility extension (App Builder) in the CF editor or Universal Editor that calls an LLM through an App Builder action and writes the field back. No `/libs` overlay.

**Q: "40 markets, 12 languages, global brand. Content model?"** Blueprint per language master, live copies per market, translation via TIF connector with MT for low-priority content, XFs for shared header/footer, CAC for market settings, and a governance model for which components a market may override.

### 11.7 Deep dive (handbook): integration recipes

**A. App Builder project anatomy**

```text
my-app/
  app.config.yaml          # runtime manifest: packages, actions, web assets
  ext.config.yaml          # for extensions (e.g. aem/cf-console-admin/1)
  actions/
    cf-published/index.js  # serverless action (Node.js)
  web-src/                 # React SPA (Spectrum) for UI or extension
  .env                     # local only, never committed
```

```yaml
# app.config.yaml (part)
application:
  runtimeManifest:
    packages:
      mysite:
        license: Apache-2.0
        actions:
          cf-published:
            function: actions/cf-published/index.js
            runtime: nodejs:20
            web: 'no'
            inputs:
              LOG_LEVEL: info
              SEARCH_API_KEY: $SEARCH_API_KEY
            annotations:
              require-adobe-auth: false
```

Commands: `aio app init`, `aio app run` (local), `aio app deploy` (to Workspace: Stage/Production), `aio app logs`.

**B. Subscribing to AEM events (step by step)**

1. In Adobe Developer Console, open the App Builder project → Workspace → Add Event → choose the AEM event provider for your environment (Sites/Assets events).
2. Select event types (e.g. content fragment published, page published, asset processing completed).
3. Choose delivery: **Runtime action** (your `cf-published` action) or **webhook URL**, or **journaling** (pull API, good for reliable batch consumers).
4. Handle retries and idempotency: events can be delivered more than once; store the event ID.
5. Monitor in the Developer Console (debug tracing shows payloads).

**C. UI Extensibility: a button in the Content Fragment console**

```js
// web-src/src/components/ExtensionRegistration.js
import { register } from '@adobe/uix-guest';

export default async function init() {
  const guestConnection = await register({
    id: 'mysite-seo-ext',
    methods: {
      headerMenu: {
        getButtons() {
          return [{
            id: 'generate-seo',
            label: 'Generate SEO',
            icon: 'MagicWand',
            onClick() {
              guestConnection.host.modal.showUrl({ title: 'Generate SEO', url: '/index.html#/seo-modal' });
            },
          }];
        },
      },
    },
  });
}
```

The modal (React) gets the selected fragments from the host, calls an App Builder action (which calls the LLM and the Sites API with a server-to-server token), and writes the field back. Deployed per environment; enabled for authors in Extension Manager.

**D. API Mesh: one GraphQL API for AEM + Commerce**

```json
{
  "meshConfig": {
    "sources": [
      { "name": "AEM", "handler": { "graphql": {
          "endpoint": "https://publish-p123-e456.adobeaemcloud.com/content/_cq_graphql/mysite/endpoint.json" } } },
      { "name": "Commerce", "handler": { "graphql": {
          "endpoint": "https://commerce.example.com/graphql" } } }
    ]
  }
}
```

```bash
aio api-mesh:create mesh.json
aio api-mesh:status
```

Add `additionalResolvers` to join e.g. a product SKU in a CF with live price/stock from Commerce.

**E. MSM in depth**

| Concept | Detail |
| --- | --- |
| Blueprint | Source tree, e.g. `/content/mysite/language-masters/en` |
| Live Copy | Target tree, e.g. `/content/mysite/uk/en` with a live relationship |
| Rollout config | What happens on rollout: Standard (`/libs/msm/wcm/rolloutconfigs/default`), Activate on Blueprint activation, Push on modify, custom configs combining synchronizers |
| Inheritance | Per component; authors can cancel/re-enable inheritance; cancelled = local override kept |
| Rollout scale | On Cloud Service rollouts of large trees run as background jobs; schedule large rollouts |

Good practice: limit live copy depth (language master → market), avoid live copies of live copies unless needed, define which components markets may change (policies/locks), audit overridden content periodically.

**F. Translation in depth**

1. Configure the translation provider (TIF connector cloud config: Smartling, RWS, Lionbridge, XTM, or Microsoft Translator machine translation).
2. Translation rules (`/conf/global/settings/translation/rules/translation_rules.xml`) decide which properties are translated (e.g. `jcr:title`, `text`, but not `fileReference`) and which referenced assets/CFs are included.
3. Create a language copy / translation project; choose "human" or "machine"; send; review returned content in the translation job; approve → content lands in the language copy.
4. Updates: "Update language copies" creates a launch with only changed content to review before promoting.
5. For EDS document authoring, translation is file-based (connectors for SharePoint/Google or a localization service).

**G. Commerce integration settings (CIF)**

- Commerce GraphQL endpoint set per environment via the Cloud Manager variable `COMMERCE_ENDPOINT`, with CIF configuration in `/conf/mysite` (store view, catalog root).
- Product/category pages are templates rendering CIF Core Components with SEO URLs; caching per product page with purge on catalog updates.
- For new builds, evaluate the EDS Commerce storefront with drop-ins first.

**H. Adobe Web SDK on AEM**

- Adobe Tags property with the Web SDK extension; datastream in Data Collection points to Analytics, Target, AEP.
- Map ACDL events to XDM (`web.webPageDetails`, `commerce`, custom field groups) in Tags rules.
- Consent: integrate the client's CMP (OneTrust etc.) with Web SDK consent commands before sending events.

**Expert-level interview answer: "Search across pages, PDFs and products with typo tolerance and facets."** Don't build it on Oak queries. Index content into a search service (Algolia, Coveo, Elastic, or Adobe's own options) using I/O events (publish/unpublish) and a nightly full reindex job in App Builder; index assets' extracted text from microservices; products from Commerce. Front end calls the search service directly from the browser with a search-only key; AEM only renders the search page component.

### References

- [App Builder overview](https://developer.adobe.com/app-builder/docs/overview/)
- [UI Extensibility](https://developer.adobe.com/uix/docs/)
- [AEMaaCS API index (events, OpenAPI)](https://developer.adobe.com/experience-cloud/experience-manager-apis/)
- [Venia CIF reference (GitHub)](https://github.com/adobe/aem-cif-guides-venia)
- [AEMaaCS release notes](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/release-notes/release-notes/release-notes-current)

## Module 12: Migration to AEM as a Cloud Service (deep refresh)

A migration from 6.x (on-prem or AMS) to AEMaaCS is **not an upgrade**: it is a code refactor + content transfer + operating-model change. Adobe's tooling now covers each step, orchestrated in **Cloud Acceleration Manager (CAM)**. Typical Infosys-style programmes run 3-9 months depending on code size, content volume and number of sites.

### 12.1 The journey (phases)

1. **Readiness / assessment**: install the **Best Practices Analyzer (BPA)** package on the source, run the report, upload it to CAM. BPA lists incompatibilities by pattern code (e.g. `OAKI` indexes, `REPL` replication agents, `URS` unsupported repository structure, `LOCP` overlays of `/libs`, `ECU` custom workflow launchers on DAM, `DG` developer guideline violations). CAM estimates effort and complexity.
2. **Plan**: target architecture (Sites/Assets/Forms programs, environments, CDN, networking, IMS/SSO), cutover strategy, content freeze window, team training.
3. **Code refactoring** (using the tools in 12.2), Maven project restructured to the archetype layout, build green in Cloud Manager.
4. **Content transfer** with the **Content Transfer Tool (CTT)**: initial extraction + ingestion to stage/prod, then **top-up** (delta) runs until cutover.
5. **Testing**: functional, performance/load on stage, security, UAT, authoring training (IMS login, new consoles).
6. **Go-live**: final content freeze, last top-up, DNS switch to the Adobe CDN (or BYO CDN origin change), hypercare.
7. **Post go-live**: decommission old infra, adopt cloud-native features (EDS, Universal Editor, OpenAPI).

### 12.2 Tools you must name

| Tool | What it does |
| --- | --- |
| Best Practices Analyzer (BPA) | Scans the 6.x instance for cloud incompatibilities; feeds CAM |
| Cloud Acceleration Manager (CAM) | Web app guiding the whole migration: readiness, code refactoring guidance, migration sets, ingestion monitoring |
| Content Transfer Tool (CTT) | Extracts content (and users/groups, ACLs) from source to a cloud staging container; ingests into target author/publish; supports top-up (delta) and "wipe" vs merge ingestion |
| AEM Modernize Tools | Converts static → editable templates, foundation → Core Components, design dialogs → policies, parsys → responsive grid |
| Repository Modernizer (aio CLI plugin) | Splits mixed packages into `ui.apps` / `ui.content` / `ui.config` per the cloud structure |
| Index Converter | Converts custom Oak index definitions to the cloud naming format |
| Dispatcher Converter | Converts AMS-style dispatcher config to the AEMaaCS format |
| Asset Workflow Migrator | Converts DAM Update Asset customizations to processing profiles / post-processing workflows |
| Bulk Import | Ingest assets from S3/Azure for non-AEM DAM sources |

```bash
aio plugins:install @adobe/aio-cli-plugin-aem-cloud-service-migration
aio aem-migration:repository-modernizer
aio aem-migration:dispatcher-converter
aio aem-migration:index-converter
aio aem-migration:all
```

### 12.3 The usual code refactors (checklist)

- [ ] Split mixed content/code packages; move `/etc` designs and configs to `/conf` and `/apps`.
- [ ] Replace `sling:OsgiConfig` XML with `.cfg.json`; collapse custom run modes to supported ones + env vars/secrets.
- [ ] Service users, ACLs, groups → repoinit.
- [ ] Custom Oak indexes → Lucene, cloud naming (`-custom-n`), remove property indexes.
- [ ] Remove `/libs` overlays; use extension points or UI Extensibility.
- [ ] Replication agents / reverse replication → Sling Content Distribution defaults; user-generated content → external store.
- [ ] Schedulers/long jobs → Sling Jobs on author or App Builder.
- [ ] DAM Update Asset customizations → processing profiles, Asset Compute, post-processing workflows.
- [ ] Disk writes, static file caches → repository/blob/external.
- [ ] Dispatcher → cloud format, flexible mode; CDN rules from the old CDN → `cdn.yaml` where appropriate.
- [ ] LDAP/SAML author login → Adobe IMS federated SSO; plan for group mapping.
- [ ] Outbound calls to IP-restricted systems → Advanced Networking (dedicated egress IP / VPN).
- [ ] Java 8 code → compiles on Java 11/17/21; remove Guava/Commons Collections 3 usage.
- [ ] Static templates/foundation components → editable templates/Core Components (optional but recommended).

### 12.4 Content transfer gotchas

- Size the content: run repository size checks; very large data stores use the **pre-copy** step (AzCopy) to speed up extraction.
- Clean up first: old versions, audit logs, unused assets and DAM renditions (renditions are regenerated by microservices; plan for reprocessing time).
- **Ingestion with "wipe"** replaces target content; later top-ups must be non-wipe. Indexing after ingestion can take hours.
- Users/groups: groups and ACLs migrate with content; author users log in via IMS and are matched to migrated groups. Plan IMS user provisioning (Admin Console, user sync) early.
- Freeze windows: agree a content-freeze policy for the final top-up; author-side work done during top-up windows may need re-entry.
- Publish: ingestion to publish directly, or ingest to author and publish trees (slower).

### 12.5 6.5 → 6.5 LTS (the other migration)

- For clients staying on-prem/AMS: in-place upgrade or new install of 6.5 LTS, moving to Java 17/21.
- Code must compile and run on Java 17/21 and drop APIs removed in LTS; run the analyzer tooling Adobe provides and check bundle imports.
- Forms JEE on LTS: Java 17/21 with JBoss EAP 8.

### 12.6 Interview angle

**Q: "Walk me through how you would migrate a 6.5 AMS client with 20 sites and 3 TB of assets."** Assessment with BPA and CAM → target design (Sites + Assets programs, CDN, IMS, networking) → refactor with Repository Modernizer, Index/Dispatcher converters, repoinit → green pipeline → CTT extraction with pre-copy, ingestion to stage, test, then prod → top-ups → content freeze → DNS cutover → hypercare. Mention risks: asset reprocessing time, IMS onboarding, integrations needing egress IPs, author training.

**Q: "Biggest lessons learned from migrations?"** Start IMS and networking early (they block testing); clean content before CTT; don't lift-and-shift every customization (many are obsolete); budget performance testing for the CDN layer; and run top-ups frequently to keep the final freeze short.

### 12.7 Deep dive (handbook): migration runbook

**A. Content Transfer Tool step by step**

1. In CAM, create a **migration set** (it generates an extraction key and a cloud staging container).
2. Install the CTT package on the source author (and publish if migrating publish separately) via Package Manager.
3. In the source AEM: Operations → Content Migration → Content Transfer → *Create migration set* using the extraction key; select paths (`/content/mysite`, `/content/dam/mysite`, `/conf/mysite`, `/content/cq:tags/mysite`, etc.); choose whether to include versions; groups/users with ACLs are included automatically.
4. Run **Check size** (estimates repository and data store size; ensure disk space on source for extraction).
5. **Extraction**: content goes to the cloud staging container. For very large data stores, enable **pre-copy** (AzCopy from S3/Azure source blob store) to cut hours/days.
6. In CAM: **Ingestion** into target author (and publish) with "wipe" for the first load. Allow the target IP allowlist rule for ingestion where needed.
7. Validate: page counts, asset counts, references, ACLs, CUGs; run the **migration validation** report; reindexing completes in background.
8. **Top-up**: re-run extraction with *overwrite staging container* off and ingest with wipe off — only changed content since last extraction is moved. Repeat weekly, then daily before go-live.
9. Final top-up during content freeze, then go-live.

During ingestion, the target environment is unavailable for authoring; plan ingestion windows with the client. Asset renditions are regenerated on the target (microservices), which takes time proportional to asset volume; you can choose to reprocess later.

**B. Before/after code examples**

OSGi config:

```xml
<!-- BEFORE: /apps/mysite/config.publish.uk/com.mysite.Service.xml -->
<jcr:root jcr:primaryType="sling:OsgiConfig" apiUrl="https://uk.api.com" apiKey="hardcoded"/>
```

```json
// AFTER: /apps/mysite/osgiconfig/config.publish/com.mysite.Service.cfg.json
{ "apiUrl": "$[env:API_URL]", "apiKey": "$[secret:API_KEY]" }
```

Index:

```text
BEFORE: /oak:index/productIndex (type=property, propertyNames=[productId])
AFTER : /oak:index/mysite.productIndex-1-custom-1 (type=lucene, compatVersion=2,
        async=[async,nrt], indexRules/nt:unstructured/properties/productId propertyIndex=true)
```

Replication agent used for a custom "push to external system":

```text
BEFORE: custom transport replication agent posting to https://search.myco.com
AFTER : AEM publish event → Adobe I/O Events → App Builder action → search API
```

DAM workflow:

```text
BEFORE: DAM Update Asset model + custom step "Add watermark" (ImageMagick)
AFTER : Processing profile with a custom Asset Compute worker "watermark" on /content/dam/mysite/public
```

**C. Cutover runbook (T = go-live)**

| When | Activity | Owner |
| --- | --- | --- |
| T-30 days | Prod environment complete; full content load; performance test passed; DNS TTLs lowered to 300 s | Platform lead |
| T-14 days | Authors trained; UAT sign-off; SEO redirect map validated; monitoring dashboards live | Delivery lead |
| T-7 days | Daily top-ups; go/no-go criteria agreed; rollback plan (keep old prod running) | Architect |
| T-2 days | Content freeze starts (emergency changes logged for re-entry) | Business |
| T-1 day | Final top-up; content validation; smoke tests on prod via hosts-file/preview domain | QA |
| T | Switch DNS/CDN origin; purge caches; smoke tests; monitor errors and 404s | Platform lead |
| T+1..14 days | Hypercare; fix 404s via redirects; tune caching; decommission plan for old infra | Whole team |

**D. Risk register (typical)**

| Risk | Mitigation |
| --- | --- |
| IMS/SSO setup delays block testing | Start Admin Console + IdP federation in week 1 |
| Integrations blocked by IP allowlists | Request dedicated egress IP early; share IPs with backend teams |
| Long asset reprocessing | Pre-copy, phased ingestion, reprocess off-peak |
| Custom code incompatible with cloud | BPA early, spike the hardest items first |
| Performance regressions | Load test stage with production-like CDN config |
| Authors resist new UI | Training, quick-reference guides, super-users per market |
| SEO traffic drop after cutover | Redirect map, canonical checks, sitemap, Search Console monitoring |

**E. Estimation approach**

Effort ≈ (BPA findings by category × effort per finding type) + components to refactor + integrations × complexity + content migration iterations + testing (30-40% of build) + contingency 15-20%. Present it as a range with assumptions, and validate with a 2-4 week **pilot** migrating one site end to end.

**Expert-level interview answer: "After go-live, authors see missing assets on some pages. What happened and how do you fix it?"** Likely causes: assets outside the migrated paths (references to `/content/dam/shared`), top-up ran before late uploads, or renditions not yet regenerated. Check reference reports for broken links, run a targeted top-up for missing paths (non-wipe), reprocess assets with failed processing, and add a reference-check step to the cutover checklist.

### References

- [Migration journey: getting started](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/migration-journey/getting-started)
- [Best Practices Analyzer](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/migration-journey/cloud-migration/best-practices-analyzer/overview-best-practices-analyzer)
- [Content Transfer Tool](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/migration-journey/cloud-migration/content-transfer-tool/overview-content-transfer-tool)
- [Cloud Acceleration Manager](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/migration-journey/cloud-acceleration-manager/introduction/getting-started-cam)
- [AEM Modernize Tools (GitHub)](https://github.com/adobe/aem-modernize-tools)
- [Migration CLI plugin (GitHub)](https://github.com/adobe/aio-cli-plugin-aem-cloud-service-migration)
- [Upgrading to AEM 6.5 LTS](https://experienceleague.adobe.com/en/docs/experience-manager-65-lts/content/implementing/deploying/upgrading/upgrade)

## Module 13: Operations, monitoring and AI

On Cloud Service, Adobe runs the platform (patching, scaling, backups, availability); you own code quality, configuration, CDN rules, monitoring of *your* application, and integrations. Since 2024 AI has also become an operational topic: Sites Optimizer fixes site issues, and LLM Optimizer handles visibility in AI answers.

### 13.1 Shared responsibility

| Adobe | You (client / Infosys team) |
| --- | --- |
| Infrastructure, OS, JVM, AEM product updates | Custom code, its performance and security |
| Auto-scaling publish, availability SLA | Cache strategy, CDN TTLs, traffic filter rules |
| Backups (point-in-time restore), DR in region | Content governance, testing restores, release process |
| CDN platform, DDoS baseline protection | WAF rule tuning (with licence), rate limits |
| Platform monitoring and incident response | Application monitoring, alert routing, log analysis |

### 13.2 Logging and monitoring

- **Logs**: AEM `error.log`/`request.log`, Apache/Dispatcher logs, **CDN logs**; download in Cloud Manager or tail with `aio cloudmanager:tail-log <env> author aemerror`.
- **Log forwarding** via the config pipeline to Splunk, Datadog, Elasticsearch/OpenSearch, Azure Blob, HTTPS endpoints, etc.

```yaml
# config/logForwarding.yaml
kind: "LogForwarding"
version: "1"
metadata:
  envTypes: ["prod"]
data:
  splunk:
    default:
      enabled: true
      host: "splunk-hec.example.com"
      token: "${{SPLUNK_TOKEN}}"   # Cloud Manager secret
      index: "aem_prod"
```

- **APM**: Adobe provides application performance monitoring access for Cloud Service environments; many clients also forward logs/metrics to their own Datadog/Dynatrace dashboards.
- **CDN dashboards**: Adobe provides tooling (e.g. ELK/Splunk dashboard templates) to analyse CDN logs, cache hit ratio and traffic filter matches.
- **Alerts**: notifications in Experience Cloud/Cloud Manager for pipeline events, incidents and proactive recommendations; configurable alert rules in the config pipeline (e.g. traffic filter alerts).

### 13.3 Maintenance, backup, environments

- **Maintenance tasks** (version purge, audit log purge) configured with a `mt.yaml` in the config pipeline; Adobe defaults are applied otherwise.
- **Backup/restore**: point-in-time restore of content; self-service restore in Cloud Manager for recent points; Adobe-run DR.
- **Content Copy**: prod → stage/dev for realistic testing.
- **Sandboxes hibernate** after inactivity; production never does.
- **Specialized testing environments** and **additional publish regions** (multi-region publish) for global latency/DR needs.
- **Licensing awareness**: usage is measured (content requests / page views, storage, users). Architects should know that CDN caching and bot filtering directly affect licence consumption.

### 13.4 AI in AEM (what to be able to talk about)

| Capability | What it does |
| --- | --- |
| **AEM Sites Optimizer** | Agentic app that auto-identifies, suggests and (with approval) implements fixes: broken links/backlinks, missing alt text, SEO metadata, Core Web Vitals, accessibility; code fixes arrive as pull requests for developer review |
| **Adobe LLM Optimizer** | Generative Engine Optimization (GEO): measures brand presence in AI answers (ChatGPT, Gemini, Perplexity) and recommends content changes; "Optimize at Edge" serves AI-friendly versions to LLM crawlers at the CDN |
| **Generate Variations** | Creates copy/image variants for pages and A/B tests (Firefly + LLM), usable from Sites and EDS |
| **AI Assistant / agents in AEM** | Natural-language help and actions (create content, find assets, author forms) inside AEM and Experience Cloud; agentic features shipping through 2025-2026 |
| **Assets AI** | Smart tags, AI-generated metadata, Firefly generative edits, content credentials |
| **Forms AI** | Generate and author Adaptive Forms by chat, import from PDF/XDP; Agentic Form Filling (EA, 2026) |

Architect stance for interviews: AI features are accelerators, not architecture. Talk about **governance** (human approval, brand guidelines, content provenance), **data readiness** (structured CFs, good metadata, accessible markup make AI and LLM visibility work), and **measurement** (RUM, experimentation).

### 13.5 Interview angle

**Q: "Production is slow at 9 am every day. How do you investigate on Cloud Service?"** Check CDN cache hit ratio and traffic spikes in CDN logs (bots?), request.log for slow URLs, error.log for query traversal warnings (missing index), APM transaction traces, recent deployments. Fixes: cache headers, traffic filter rules, indexes, query rewrites, and moving heavy work off publish.

**Q: "How do you make the client's content 'AI-ready'?"** Structured content (Content Fragments with clear models), semantic accessible HTML, good metadata and schema.org, fast pages (EDS), an LLM Optimizer / GEO practice, and permissions on what AI crawlers may access (robots, edge rules).

### 13.6 Deep dive (handbook): running AEM in production

**A. Log types and what to look for**

| Log | Where | Look for |
| --- | --- | --- |
| `aemerror` | Author/publish | Exceptions, query traversal warnings, OSGi bundle failures, slow job warnings |
| `aemrequest` / `aemaccess` | Author/publish | Slow requests (duration), 4xx/5xx by URL, request rate |
| `httpderror` / `httpdaccess` | Dispatcher (Apache) | Filter denials, cache misses, upstream errors |
| `aemdispatcher` | Dispatcher | Cache decisions, invalidations, filter rule matches |
| CDN log | Adobe CDN | Per-request JSON: `cache` (HIT/MISS/PASS), `status`, `ttfb`, `url`, `host`, `cli_ip`, `cli_country`, `pop`, `rules` (traffic filter matches) |

Example Splunk searches over forwarded CDN logs:

```text
index=aem_prod sourcetype=cdn | stats count by cache            # cache hit ratio
index=aem_prod sourcetype=cdn status>=500 | top url               # failing URLs
index=aem_prod sourcetype=cdn rules=* | stats count by rules, cli_ip   # traffic filter matches
index=aem_prod sourcetype=cdn | timechart span=1m avg(ttfb)       # latency trend
```

**B. Incident runbook (P1: site down or 5xx spike)**

1. Confirm scope: CDN logs (all URLs or some? all regions?), Cloud Manager environment status, Adobe status page.
2. Recent changes? Last pipeline run, config pipeline (CDN rules!), environment variable changes, content publishes (huge tree?).
3. If caused by your release: re-run the pipeline with the previous Git tag (there is no "rollback button" for code; redeploy the last good commit). If caused by CDN config: redeploy previous `cdn.yaml`.
4. If platform-side: open a P1 with Adobe Support (Admin Console → Support) with program/env IDs, timestamps, sample request IDs (`x-request-id` / CDN request ID).
5. Mitigate: serve stale (stale-if-error headers help), block abusive traffic with a traffic filter rule, maintenance page via CDN error pages.
6. Post-incident review: root cause, monitoring gap, test gap.

**C. Performance test plan for stage**

- Stage is sized like production, so test there, scheduled with Adobe notice for big tests.
- Test through the CDN (real hostnames) and also cache-busting scenarios (cold cache after deploy).
- Model: 80% cached anonymous pages, 15% search/API, 5% form submits; ramp to 2× expected peak.
- Pass criteria: TTFB p95 at origin, error rate < 0.1%, CDN hit ratio > 90%, author responsiveness during publish storms.
- Include author tests (bulk publish, MSM rollout, asset upload) — author slowness is a common go-live complaint.

**D. SLOs you should propose to a client**

| SLO | Example target |
| --- | --- |
| Availability (publish, measured at CDN) | 99.9% monthly (contractual SLA from Adobe depends on licence) |
| LCP p75 (mobile, RUM) | < 2.5 s |
| INP p75 | < 200 ms |
| Publish-to-live latency | < 5 min including CDN TTL |
| Prod deployment frequency | Weekly, zero-downtime |
| Mean time to recover (code issue) | < 1 h via redeploy of last good tag |

**E. AI governance checklist (for GenAI features in AEM)**

- [ ] Define which AI features are enabled per role (Admin Console / product profiles).
- [ ] Human review before publish for generated copy and images.
- [ ] Brand voice and terminology rules supplied to generation features.
- [ ] Legal review of image generation usage (Firefly is trained for commercial safety; still check client policy).
- [ ] Content credentials retained on generated assets.
- [ ] No confidential/PII content pasted into prompts unless approved.
- [ ] Measure outcomes: experiments for generated variations, Sites Optimizer fixes tracked.

**Expert-level interview answer: "How do you know your AEM platform is healthy without logging into it?"** Dashboards over forwarded logs (CDN hit ratio, 5xx rate, TTFB p95, traffic filter hits), RUM Core Web Vitals, synthetic checks on key journeys every few minutes, pipeline success rate, Adobe incident notifications routed to the team's on-call tool, and weekly review of Sites Optimizer findings and Cloud Manager recommendations.

### References

- [Logging for AEMaaCS](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/implementing/developing/logging)
- [Log forwarding](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/implementing/developing/log-forwarding)
- [Config pipeline](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/implementing/using-cloud-manager/config-pipeline)
- [Sites Optimizer (product page)](https://business.adobe.com/products/experience-manager/sites/optimizer.html)
- [Sites Optimizer release notes](https://experienceleague.adobe.com/en/docs/experience-manager-sites-optimizer/content/documentation/release-notes)
- [Make your AEM content visible to AI](https://experienceleague.adobe.com/en/perspectives/make-your-aem-content-visible-to-ai-a-practitioner-guide)
- [Agentic AI in Adobe applications](https://experienceleague.adobe.com/en/docs/core-services/interface/features/agentic-ai)
- [LLM Optimizer: Optimize at Edge with AEM managed CDN](https://experienceleague.adobe.com/en/docs/llm-optimizer/using/resources/optimize-at-edge/aemcs-managed-cdn)

## Module 14: Architect interview rehearsal

Answer every design question in the same order: **clarify requirements → choose platform (AEMaaCS / EDS / headless / 6.5 LTS) and say why → content model → integration → delivery and caching → security → DevOps → risks**. Interviewers at client side want trade-offs, not feature lists.

### 14.1 Rapid-fire questions (answer in 2-3 sentences each)

**Platform and architecture**

1. Difference between AMS, on-prem and AEMaaCS? → Module 1.2.
2. What is immutable in AEMaaCS and why? → `/apps`, `/libs`; reproducible, scalable pods.
3. What is the Preview tier? → A publish-like tier to review content before go-live; authors "Publish to Preview".
4. How is content replicated? → Sling Content Distribution via Adobe pipeline; no agents.
5. Why can't you use custom run modes? → Only author/publish + dev/stage/prod; use env vars, secrets, CAC.
6. What happens to your code when Adobe releases an update? → Rebuilt with the new AEM image and redeployed by rolling deploy; your build must pass.
7. What is a golden master? → The reference publish repository from which new publish pods are created.
8. When would you still recommend 6.5 LTS? → Data residency, air-gapped, JEE Forms, contractual constraints.

**Development**

9. How do you create service users? → Repoinit + ServiceUserMapper amended config.
10. Where do OSGi configs live and in what format? → `ui.config`, `.cfg.json`, run-mode folders.
11. How do you add a custom Oak index? → Lucene, `-custom-n` naming, deployed via pipeline.
12. How do you run a job once in a cluster? → Sling Jobs; scheduler with `runOn=SINGLE` only to enqueue.
13. How do you extend a Core Component? → Proxy component + Sling Model delegation via `@Via(type = ResourceSuperType.class)`.
14. What is RDE? → A cloud dev environment that you deploy to in seconds with the `aio aem:rde` commands.
15. What does the AEM analyser plugin do? → Runs cloud build checks locally (bundles, repoinit, package types).
16. Which Java version? → Build 21 (or 17/11) via `.cloudmanager/java-version`; runtime Java 21, moving to 25.

**Front end, EDS, headless**

17. What is EDS and how is it different from Dispatcher delivery? → Module 6.
18. Three EDS authoring options? → Docs/SharePoint, Document Authoring, Universal Editor (AEM).
19. What is a block? → A JS/CSS unit decorating a content table/component; `decorate(block)`.
20. Eager/lazy/delayed? → EDS loading phases protecting LCP.
21. Universal Editor vs SPA Editor? → UE is framework-agnostic and the recommended path; SPA Editor deprecated for new projects since 2025.1.0.
22. Why persisted queries? → GET + CDN caching + security (no arbitrary queries).
23. What is the front-end pipeline? → Deploys only theme CSS/JS in minutes.

**Delivery, security, ops**

24. How do you configure WAF/rate limiting? → Traffic filter rules in `cdn.yaml`, config pipeline.
25. How do you route `/shop` to another backend? → CDN origin selectors.
26. How do authors log in? → Adobe IMS with federated SSO.
27. How do you call an on-prem API that allowlists IPs? → Advanced Networking: dedicated egress IP or VPN.
28. How do you send logs to Splunk? → Log forwarding config via config pipeline.
29. How do you keep secrets? → Cloud Manager secret env vars.

**Assets, Forms, integrations, migration**

30. How are renditions generated? → Asset microservices; custom via Asset Compute.
31. What is Content Hub? → Approved-asset self-service portal for brand-safe distribution.
32. Core Components vs foundation adaptive forms? → Core: modern, accessible, themeable, headless-compatible; foundation is legacy.
33. How do you generate 100k PDFs a night? → Communications APIs batch, XDP templates, data from source systems.
34. Where should integration logic live? → App Builder / integration platform, event-driven via I/O Events.
35. Steps of a migration? → BPA → CAM → refactor → CTT + top-ups → test → cutover.
36. Name four refactoring tools. → Repository Modernizer, Index Converter, Dispatcher Converter, AEM Modernize Tools.

### 14.2 Scenario 1: "Design a global multi-brand marketing platform (5 brands, 30 markets, 15 languages)"

- **Platform**: AEMaaCS Sites (one program, or one per brand group if teams/release cycles differ) + EDS with Universal Editor authoring for brand sites; shared AEM Assets program as central DAM.
- **Content model**: blueprint per brand/language master, live copies per market; shared XFs for header/footer; CFs for reusable structured content (offers, product teasers); CAC per brand/market.
- **Translation**: TIF connector to the client's LSP; machine translation for low-risk content; translation of CFs with references.
- **Delivery**: EDS at edge, Adobe CDN with origin selectors for any classic sections; Lighthouse 90+ targets; Web SDK delayed loading.
- **Personalization**: Target/AJO with AEP audiences; EDS experimentation for simple A/B tests.
- **Governance**: component library and block library per brand with shared base; IMS groups per brand/market; workflows for legal approval in regulated markets.
- **DevOps**: one Git repo per EDS site (or monorepo with branches), Cloud Manager for AEM code, config pipeline for CDN and WAF.
- **Risks**: authoring complexity of MSM at scale, translation cost, brand divergence; mitigate with governance board and design system.

### 14.3 Scenario 2: "Bank wants secure customer onboarding with forms, e-sign and CRM"

- Adaptive Forms Core Components (or Forms on EDS for public pre-qualification), prefilled via FDM from CRM, save-and-resume, Acrobat Sign, submission to the bank's API gateway (not stored in AEM), Document of Record generated by Communications APIs and archived in ECM.
- Security: publish behind WAF rules, OIDC login for returning customers, data encryption in transit, no PII in AEM logs, VPN/dedicated egress to core banking.
- Compliance: audit trail in the system of record; accessibility (WCAG 2.2) of Core Components.

### 14.4 Scenario 3: "Headless commerce site with Next.js and AEM content"

- AEM CFs for editorial and product-enrichment content, persisted GraphQL/OpenAPI delivery cached at the CDN; Adobe Commerce via API Mesh (App Builder) to unify content + catalog; Next.js on Vercel/Azure with ISR; Universal Editor instrumentation for authors' in-context editing; Dynamic Media with OpenAPI for images.
- Alternative to propose: Commerce Storefront on EDS with drop-ins, if the client values performance and lower front-end cost over a custom React stack.

### 14.5 Scenario 4: "Migrate 6.5 AMS with heavy customizations, go live in 6 months"

- Week 0-3: BPA/CAM, target design, IMS and networking requests started immediately.
- Week 3-12: refactor with tools; build green; Dispatcher converted; integrations moved to egress IP or App Builder.
- Week 8-16: CTT initial migration to stage, performance tests, UAT; author training.
- Week 16-24: prod ingestion, weekly top-ups, freeze, cutover, hypercare.
- Push back on scope: don't modernize templates/components in the same release unless needed; plan EDS adoption as phase 2.

### 14.6 Questions you should ask the interviewer

- Is the target AEMaaCS, EDS, headless, or 6.5 LTS, and is there already an Adobe licence decision (Sites, Assets, Forms, Dynamic Media, WAF)?
- Who authors content and how technical are they?
- What integrations (CRM, PIM, Commerce, identity) and what non-functional targets (Core Web Vitals, availability, data residency)?
- How mature are their DevOps and their governance today?

### 14.7 Final revision checklist (night before)

- [ ] I can draw the AEMaaCS architecture (author, publish, preview, CDN, Dispatcher, blob store, Cloud Manager).
- [ ] I can explain EDS end to end and when not to use it.
- [ ] I can write a repoinit script, a `.cfg.json`, a Sling Model, a Sling Job, and an EDS block from memory.
- [ ] I can list pipeline types and quality gates.
- [ ] I can explain `cdn.yaml` (traffic filters, WAF, redirects, origin selectors).
- [ ] I can walk through a migration with the tool names.
- [ ] I can talk about Forms Core Components, headless forms, EDS forms and Communications APIs.
- [ ] I have one or two stories from my own migration work (problem, decision, outcome).

### 14.8 Deep technical questions (with model answers)

**Q: How is `/apps` immutability enforced technically?** A composite node store mounts `/apps` and `/libs` from a read-only segment store baked into the image; writes fail at the repository level (2.6).

**Q: What runs when a publish pod starts?** Created from the golden master, starts AEM with the current image, repoinit runs, it subscribes to the distribution queue and replays pending items, passes health checks, then receives traffic.

**Q: Why must content changes be backward compatible during deployments?** Old and new code run side by side against the same author repository during a rolling deployment.

**Q: Difference between `Cache-Control` and `Surrogate-Control`?** Surrogate applies only to the CDN and is stripped; Cache-Control applies to browsers (and CDN if no surrogate).

**Q: What does `statfileslevel` do and how would you set it for 20 language sites?** It sets the depth of `.stat` files so invalidation only affects that subtree; set it to the language level (e.g. 3 for `/content/brand/country/lang`) so publishing in one site doesn't flush the others.

**Q: Sling Model injection: `@ValueMapValue` vs `@Inject`?** Use specific injectors (`@ValueMapValue`, `@OSGiService`, `@ScriptVariable`, `@Self`) for clarity and performance; generic `@Inject` tries all injectors.

**Q: How do you make a Sling Model cacheable per request?** `@Model(cache = true)` caches the adaptation result for the same adaptable within a request.

**Q: Explain `ResourceSuperType` delegation.** `@Self @Via(type = ResourceSuperType.class)` adapts the request as if it were the super type, giving you the Core Component's model to wrap.

**Q: When does Oak traverse, and how do you detect it?** When no index matches the query constraints; logs a traversal warning, and the Developer Console's query explain shows `traverse`. Fix with a Lucene index rule including the property and node type.

**Q: How do you handle 10,000 vanity redirects?** Not with `sling:vanityPath` on pages; use `cdn.yaml` redirects or an Apache `RewriteMap` (or EDS redirects sheet), managed by a business-friendly source.

**Q: How do GraphQL persisted queries get cached?** They are GET requests with a stable URL; Dispatcher and CDN cache them using the cache headers stored with the persisted query; publishing content invalidates Dispatcher, CDN TTLs bound staleness.

**Q: What's the difference between the Preview tier and author preview mode?** Preview tier is a separate publish-like service with its own URL and Dispatcher/CDN; author preview mode only renders in the editor for the logged-in author.

**Q: EDS: what are the code bus, content bus and media bus?** Code from GitHub, converted content, and content-addressed media, each cached at the edge (6.8).

**Q: How do you secure a preview site on EDS?** Site authentication with tokens for preview and/or live; or restrict at your CDN.

**Q: What would you never do on AEMaaCS?** Write to `/apps` at runtime, overlay `/libs`, keep state on local disk, run long synchronous requests, hard-code secrets, rely on a specific publish instance, or process binaries in the JVM.

### 14.9 Whiteboard exercise: draw this in 5 minutes

"Draw the architecture for a global site on AEMaaCS with EDS front end, AEM authoring, Commerce, search and analytics."

What to draw (top to bottom):

1. **Users** → **DNS** → **Adobe CDN (Fastly)** with traffic filters/WAF, origin selectors.
2. CDN origins: **EDS** (pages), **AEM publish/Dispatcher** (APIs, legacy paths, forms submit), **Commerce** (GraphQL via API Mesh), **Dynamic Media with OpenAPI** (images).
3. **AEM author** (Universal Editor, MSM, translation, workflows) → publishes to EDS (content bus) and to publish (SCD).
4. **AEM Assets** program shared, Content Hub for marketers.
5. **App Builder**: I/O events → search indexer (Algolia/Elastic), PIM sync, UI extensions.
6. **Adobe Tags/Web SDK** → Analytics/Target/AEP; RUM for CWV.
7. **DevOps**: GitHub (EDS code, AEM code) → Code Sync / Cloud Manager pipelines; config pipeline for CDN; logs forwarded to Splunk.
8. **Identity**: IMS (authors) with client IdP federation; OIDC for customers.

Narrate the request flow for one anonymous page view and one author publish; then name 3 risks (cache invalidation across EDS and AEM, authoring complexity, third-party script impact on LCP).

### 14.10 Behavioural questions (prepare STAR stories)

| Question | What they look for |
| --- | --- |
| Tell me about a migration you led and what went wrong | Honesty, lessons, risk management |
| A client insists on a design you think is wrong | Influence with data, trade-off documents, escalation path |
| You inherited a slow, unstable AEM platform | Diagnosis method, prioritization, measurable outcome |
| How do you keep your AEM knowledge current? | Release notes, Adobe community, hands-on sandbox/RDE, certifications (AEM Sites Architect / Developer Expert) |
| How do you guide junior developers? | Code reviews, standards, pairing, reusable patterns |

For your gap since 2020, say it confidently: you led/participated in a migration, you have refreshed hands-on on the current stack (EDS, Universal Editor, Cloud Manager pipelines, cdn.yaml), and you bring architecture judgment that tools don't replace.

### Note on sources

Key dates and recent changes (6.5 LTS, Java versions, SPA Editor deprecation, OpenAPI delivery, EDS authoring, Forms 2026 releases, Sites/LLM Optimizer) were checked against Adobe search results in Nov 2026; the rest is from experience and may lag recent releases. Some reference URLs could not be opened from this environment, so if a link 404s, search its title on Experience League. Adobe renames and reshuffles features often (especially AI products and EDS authoring defaults), so skim the latest AEMaaCS release notes the week of your interview.
