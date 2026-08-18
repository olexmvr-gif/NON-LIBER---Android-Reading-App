# NON·LIBER v0.6.0.0.5 - Advanced Technical Map

> **Audience:** advanced users, testers, contributors, and developers who want to understand not only what NON·LIBER does, but how the application is assembled and where its major behaviours live in code.
>
> **Audited build:** Android `versionCode 194`, `versionName 0.6.0.0.5-debug`  
> **Audited source:** `/Users/Development/NON-LIBER v0.6.0.0.5`  
> **Audited commit:** `79d3685`  
> **Evidence date:** 2026-08-18

---

## Contents

1. [What NON·LIBER actually is](#1-what-nonliber-actually-is)
2. [The shortest useful architecture model](#2-the-shortest-useful-architecture-model)
3. [Languages and technology stack](#3-languages-and-technology-stack)
4. [Repository map](#4-repository-map)
5. [How the Android application boots](#5-how-the-android-application-boots)
6. [Native Android vs Web application ownership](#6-native-android-vs-web-application-ownership)
7. [The four major user surfaces](#7-the-four-major-user-surfaces)
8. [Publication and format architecture](#8-publication-and-format-architecture)
9. [Reader architecture](#9-reader-architecture)
10. [Pagination, locations, and page count](#10-pagination-locations-and-page-count)
11. [Page Flip architecture](#11-page-flip-architecture)
12. [PDF architecture](#12-pdf-architecture)
13. [Markdown, Journals, Documents, and Scriptum](#13-markdown-journals-documents-and-scriptum)
14. [Annotation and marking architecture](#14-annotation-and-marking-architecture)
15. [Search architecture](#15-search-architecture)
16. [Typography and theme architecture](#16-typography-and-theme-architecture)
17. [Gesture and input ownership](#17-gesture-and-input-ownership)
18. [Storage architecture](#18-storage-architecture)
19. [Publication identity and metadata ownership](#19-publication-identity-and-metadata-ownership)
20. [Backup, restore, and reimport](#20-backup-restore-and-reimport)
21. [Android bridges](#21-android-bridges)
22. [Performance model](#22-performance-model)
23. [Build and packaging](#23-build-and-packaging)
24. [Test architecture](#24-test-architecture)
25. [What is strong today](#25-what-is-strong-today)
26. [What is still risky or incomplete](#26-what-is-still-risky-or-incomplete)
27. [Dormant and placeholder systems](#27-dormant-and-placeholder-systems)
28. [Developer navigation: where to look first](#28-developer-navigation-where-to-look-first)
29. [End-to-end flows](#29-end-to-end-flows)
30. [Practical contributor rules](#30-practical-contributor-rules)
31. [Compact architecture glossary](#31-compact-architecture-glossary)

---

# 1. What NON·LIBER actually is

NON·LIBER is a **local-first Android reading and writing application**.

It is not architected as a conventional all-native Android UI. Instead, the product is deliberately split into two major layers:

- a **Kotlin Android shell** that owns device integration;
- a large **JavaScript/HTML/CSS application** running inside Android WebView.

This split is important because most of the product - Library, Artefact, Reader, Scriptum, publication parsing, state resolution, annotations, typography, search, and much of persistence - is implemented in JavaScript.

Kotlin is comparatively narrow. It gives the web application access to things the browser layer cannot safely or reliably own by itself on Android: Storage Access Framework folders and files, window brightness, haptics, volume keys, immersive system bars, native Back routing, external links, and backup/export file streams.

The result is best understood as:

```text
Android shell
    |
    v
secure local WebView origin
    |
    v
NON·LIBER JavaScript application
    |
    +-- Library
    +-- Artefact
    +-- Reader
    +-- Scriptum
    |
    +-- publication engines
    +-- IndexedDB/local state
    +-- search / marks / typography / backup logic
```

The app is intentionally offline-first. Runtime assets and important third-party libraries are bundled locally. The audited Android manifest does not declare ordinary Internet permission for the application runtime.

---

# 2. The shortest useful architecture model

For an advanced user, the product can be reduced to four ideas:

```text
Library   = find and organise publications
Artefact  = compose and curate views over the same library
Reader    = render, navigate, search, mark, and style content
Scriptum  = write and manage Markdown/Journals/Documents/Notes/Vaults
```

These are **not four independent databases**.

They sit over shared publication records and shared user-owned metadata. Changing a title, author, collection, reading state, mark, cover, or classification in one surface affects the same underlying record that other surfaces later read.

This shared-data rule is one of the most important architectural laws in NON·LIBER.

---

# 3. Languages and technology stack

## 3.1 Main languages

| Language / technology | Main responsibility |
|---|---|
| **Kotlin** | Android Activity, WebView configuration, Storage Access Framework, native bridges, lifecycle, Back, volume keys, brightness, haptics, system UI |
| **JavaScript ES modules** | Main application logic, UI state, publication engines, search, annotations, Reader orchestration, Library/Artefact/Scriptum behaviour, persistence logic |
| **HTML** | Application shell and rendered publication/editor structures |
| **CSS** | UI components, themes, Reader styling, PDF surfaces, typography injection targets |
| **Gradle / Kotlin DSL** | Android build, SDK/ABI/version/dependency configuration |
| **IndexedDB** | Durable local application database inside WebView |
| **localStorage / sessionStorage** | Lightweight preferences, prepaint state, recovery/transient state |
| **Android SharedPreferences** | Device-local configuration such as selected folders and native shell state |

## 3.2 Major bundled runtime libraries

| Dependency | Purpose |
|---|---|
| **PDF.js 6.1.200 legacy build** | Local PDF parsing, rendering, text extraction, outline/links, search foundation |
| **StPageFlip 2.0.7** | Page-curl visual/gesture engine for eligible reflowable page turns |
| **Atomic Editor island** | Raw Markdown editing candidate path |
| **AndroidX WebKit** | Secure WebView asset loading and compatibility support |
| **AndroidX AppCompat / Activity** | Android Activity/window lifecycle foundation |

There is no active React, Vue, Svelte, Tauri, Rust front end, or server runtime in the audited product.

The UI is largely imperative DOM code using small internal helpers rather than a modern component framework.

---

# 4. Repository map

A useful high-level view of the source tree is:

```text
NON-LIBER v0.6.0.0.5/
|
+-- index.html
+-- package.json
|
+-- src/
|   +-- main.js
|   +-- settings.js
|   |
|   +-- startup/
|   |   +-- boot.js
|   |   +-- themePrepaint.js
|   |   +-- routeRecovery.js
|   |   +-- deferredFeatures.js
|   |
|   +-- ui/
|   |   +-- library.js
|   |   +-- artefact.js
|   |   +-- scriptum.js
|   |   +-- reader.js
|   |   +-- reader/
|   |   +-- notesEditor.js
|   |   +-- shared/
|   |
|   +-- engine/
|   |   +-- publication/
|   |   +-- LiteEpubEngine.js
|   |   +-- epub/
|   |   +-- fb2.js
|   |   +-- kindle.js
|   |   +-- markdown/
|   |   +-- pdf/
|   |   +-- pageFlip/
|   |   +-- progress/
|   |   +-- rcvp/
|   |   +-- fonts.js
|   |   +-- theme.js
|   |   +-- annotate.js
|   |
|   +-- storage/
|   |   +-- db.js
|   |   +-- catalog.js
|   |   +-- identity.js
|   |   +-- fileIdentity.js
|   |   +-- exportPackage.js
|   |   +-- importPackage.js
|   |   +-- restoreApply.js
|   |   +-- stateRegistry.js
|   |   +-- typographyProfiles.js
|   |   +-- readerCategorySettings.js
|   |   +-- scriptum*.js
|   |   +-- vault*.js
|   |   +-- notesRepository.js
|   |
|   +-- editor/atomic/
|   +-- domain/
|   +-- testing/
|
+-- styles/
+-- assets/
+-- third_party/
|   +-- StPageFlip/
|   +-- atomic-editor/
|
+-- android/
|   +-- app/build.gradle.kts
|   +-- app/src/main/AndroidManifest.xml
|   +-- app/src/main/java/.../MainActivity.kt
|   +-- app/src/main/assets/web/
|   +-- runtime-assets.json
|   +-- build-apk.bat
|
+-- tools/
+-- modules/ai-narration/
+-- docs/
+-- evidence/
```

The important distinction is that `android/app/src/main/assets/web/` is a **generated runtime mirror**, not the canonical place to edit the web application.

Canonical web source lives at the root in `index.html`, `src/`, `styles/`, `third_party/`, and `assets/`.

---

# 5. How the Android application boots

The audited startup path is approximately:

```text
Android process
  -> MainActivity.onCreate
  -> configure WebView + native bridges
  -> load WebViewAssetLoader local HTTPS origin
  -> load index.html + src/main.js
  -> native-launch phase
  -> web-shell phase
  -> theme prepaint
  -> open IndexedDB
  -> hydrate/migrate settings
  -> validate startup route
  -> render primary surface
  -> mark interactive
  -> defer large modules and progress-index work
```

## Why this matters

NON·LIBER avoids doing all expensive work before the first usable screen.

The startup code deliberately separates:

- early shell rendering;
- theme application;
- persistent storage opening;
- migration;
- route recovery;
- primary screen rendering;
- lower-priority deferred work.

`src/startup/boot.js` acts as a phase controller. Startup is expected to progress through known states rather than allowing arbitrary modules to race one another.

`themePrepaint.js` uses a lightweight locally stored token so the app can paint close to the expected theme before the complete settings database is ready. This reduces white/incorrect-theme startup flashes.

`routeRecovery.js` is deliberately conservative. A Reader restore only happens if the referenced publication and source bytes still exist. Invalid restore state falls back instead of leaving the application stranded in a broken Reader.

---

# 6. Native Android vs Web application ownership

The easiest way to understand the app is to ask: **who owns this behaviour?**

## Kotlin / Android owns

- Android Activity lifecycle;
- WebView policy;
- local app-assets origin;
- Storage Access Framework file and folder acquisition;
- persisted URI grants;
- native brightness control;
- haptics;
- hardware volume-button interception;
- Android Back dispatch into the web state machine;
- system-bar / immersive mode integration;
- external URL handoff;
- backup/export streams to user-selected folders;
- some crash/report writing;
- Android-specific callbacks and memory signals.

## JavaScript owns

- Library query/filter/sort/group logic;
- Artefact composition;
- Reader state and navigation;
- publication parsing and adapters;
- pagination;
- page-count state;
- Page Flip eligibility/commit/fallback;
- text search;
- marks and annotations;
- typography/profile resolution;
- themes;
- Scriptum/Vault/Notes logic;
- IndexedDB persistence;
- backup package structure and restore merging;
- return-origin state;
- most app UI.

This is why calling NON·LIBER a "WebView app" is technically true but incomplete. The Android shell is narrow, while the JavaScript application is large and behaves more like an embedded local software platform than a thin website wrapper.

---

# 7. The four major user surfaces

## 7.1 Library

Library is the conventional publication-management surface.

Internally it:

1. loads publication records;
2. loads collection and shelf catalogues;
3. obtains annotation statistics;
4. applies eligibility rules;
5. applies search/filter/sort;
6. chooses flat or grouped presentation;
7. remembers lightweight view preferences;
8. captures return state before opening a Reader.

Important implementation areas:

```text
src/ui/library.js
src/storage/catalog.js
src/storage/importPipeline.js
src/storage/identity.js
src/storage/sidecar.js
src/domain/publicationQuery.js
```

Library is not expected to own every mutation directly. Shared domain/catalog code exists so metadata changes made from Library, Artefact, or Scriptum converge on the same record rules.

## 7.2 Artefact

Artefact is a **composition layer over the shared library**.

It does not duplicate publications.

It lets the user build a more ceremonial/curated surface using:

- built-in sections;
- custom book rows;
- entity galleries;
- authors;
- genres;
- collections;
- shelves;
- series;
- Smart Intersection-style filtering;
- custom layout/profile preferences.

Important implementation areas:

```text
src/ui/artefact.js
src/storage/artefactRows.js
src/domain/smartIntersection.js
settings/artefactPrefs
settings/artefactEntityMeta
```

A useful architectural rule is:

> Artefact stores **presentation state**, not duplicate book content.

Entity-room descriptions or presentation metadata can exist separately from publication metadata so a curated author/entity page does not have to rewrite embedded publication information.

## 7.3 Reader

Reader is the central integration surface.

It coordinates:

- publication engine selection;
- current logical location;
- pages/scroll/seamless modes;
- TOC and internal links;
- search;
- bookmarks;
- annotations;
- typography;
- themes;
- progress;
- Page Flip and other transitions;
- Markdown editing;
- PDF surface handoff;
- gesture arbitration;
- persistence;
- lifecycle recovery.

Its main orchestration file, `src/ui/reader.js`, is extremely large - nearly 12,000 lines in the audited tree - and remains one of the major complexity centres in the application.

Smaller controllers exist under `src/ui/reader/`, but many still depend on shared Reader closure/state ownership.

## 7.4 Scriptum

Scriptum is the writing/document side of NON·LIBER.

It works with:

- Markdown publications;
- Journals;
- Documents;
- Notes;
- Vaults;
- writing-specific state;
- Smart rows/layouts;
- editing and recovery.

Scriptum is intentionally built on the same publication substrate rather than becoming a separate file-manager database.

Important areas include:

```text
src/ui/scriptum.js
src/ui/scriptumDetails.js
src/storage/scriptum*.js
src/storage/vault*.js
src/storage/notesRepository.js
src/editor/atomic/
src/engine/markdown/
```

---

# 8. Publication and format architecture

NON·LIBER does not treat every imported file as if it has identical capabilities.

An engine/capability layer decides what the file can actually do.

## Active user-facing formats

| Format | Core engine behaviour |
|---|---|
| **EPUB** | Reflowable publication path with Pages/Scroll/Seamless, search, marks, bookmarks, typography |
| **FB2 / FB2.zip** | Adapted into shared reflowable reading model |
| **MOBI / PRC** | DRM-free legacy Kindle parsing/conversion path |
| **AZW / AZW3** | DRM-free legacy Kindle parsing/conversion path |
| **Markdown** | Safe rendered document + editing paths |
| **PDF** | Separate fixed-layout PDF.js architecture |

The important design choice is **capability gating**.

UI controls are expected to ask whether the current publication supports a behaviour rather than blindly exposing every Reader function for every format.

Examples:

- PDF does not pretend to support EPUB typography;
- PDF does not use reflow Page Flip;
- Markdown exposes editing paths that EPUB does not;
- image-only PDF search cannot work without OCR;
- DRM Kindle content is not treated as readable merely because the extension is recognised.

---

# 9. Reader architecture

The Reader pipeline can be summarised as:

```text
book record + source bytes
  -> EngineRegistry / format adapter
  -> publication manifest + capabilities + TOC
  -> reflow paginator OR PdfReaderSurface
  -> logical location + progress model
  -> Reader controls / input / overlays
  -> persisted book state + category/profile settings
```

## Key principle: logical location beats page number

A page number is not a stable identity in a reflowable book.

Changing:

- font size;
- font family;
- line height;
- margins;
- viewport size;
- orientation;
- publisher CSS handling;

can change page boundaries.

Therefore Reader navigation relies on logical anchors such as chapter/spine identity plus offsets/anchors, while page numbers are derived from the current layout.

This is why NON·LIBER can recalculate page counts without intentionally losing the user's conceptual reading location.

---

# 10. Pagination, locations, and page count

Reflowable pagination is layout work, not a simple arithmetic count.

The paginator must:

1. load chapter/spine HTML;
2. resolve local publication resources;
3. apply publisher CSS plus safe NON·LIBER overrides;
4. inject theme/typography rules;
5. measure content in the current viewport;
6. create page/column boundaries;
7. map logical offsets to current pages.

Important areas:

```text
src/engine/paginator.js
src/engine/progress/
src/engine/PageEstimator or related estimator modules
src/ui/reader.js
```

## Why totals can move

The app distinguishes between:

- an early estimate or partial map;
- progressively improved page information;
- a stable exact count for the current layout.

This is intentional. It allows the Reader to become usable before the entire book has been completely measured.

The cost is that page totals can temporarily change after opening a book or after major layout changes.

This subsystem is considered functional and tested, but page-count convergence remains an area where broader content/device evidence is valuable.

---

# 11. Page Flip architecture

Page Flip is not allowed to become the navigation authority.

That is a crucial design decision.

The Reader still owns:

- whether Page Flip is eligible;
- current page identity;
- target page resolution;
- accept/cancel;
- final navigation commit;
- fallback behaviour.

StPageFlip is used as a controlled visual/gesture engine.

A simplified model is:

```text
Reader resolves current + target page
    -> capture/reuse rendered previews
    -> StPageFlip animates fold geometry
    -> Reader decides accept or reject
    -> only then commit navigation
```

This reduces the risk of the animation library becoming a second, competing navigation system.

## Why Page Flip remains high-risk

It has to coordinate:

- vendor animation DOM;
- captured Reader page content;
- pointer ownership;
- cross-spine boundaries;
- selection state;
- lower-bar behaviour;
- acceptance/rollback;
- navigation state.

The project already fixed real issues around stale selection state and held-drag continuity across one-page boundaries. Focused tests pass, but it remains one of the most lifecycle-sensitive Reader subsystems.

---

# 12. PDF architecture

PDF is a separate fixed-layout path rather than a forced adaptation of the EPUB Reader.

Key components include:

```text
PdfReaderSession
PdfReaderSurface
PdfSearchController
PdfTextCache
PdfAnnotationController
PdfAnnotationAnchor
```

The PDF surface owns:

- document/password lifecycle;
- page arrangement;
- page rendering;
- virtualised mounts;
- raster caching;
- text layers;
- links and destinations;
- outline/page labels;
- search;
- text selection;
- PDF-specific marks;
- zoom/pan;
- crop;
- rotation;
- fixed-layout position state.

PDF anchors use page geometry/text facts rather than EPUB-style chapter character offsets.

This separation prevents reflowable assumptions from leaking into fixed-layout documents.

---

# 13. Markdown, Journals, Documents, and Scriptum

Markdown is both a readable publication type and an editable source format.

## 13.1 Render path

```text
book_files Markdown bytes
  -> Markdown parser / block scanner
  -> sanitised safe HTML
  -> rendered publication in Reader
```

The parser preserves source-range information so visible rendered blocks can be mapped back to exact source sections.

## 13.2 Edit path

```text
rendered block
  -> locate exact source range
  -> rendered edit OR source edit OR Atomic editor
  -> revision-aware commit
  -> re-render
  -> restore logical viewport
```

The durable authority remains Markdown source bytes, not generated HTML.

## 13.3 Journals and Documents

These are classifications layered onto publication records.

Important consequence:

- a Markdown item can remain Markdown while also being classified as a Journal;
- Document state is additive rather than a new parser;
- Mark as Book can allow writing content to appear in conventional Library surfaces without erasing its writing identity.

## 13.4 Vaults

Vaults are organisational containers that can also reference external Android folders through SAF.

The app separates:

- logical Vault structure;
- publication membership;
- Android URI permissions;
- conflict/sync planning.

Portable backup strips device-specific URIs while retaining the logical Vault model.

---

# 14. Annotation and marking architecture

Reflowable annotations are stored independently from publication source files.

The typical flow is:

```text
long press / selection handles
  -> normalise selection
  -> create source anchor
  -> show NON·LIBER mark actions
  -> persist annotation record
  -> render wrappers in chapter DOM
  -> expose in marks panel/statistics
```

Reflowable anchors use chapter-relative character information so they can survive layout changes.

PDF marks use page geometry and a separate anchor model.

An annotation can carry combinations of:

- highlight;
- underline;
- strikeout;
- comment;
- colour;
- selected text/context;
- timestamps;
- anchor/signature data.

## Overlapping marks

The reflowable renderer can create multiple wrappers around overlapping ranges rather than forcing annotations into a single mutually exclusive style.

## Comments vs Notes

These are deliberately different systems.

- **Comment** belongs to an annotation/marked range.
- **Note** is a separate app-owned record that may be standalone or attached to publication/location/text.

---

# 15. Search architecture

## 15.1 Reflowable and Markdown search

Search is asynchronous whole-publication scanning.

A simplified flow:

```text
query
  -> scan normalised chapter text
  -> periodically yield to UI
  -> produce snippets + logical anchors
  -> retain bounded result set
  -> render initial batch
  -> lazy-expand more results
```

The current Reader retains up to roughly 1,000 reflowable results and paints them in smaller batches rather than dumping the whole result list into the DOM at once.

Page-range search is a filter against available page estimates; it is not a separate full-text index.

## 15.2 PDF search

PDF search operates over extractable PDF text pages and can retain a larger result set.

There is no OCR in the audited build, so image-only scanned PDFs cannot become searchable automatically.

---

# 16. Typography and theme architecture

Typography and Reader behaviour are intentionally separate ownership systems.

## 16.1 Typography resolution

The active profile model resolves approximately as:

```text
file-specific override
  -> enabled field from assigned typography profile
  -> global typography fallback
  -> factory default
```

Automatic profile scopes include:

- Book;
- Document;
- Markdown;
- Journal.

This means the application can maintain different reading styles for different content classes without copying all settings into every file.

## 16.2 What typography profiles own

They can own text appearance such as:

- font family;
- font size;
- line height;
- side margin;
- paragraph spacing;
- letter spacing;
- word spacing;
- alignment;
- hyphenation;
- weight;
- kerning;
- ligatures;
- first-line indent;
- font-family force override;
- top/bottom text borders;
- per-theme font colour.

They do **not** own unrelated app state such as reading progress, annotations, metadata, or navigation history.

## 16.3 Theme ownership

The app can resolve:

- one global theme;
- separate app and book themes;
- optional per-surface themes for Library, Artefact, and Scriptum.

The four audited theme identities are:

- Light / Cloud Manuscript;
- Sepia / Leather Archive;
- Dark / Umber Graphite;
- Abyss / Black Lacquer.

Reader text colour and Reader paper/background can have independent overrides without forcing a new global UI theme.

---

# 17. Gesture and input ownership

NON·LIBER has several competing gesture systems, so input cannot be handled as "every recogniser listens at once".

The Reader input controller classifies ownership.

A simplified priority stack is:

```text
1. active modal / editor / Atomic / RCVP
2. active selection or annotation state
3. PDF zoom / pan / text layer
4. brightness edge candidate
5. scroll intent
6. Page Flip hold/drag
7. ordinary swipe/tap page navigation
8. lower-bar recovery gesture
```

This is one of the reasons Reader input is complex: the same finger movement can mean different things depending on mode and context.

Native events such as volume keys and Android Back are routed into logical application commands rather than mutating random UI state directly.

---

# 18. Storage architecture

The main durable store is IndexedDB:

```text
Database: nonliber2
Schema:   version 7
```

## 18.1 Main stores

| Store | Purpose |
|---|---|
| `books` | Publication metadata, progress, classifications, Reader settings, memberships, custom cover state |
| `book_files` | Original imported publication source bytes |
| `settings` | Global settings, UI state, Artefact/Scriptum/Vault preferences, restore state |
| `bookmarks` | Reflowable and PDF bookmarks |
| `book_file_meta` | Sidecar metadata and recovery facts keyed by file identity |
| `annotations` | Highlights, line marks, comments, anchors |
| `fonts` | Imported font families/files and font bytes |
| `collections` | Collection catalogue |
| `shelves` | Shelf catalogue |
| `notes` | Standalone/attached revisioned Notes |

## 18.2 Other persistence lanes

| Mechanism | Examples |
|---|---|
| **localStorage** | theme prepaint, Library view/sort/group preferences, recovery drafts, progress-index cache |
| **sessionStorage** | short-lived repair/reset state |
| **Android SharedPreferences** | SAF folder state, shell background, seamless mode |
| **Android SAF folders** | exported metadata, publication source backups, Vault source trees, crash reports |
| **In memory** | active screen state, caches, current engine, open overlays, gesture ownership |

## 18.3 Why multiple stores exist

Not every piece of state has the same portability or lifetime.

`stateRegistry.js` explicitly classifies state lanes such as:

- portable user data;
- device-local state;
- transient recovery state;
- derived caches;
- externally backed source data.

This prevents, for example, a device-specific SAF URI from being mistaken for portable backup data.

---

# 19. Publication identity and metadata ownership

NON·LIBER does not rely only on display title/author to identify a publication.

A publication record can contain:

- internal `book.id`;
- `fileKey`;
- original filename;
- extension;
- format;
- MIME;
- source size;
- source hash;
- extracted format metadata;
- app-owned display metadata.

Display title or author can therefore be edited without rewriting the original EPUB/PDF/FB2/Markdown source metadata.

Identity and restore logic prefers strong content/file facts and uses ambiguity guards rather than silently assuming two same-named files are identical.

This is essential for:

- reimport;
- sidecar recovery;
- metadata restore;
- avoiding marks being attached to the wrong edition.

---

# 20. Backup, restore, and reimport

NON·LIBER has **three related but different durability mechanisms**.

## 20.1 Metadata Backup

The metadata package is a checksummed JSON archive of app-owned state.

It can include:

- publication records;
- identity facts;
- progress;
- per-file settings;
- bookmarks;
- annotations/comments;
- Notes;
- collections;
- shelves and order;
- custom covers;
- fonts and font bytes;
- typography/theme settings;
- Library preferences;
- Artefact state;
- Scriptum/Vault logical state.

It intentionally does **not** include publication source bytes.

## 20.2 Library Source Backup

This is the complementary source-byte lane.

It copies actual imported publication bytes into a user-selected Android folder with a manifest.

This is what protects the original file data that metadata backup does not contain.

## 20.3 Sidecars

Before destructive deletion, NON·LIBER can keep file-identity-linked sidecar metadata so reimport of the matching source can recover metadata and marks.

Sidecars are not source backups.

The correct mental model is:

```text
Metadata Backup       = app-owned organisation/state
Library Source Backup = actual publication bytes
Sidecar recovery      = deletion/reimport continuity
```

## Restore design

Restore is not a blind overwrite.

The architecture includes:

- package validation;
- checksums;
- preview;
- publication matching;
- weak/ambiguous match handling;
- ID remapping;
- merge phases;
- Note revision logic;
- profile/theme normalisation;
- restore journal/checkpoint state.

This is one of the stronger durability areas in the project, though full destructive cross-device testing remains high-value because restore touches many state lanes.

---

# 21. Android bridges

JavaScript calls named Android interfaces for native-only work.

## Main bridge families

| Bridge | Responsibility |
|---|---|
| `NovaHaptics` | light tap/page-turn tactile feedback |
| `NovaBrightness` | Reader window brightness apply/clear/capability |
| `NovaSelection` | suppress Android native selection menu while retaining selection handles |
| `NovaVolume` | enable/disable volume-button page-turn interception |
| `NovaFonts` | select/scan Android font folders |
| `NovaVaults` | one-shot Markdown folder import |
| `NovaVaultSync` | persistent SAF source operations for Vaults |
| `NovaBackup` | metadata/source backup streaming |
| `NovaShell` | match Android shell background to app state/theme |
| `NovaLinks` | safely open external URLs via Android |
| `NovaFiles` | configure/reset NON·LIBER folder and write crash reports |
| `NovaSeamless` | immersive system-bar state |

The bridge layer is deliberately explicit rather than giving JavaScript broad unrestricted filesystem/native access.

---

# 22. Performance model

NON·LIBER has several design choices intended to keep the application responsive without pretending expensive work does not exist.

## Helpful strategies

- shell-first startup;
- prepaint theme before full DB hydration;
- deferred loading of large surfaces;
- current-content-first Reader opening;
- idle progress/page-index backfill;
- cached Blob URLs for publication resources;
- async search yielding;
- lazy result rendering;
- PDF page virtualisation;
- bounded PDF raster/text caches;
- render scheduling;
- warm PDF sessions;
- memory-pressure cache release;
- streamed backup writes instead of one massive bridge string.

## Expensive paths

- storing and reading full source bytes through IndexedDB;
- exact pagination across large reflowable books;
- repagination after typography/orientation change;
- whole-book scan search;
- dense overlapping annotation wrapping;
- serialising large metadata backups containing font bytes;
- very large monolithic UI modules;
- Page Flip preview capture and cross-spine coordination;
- synchronous native bridge crossings;
- base64 transfer for some binary paths.

There is no audited universal benchmark or guaranteed maximum library size. Performance claims should therefore be made qualitatively unless a specific measured device/content result exists.

---

# 23. Build and packaging

## Android configuration

The audited app has:

```text
applicationId: com.nonlibrenova.reader
debugId:       com.nonlibrenova.reader.debug
minSdk:        26
targetSdk:     36
compileSdk:    36
versionCode:   194
versionName:   0.6.0.0.5
ABI:           arm64-v8a
Java/Kotlin:   target 17
```

## Canonical runtime asset owners

`android/runtime-assets.json` defines the sources that are allowed into the packaged WebView application:

```text
index.html
src/
styles/
third_party/
assets/
```

These are mirrored into:

```text
android/app/src/main/assets/web/
```

The mirror is generated and parity-checked.

## Canonical Windows build

```text
cd android
build-apk.bat --clean
```

The script is intended to:

1. regenerate the web asset mirror;
2. verify asset policy/parity;
3. run a clean Gradle build;
4. emit the APK.

## macOS audit build

The audited verification also succeeded with:

```text
cd android
./gradlew :app:clean :app:assembleDebug
```

The current public-release caveat is that release signing was still configured around debug signing in the audited tree. That is acceptable for internal/public-test APK work but not a finished production release pipeline.

---

# 24. Test architecture

Testing is unusually extensive for a local reader project.

The repository includes hundreds of scripts/contracts/runners spanning:

- Node pure-logic contracts;
- parser fixtures;
- migration tests;
- settings ownership;
- publication capabilities;
- backup/restore;
- annotations;
- browser/CDP integration;
- PDF rendering;
- Page Flip;
- Markdown editing;
- Android source/bridge contracts;
- ADB/device evidence;
- deterministic packaging/parity checks.

The audit reported focused passes including:

| Area | Audit result |
|---|---|
| Startup / packaging | Pass |
| Publication capability matrix | 536 checks passed |
| Typography scope | 20 passed |
| Settings ownership | 64 passed |
| Portable-state registry | 27 passed |
| Restore ownership graph | 43 passed |
| Scriptum Vaults | 147 passed |
| Notes storage | 86 passed |
| PDF capabilities | 31 passed |
| StPageFlip integration | 22 passed |
| AI Narration dormant boundary | Pass |

Not every script is an independent current test. Some are aggregate aliases, historical evidence lanes, source contracts, or environment-dependent browser/device runners.

---

# 25. What is strong today

The strongest architectural characteristics are:

## 25.1 Deterministic local runtime packaging

The app does not depend on remote CDN runtime code for its core behaviour. The packaged web mirror has an explicit allowlist/policy and parity verification.

## 25.2 Explicit capability gating

EPUB, Markdown, Kindle-family files, FB2, and PDF do not lie about having identical feature sets.

## 25.3 Strong identity and reimport thinking

Publication identity is based on more than mutable display metadata, reducing wrong-edition restore risk.

## 25.4 Separate durability lanes

Metadata backup, source backup, sidecars, portable state, device state, and recovery state are not all mixed into one opaque blob.

## 25.5 Logical locations survive reflow

Navigation is not built around fragile page-number identity.

## 25.6 Narrow native shell

Android-specific privileges are explicit and relatively contained.

## 25.7 Testing culture

The project has substantial contract/browser/device proof around high-risk systems rather than relying only on "it opened once on my phone".

---

# 26. What is still risky or incomplete

## 26.1 `reader.js` concentration

`reader.js` remains a very large integration owner. This creates change-blast risk because engines, gestures, overlays, settings, marks, editing, transitions, and persistence remain closely coupled.

## 26.2 Large companion modules

Artefact, Scriptum, PDF surface, MainActivity, and the global stylesheet are also large enough that changes can have distant consequences.

## 26.3 Page Flip lifecycle complexity

The feature is functional and repaired, but it coordinates many owners and remains one of the easiest places for stale state or timing problems to reappear.

## 26.4 External SAF provider variability

Google Files, OEM document providers, cloud-backed providers, removable storage, and permission revocation can behave differently. Browser tests cannot fully prove these paths.

## 26.5 WebView storage limits

Publication bytes and imported fonts live in IndexedDB. The audited app has no universal user-facing storage quota estimator or guaranteed maximum library contract.

## 26.6 Restore blast radius

Restore is carefully designed, but it modifies many important state lanes and deserves destructive device testing with real long-lived libraries.

## 26.7 Accessibility breadth

Some semantics and reduced-motion work exists, but there is no complete audited TalkBack/switch-access/large-font/device matrix.

## 26.8 Tablet readiness

Responsive code exists, but broad physical tablet proof was not part of the current evidence set.

---

# 27. Dormant and placeholder systems

Advanced users should distinguish source presence from active product capability.

## Dormant / legacy

- AI Narration module source remains in the repository;
- custom local voice/model concepts remain in historical code/assets;
- the active packaged runtime excludes that narration system;
- the Audiobook UI does not currently represent a working packaged narration runtime.

## Recognised but unavailable / placeholder

- RAR extraction;
- TXT Reader;
- HTML/HTM Reader;
- DOC/DOCX/RTF Reader;
- FB3 Reader;
- generic rare-format reader.

These should not be advertised as supported merely because classifications or extension helpers exist.

---

# 28. Developer navigation: where to look first

If you are changing a specific subsystem, start here.

| Task | First places to inspect |
|---|---|
| App startup | `src/startup/`, `src/main.js`, `MainActivity.kt` |
| Library query/UI | `src/ui/library.js`, publication query/domain modules, `src/storage/catalog.js` |
| Artefact | `src/ui/artefact.js`, `src/storage/artefactRows.js`, `src/domain/smartIntersection.js` |
| Reader integration | `src/ui/reader.js`, `src/ui/reader/` |
| EPUB/reflow | `src/engine/LiteEpubEngine.js`, `src/engine/epub/`, paginator |
| FB2 | `src/engine/fb2.js` and reflow adapter path |
| Kindle | `src/engine/kindle.js` and publication adapter path |
| Markdown render | `src/engine/markdown/` |
| Markdown edit | `src/engine/markdown/`, `src/editor/atomic/`, Reader edit controllers |
| PDF | `src/engine/pdf/` |
| Page Flip | `src/engine/pageFlip/`, Reader transition/input code |
| Search | Reader search controller + engine adapters + PDF search controller |
| Marks | `src/engine/annotate.js`, annotation storage/controllers, PDF annotation modules |
| Typography | `src/settings.js`, `src/storage/typographyProfiles.js`, `src/engine/fonts.js`, Reader typography UI |
| Themes | theme ownership modules, `src/engine/theme.js`, CSS tokens |
| IndexedDB | `src/storage/db.js` |
| Publication identity | `src/storage/identity.js`, `src/storage/fileIdentity.js` |
| Backup/export | `src/storage/exportPackage.js`, Android `NovaBackup` bridge |
| Restore | `src/storage/importPackage.js`, restore matcher/apply/journal modules |
| Scriptum | `src/ui/scriptum.js`, Scriptum storage/domain modules |
| Vault sync | `src/storage/vault*.js`, `NovaVaultSync` in Android shell |
| Notes | `src/storage/notesRepository.js`, `src/ui/notesEditor.js` |
| Native Android | `android/app/src/main/java/.../MainActivity.kt` |
| Build/package | `android/app/build.gradle.kts`, `android/runtime-assets.json`, `android/build-apk.bat` |
| Tests | `tools/`, `package.json`, `src/testing/`, evidence/docs |

---

# 29. End-to-end flows

This section shows how major user actions travel through the architecture.

## 29.1 Import an EPUB

```text
User chooses file
  -> Android SAF picker
  -> JavaScript receives selected file/bytes
  -> detect extension/MIME/container
  -> publication parser/probe
  -> compute file identity + content hash
  -> extract metadata/cover
  -> write book record + source bytes + identity/sidecar data
  -> optional Library Source Backup
  -> Library query sees new publication
```

The original source bytes and app-owned metadata are intentionally separate.

## 29.2 Open a reflowable book

```text
Library/Artefact selects book
  -> load book record + source bytes
  -> EngineRegistry chooses adapter
  -> build manifest / TOC / capabilities
  -> restore logical location if valid
  -> render current spine/chapter
  -> apply typography + theme
  -> paginate current layout
  -> Reader becomes interactive
  -> background progress/page indexing continues
```

## 29.3 Change font size

```text
User changes font size
  -> resolve current typography owner/profile/file scope
  -> update effective typography state
  -> inject new Reader CSS
  -> invalidate old page geometry
  -> preserve logical reading anchor
  -> rebuild current pagination
  -> page count may temporarily become provisional
  -> persist applicable setting/profile field
```

## 29.4 Turn a page with Page Flip

```text
Pointer starts eligible gesture
  -> Reader input controller grants Flip ownership
  -> resolve target page/spine
  -> capture current/target preview surfaces
  -> StPageFlip renders fold
  -> user releases
  -> Reader evaluates commit/cancel
  -> accepted: navigation commits once
  -> rejected: restore exact original state
```

## 29.5 Create a highlight/comment

```text
Long press text
  -> native handles remain available
  -> Android native action menu suppressed
  -> Reader normalises selected text
  -> create logical source anchor
  -> user chooses highlight/line/comment
  -> persist annotation in IndexedDB
  -> render wrappers in current content
  -> sidecar/update statistics become available
```

## 29.6 Edit Markdown

```text
Open Markdown in Reader
  -> rendered HTML includes source-range mapping
  -> user enters Edit
  -> rendered/source/Atomic path obtains source revision
  -> edit source text
  -> revision-aware commit to stored Markdown bytes
  -> preserve/recover viewport
  -> re-render document
```

Recovery drafts exist separately so a failed/interrupted edit does not have to immediately corrupt portable user state.

## 29.7 Export metadata backup

```text
User chooses Metadata Backup
  -> stateRegistry decides portable lanes
  -> exportPackage builds schema-v3 package
  -> add checksums + identity information
  -> serialise fonts/user metadata/settings/marks/Notes/etc.
  -> NovaBackup streams output to Android-selected folder
```

Publication source bytes are deliberately excluded and require Library Source Backup.

---

# 30. Practical contributor rules

## Rule 1 - Edit canonical sources, not the Android mirror

Do not hand-edit:

```text
android/app/src/main/assets/web/
```

Treat it as generated output.

## Rule 2 - Respect ownership boundaries

Before adding behaviour, determine whether it belongs to:

- Reader;
- publication engine;
- storage/domain;
- Android bridge;
- UI surface;
- transient state;
- portable state.

Do not solve a local UI problem by inventing a second data model.

## Rule 3 - Capability-gate format-specific UI

Do not expose a feature merely because another format supports it.

## Rule 4 - Preserve logical location during reflow

Never treat page number as the canonical reading identity for reflowable content.

## Rule 5 - Keep source files canonical

EPUB/PDF/FB2 source bytes should not be rewritten merely because app metadata or annotations changed.

For Markdown, edited Markdown source is canonical; rendered HTML is derived.

## Rule 6 - Treat Page Flip as visual preview, not navigation authority

The Reader owns final commit/cancel.

## Rule 7 - Be explicit about durability

When adding state, decide whether it is:

- portable;
- device-local;
- transient;
- derived/cache;
- externally backed.

Then add export/restore rules intentionally.

## Rule 8 - Test lifecycle, not just happy-path clicks

Important systems should be tested across:

- open/close;
- background/return;
- force close;
- orientation;
- device memory pressure;
- cancelled picker;
- missing source;
- invalid restore state;
- different Android/WebView implementations where possible.

---

# 31. Compact architecture glossary

**Adapter**  
Format-specific layer that exposes a publication through a common Reader-facing contract.

**Artefact**  
Customisable composition layer over shared Library records.

**Book record**  
App-owned publication metadata/state stored separately from the original source bytes.

**Capability gating**  
Enabling UI/actions only when the active format/engine actually supports them.

**EngineRegistry**  
Format/engine selection authority for opening publications.

**Logical location**  
Stable reading anchor such as chapter/spine + offset rather than current page number.

**Page map**  
Current-layout mapping between logical content positions and visible pages.

**Portable state**  
User data that should survive export/restore to another installation/device.

**Reflowable**  
Content whose layout changes with viewport/typography, such as EPUB/FB2/Kindle-converted content.

**Return origin**  
Saved UI context used so exiting Reader can restore the previous surface/search/filter/scroll state.

**SAF**  
Android Storage Access Framework, used for user-granted file/folder access without broad filesystem permission.

**Sidecar**  
App-owned metadata/recovery record associated with durable file identity, separate from source bytes.

**Source Backup**  
Backup of actual imported publication bytes, distinct from metadata backup.

**Typography profile**  
Reusable set of text-appearance fields resolved by scope/file ownership.

**Vault**  
Scriptum organisational structure that may also be connected to an Android folder through SAF.

**WebView shell**  
Android host that runs the bundled JavaScript/HTML/CSS application from a secure local app-assets origin.

---

# Final architecture summary

NON·LIBER is best understood as a **large local JavaScript reading/writing system running inside a deliberately narrow Kotlin Android shell**.

Its strongest engineering ideas are not visual tricks. They are ownership rules:

- one shared publication/data model across Library, Artefact, Reader, and Scriptum;
- format capabilities instead of pretending all files are equal;
- logical reading anchors instead of fragile page identity;
- local source bytes separated from app-owned metadata;
- portable state separated from device/transient state;
- Android-native work exposed through explicit bridges;
- Page Flip treated as a visual interaction layer rather than a second navigation engine;
- backup split into metadata and actual source-byte protection;
- tests aimed at state/lifecycle boundaries, not only static UI.

The main technical challenge is no longer proving that the app can do complicated things. It already can. The challenge is containing complexity: reducing large integration surfaces, protecting state ownership, increasing device breadth, and hardening release/distribution without losing the fast, local, deeply configurable behaviour that defines NON·LIBER.
