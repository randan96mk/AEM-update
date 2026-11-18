# AEM Architect Refresher 2020 → 2026

As of Nov 18, 2026

## How to use this guide

The single biggest change since 2020: AEM is now a cloud-native, continuously updated product (AEM as a Cloud Service, "AEMaaCS"), with Edge Delivery Services and Universal Editor as the new default for building sites. On-Prem and Adobe Managed Services (AMS) still exist, now on AEM 6.5 LTS.

Each module follows the same pattern: **2020 (what you knew)** → **Now (2026)** → **Architect view** → **Code / config sample** → **Interview angle** → **References**. Read Module 1 and the cheat sheet below first; then go module by module. Module 14 is a rehearsal set of interview questions.

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

### Note on sources

Key dates and recent changes (6.5 LTS, Java versions, SPA Editor deprecation, OpenAPI delivery, EDS authoring, Forms 2026 releases, Sites/LLM Optimizer) were checked against Adobe search results in Nov 2026; the rest is from experience and may lag recent releases. Some reference URLs could not be opened from this environment, so if a link 404s, search its title on Experience League. Adobe renames and reshuffles features often (especially AI products and EDS authoring defaults), so skim the latest AEMaaCS release notes the week of your interview.
