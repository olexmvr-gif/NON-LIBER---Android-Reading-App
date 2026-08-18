# NON·LIBER — Advanced User Guide

> **Public technical overview**  
> **Audience:** advanced users, testers, technically curious readers, and people who want to understand how NON·LIBER behaves without exposing the private implementation blueprint.  
> **Reference build:** NON·LIBER v0.6.0.0.5  
> **Platform:** Android

---

## Contents

1. [What NON·LIBER is](#1-what-nonliber-is)
2. [The shortest useful model](#2-the-shortest-useful-model)
3. [What it is made with](#3-what-it-is-made-with)
4. [The four main surfaces](#4-the-four-main-surfaces)
5. [Supported publication families](#5-supported-publication-families)
6. [How importing works from a user perspective](#6-how-importing-works-from-a-user-perspective)
7. [How the Reader works](#7-how-the-reader-works)
8. [Why page counts can change](#8-why-page-counts-can-change)
9. [Reading modes](#9-reading-modes)
10. [Page turns and Page Flip](#10-page-turns-and-page-flip)
11. [PDF reading](#11-pdf-reading)
12. [Typography and reading appearance](#12-typography-and-reading-appearance)
13. [Themes and colour ownership](#13-themes-and-colour-ownership)
14. [Search](#14-search)
15. [Bookmarks, highlights and annotations](#15-bookmarks-highlights-and-annotations)
16. [Library metadata and organisation](#16-library-metadata-and-organisation)
17. [Artefact](#17-artefact)
18. [Scriptum, Markdown, Journals and Documents](#18-scriptum-markdown-journals-and-documents)
19. [Notes and comments](#19-notes-and-comments)
20. [Vaults and external folders](#20-vaults-and-external-folders)
21. [Backup, restore and reimport](#21-backup-restore-and-reimport)
22. [Local-first privacy model](#22-local-first-privacy-model)
23. [Performance model](#23-performance-model)
24. [Android integration](#24-android-integration)
25. [What is mature and what still needs broader testing](#25-what-is-mature-and-what-still-needs-broader-testing)
26. [Advanced-user tips](#26-advanced-user-tips)
27. [Compact glossary](#27-compact-glossary)
28. [Public documentation boundary](#28-public-documentation-boundary)

---

# 1. What NON·LIBER is

NON·LIBER is a **local-first Android reading and writing application** designed around a simple idea: the user owns the files, the library, the reading state and the organisation.

It combines four large areas:

- a conventional **Library**;
- a customisable **Artefact** library surface;
- a full **Reader**;
- **Scriptum**, the Markdown / Journal / Document side of the app.

These are not separate apps stitched together. They work over the same local publication collection and the same app-owned metadata.

A title changed in one place is still the same title elsewhere. A collection, shelf, reading state, bookmark, mark or custom cover belongs to the publication itself rather than to one screen.

---

# 2. The shortest useful model

```text
Library   = find, sort and organise
Artefact  = compose a personalised library surface
Reader    = read, search, navigate, mark and style
Scriptum  = write, edit and organise Markdown-based material
```

The important principle is **shared ownership**.

NON·LIBER does not create a second copy of your books merely because you view them through Artefact, Scriptum or another organisation layer.

---

# 3. What it is made with

NON·LIBER uses a hybrid Android architecture.

At a high level:

- **Kotlin** handles Android-specific integration;
- **JavaScript** handles most application behaviour;
- **HTML and CSS** render the interface and reading surfaces;
- **Android WebView** hosts the local application interface;
- **local device storage** keeps publications, settings and user data;
- locally bundled reading components handle reflowable books, PDF, Markdown and visual page-turn effects.

This is not a cloud web app presented inside a phone shell. The application assets are packaged locally and the core reading workflow is designed to function offline.

## Why use this architecture?

It allows the app to combine:

- Android-native file and device access;
- highly custom reading layouts;
- rapid interface iteration;
- format-specific rendering systems;
- local-first storage;
- a single shared UI architecture across Library, Reader, Artefact and Scriptum.

The Android layer remains focused on device integration while the reading and organisation logic lives primarily in the local application layer.

---

# 4. The four main surfaces

## Library

Library is the straightforward collection view.

Use it when you want to:

- import publications;
- browse normally;
- sort and filter;
- organise by author, series, genre, collection or shelf;
- edit app-owned metadata;
- continue reading;
- find books quickly.

## Artefact

Artefact is a custom composition layer over the same library.

It can present books through arranged rows, entities, collections, shelves, authors, genres, series and other user-configured blocks.

Use it when you want your library to feel curated rather than merely sorted.

## Reader

Reader is where publications are opened.

Depending on format, it can provide:

- paginated reading;
- scrolling;
- seamless reading;
- PDF fixed-layout reading;
- search;
- bookmarks;
- annotations;
- typography controls;
- progress;
- custom page-turn behaviour;
- Markdown editing where supported.

## Scriptum

Scriptum is the writing and document environment.

It is designed around:

- Markdown;
- Journals;
- Documents;
- Notes;
- Vault organisation;
- writing-focused rows and views.

---

# 5. Supported publication families

The reference build supports these main families:

| Format family | Typical use | Reading model |
|---|---|---|
| EPUB | General e-books | Reflowable |
| FB2 / FB2.zip | FictionBook publications | Reflowable |
| DRM-free MOBI / PRC | Legacy Kindle-style books | Reflowable conversion path |
| DRM-free AZW / AZW3 | Kindle-family books | Reflowable conversion path |
| Markdown | Reading and writing | Reflowable + editable |
| PDF | Fixed-layout documents/books | Dedicated PDF reader |

Not every format has every feature.

That is intentional.

NON·LIBER enables features according to what the current format can genuinely support rather than pretending that PDF, EPUB and Markdown are equivalent.

Examples:

- PDF does not use EPUB typography controls;
- Markdown can be edited while EPUB cannot;
- image-only PDF pages cannot be text-searched without OCR;
- DRM-protected Kindle files are outside normal support.

---

# 6. How importing works from a user perspective

When you import a publication, NON·LIBER separates two ideas:

1. **the original publication file**;
2. **the app-owned information about that publication**.

App-owned information can include:

- display title;
- author overrides;
- series information;
- collections and shelves;
- reading progress;
- custom cover;
- marks and comments;
- classification;
- per-book reading settings.

This separation is important because it allows you to customise your library without rewriting the original EPUB, PDF or FB2 file.

Markdown is different because editing Markdown intentionally changes the stored Markdown source used by the app.

---

# 7. How the Reader works

For reflowable publications, the Reader does not treat the visible page number as the permanent identity of your position.

Instead, it retains a logical place in the publication and then calculates how that place appears under the current layout.

Why?

Because changing any of these can alter the page count:

- font family;
- font size;
- line spacing;
- margins;
- orientation;
- screen size;
- publisher styling.

The Reader therefore tries to preserve **where you are in the text**, even when the number printed at the bottom changes.

This is one of the most important things to understand about a reflowable reader.

---

# 8. Why page counts can change

Page count in a reflowable e-book is not stored as an absolute fact inside the book.

It must be calculated from the current layout.

NON·LIBER can show usable reading content before every page in a long book has been completely measured. This means the total can initially be provisional and then settle as more of the publication is mapped.

You may notice this after:

- first opening a large book;
- changing font size significantly;
- changing margins;
- rotating the device;
- switching layout conditions.

The important point is that the **reading location** and the **current calculated page number** are different concepts.

---

# 9. Reading modes

## Pages

Traditional paginated reading.

Best when you want:

- clear page boundaries;
- tap/swipe navigation;
- page-turn effects;
- visible page-based progress.

## Scroll

Continuous vertical reading.

Best when you prefer a web/article-like flow or do not want discrete page turns.

## Seamless

Continuous publication flow designed to reduce visible chapter boundaries and support certain Markdown/document workflows more naturally.

## PDF modes

PDF has its own fixed-layout choices, including single-page, scrolling and spread-style presentation depending on configuration.

---

# 10. Page turns and Page Flip

NON·LIBER supports several page-turn styles for eligible reflowable content.

The visual effect is kept separate from the underlying reading position.

That means the animation does not redefine what the current page actually is. The Reader remains responsible for deciding whether a turn completes or returns to the original page.

This makes it possible to offer a physical-looking page curl without making the visual effect the authority over navigation.

Page Flip is one of the more complex interaction features and remains especially useful for public testing across different screen sizes and refresh rates.

---

# 11. PDF reading

PDF uses a dedicated fixed-layout reading system rather than pretending to be an EPUB.

Its feature set includes, where the document permits:

- page rendering;
- scrolling;
- spread/double-page presentation;
- zoom and pan;
- text selection;
- search;
- bookmarks;
- PDF-specific marks;
- links and outline navigation;
- crop and rotation controls.

Because PDF is fixed-layout, ordinary e-book typography settings do not reflow the document.

A scanned PDF made only from page images will also not become text-searchable automatically because the reference build does not provide OCR.

---

# 12. Typography and reading appearance

Typography is one of NON·LIBER's deeper systems.

For reflowable books and Markdown, users can control combinations of:

- font family;
- imported fonts;
- font size;
- line height;
- paragraph spacing;
- side margins;
- letter spacing;
- word spacing;
- alignment;
- hyphenation;
- weight;
- kerning;
- ligatures;
- first-line indent;
- top and bottom text borders;
- per-theme text colour.

## Profiles

Typography profiles let a reusable reading style apply automatically to classes of content such as:

- Books;
- Documents;
- Markdown;
- Journals.

A specific file can also be given its own profile choice when it needs an exception.

The useful mental model is:

```text
specific file choice
    ↓
assigned typography profile
    ↓
fallback reading appearance
```

Profiles concern text appearance. They do not replace reading progress, metadata, bookmarks or annotations.

---

# 13. Themes and colour ownership

NON·LIBER supports four main visual theme families:

- Light;
- Sepia;
- Dark;
- Abyss.

Theme ownership can be more flexible than a single app-wide switch.

Depending on configuration, the interface and Reader can use separate theme choices, and major surfaces can use their own presentation.

Reader text colour and reading-paper colour can also be adjusted independently from the overall application chrome.

This allows the library UI and the actual page to remain visually distinct without requiring separate applications or duplicated theme systems.

---

# 14. Search

Search is local.

For reflowable publications and Markdown, the Reader searches through publication text and returns navigable results with excerpts.

The UI does not need to display every result at once. Larger result sets can be revealed progressively so the interface remains responsive.

PDF search uses extractable PDF text rather than the reflowable book path.

Search results are navigation aids, not permanent annotations. Temporary search emphasis disappears after navigation.

---

# 15. Bookmarks, highlights and annotations

The marking system is app-owned rather than written back into the original publication.

Depending on format, users can create:

- bookmarks;
- highlights;
- underlines;
- strikeouts;
- comments.

Reflowable marks are attached to the textual location rather than to a fixed screen rectangle, so changing typography does not intentionally detach them from the marked words.

PDF annotations use a PDF-appropriate location model because PDF pages do not reflow like EPUB text.

Overlapping marks are supported in the reflowable Reader, allowing multiple forms of emphasis to coexist.

---

# 16. Library metadata and organisation

NON·LIBER keeps user organisation separate from embedded publication metadata.

You can organise publications using concepts such as:

- authors;
- series;
- genres;
- tags;
- collections;
- shelves;
- favourites;
- Reading List;
- finished / paused / unread states;
- Documents;
- Journals;
- custom covers.

Changing these in NON·LIBER does not normally rewrite the original e-book metadata inside the publication file.

This makes the library highly customisable while keeping original files intact.

---

# 17. Artefact

Artefact is best understood as a **view composer**.

It lets the same underlying library be presented through arranged blocks rather than a single permanent grid.

Possible building ideas include:

- recently opened books;
- reading lists;
- favourite books;
- one chosen shelf;
- one collection;
- an author-focused row;
- genre groupings;
- series views;
- entity galleries;
- custom sections;
- combined smart views.

The important rule is that Artefact changes **presentation**, not publication identity.

Deleting or rearranging an Artefact section does not mean deleting the underlying books.

---

# 18. Scriptum, Markdown, Journals and Documents

Scriptum extends NON·LIBER beyond conventional e-book reading.

Markdown can be:

- imported;
- rendered as readable text;
- edited;
- classified;
- organised into writing-focused views.

## Journals

Journal is an app-owned classification used to distinguish writing-oriented Markdown material.

## Documents

Document is another classification and can coexist with other states rather than representing a completely different parser.

## Editing

The app supports rendered and source-oriented Markdown editing paths.

The principle is simple:

- Markdown source is the durable content;
- rendered presentation is derived from that source;
- after editing, the document is re-rendered while the app attempts to preserve reading/editing continuity.

---

# 19. Notes and comments

Comments and Notes are different.

## Comment

A Comment belongs to an annotation or marked text range.

Use it when the note is directly about a particular passage.

## Note

A Note is a separate writing object.

It can be standalone or connected to a publication/location depending on how it is created.

This distinction keeps quick textual annotation separate from larger user-created writing.

---

# 20. Vaults and external folders

Vaults organise writing material inside Scriptum.

They can group Markdown/Journals/Documents without requiring the original Android filesystem to mirror the same structure exactly.

Where external folder access is configured, NON·LIBER can work with Android's user-granted document/folder system rather than broad unrestricted filesystem access.

External-folder behaviour can vary between Android document providers, so users should test important two-way workflows with non-critical material first and keep backups.

---

# 21. Backup, restore and reimport

NON·LIBER separates several durability concepts because they protect different things.

## Metadata Backup

Protects app-owned information such as:

- organisation;
- settings;
- progress;
- marks;
- Notes;
- typography data;
- customisation;
- selected local assets such as imported fonts.

It is not the same as backing up every imported publication file.

## Library Source Backup

Protects the original publication files themselves.

## Reimport continuity

The app keeps identity information so a matching publication can recover associated user state after deletion/reimport where possible.

The safest practice is to keep both:

```text
Metadata Backup       → your NON·LIBER state
Library Source Backup → your publication files
```

Neither replaces the other.

---

# 22. Local-first privacy model

The core application is designed around local storage and local processing.

Important principles in the reference build include:

- no account requirement for core reading;
- no advertising layer in the reading workflow;
- no cloud dependency for ordinary library use;
- publication data remains on the device unless the user explicitly exports, backs up or connects an external folder;
- Android file/folder access is user-granted.

Opening an external web link leaves the local application context and is handled by Android separately.

---

# 23. Performance model

NON·LIBER tries to make expensive work happen without blocking ordinary reading unnecessarily.

Examples include:

- opening current content before every long-running calculation is complete;
- delaying lower-priority work until after the main surface is usable;
- gradually refining page information;
- progressively displaying large search result sets;
- caching publication resources;
- virtualising expensive PDF page rendering;
- releasing caches when memory pressure requires it;
- streaming larger backup operations rather than treating everything as one giant in-memory action.

There is no single meaningful maximum-book-size or maximum-library-size figure that applies to every Android device.

Actual limits depend on:

- available storage;
- WebView behaviour;
- file complexity;
- image size;
- number of marks;
- imported fonts;
- device memory.

---

# 24. Android integration

Although much of the interface runs in a local WebView environment, NON·LIBER still relies on native Android integration for tasks a browser surface should not own directly.

Examples include:

- choosing files and folders;
- persistent access to approved folders;
- brightness control;
- haptic feedback;
- hardware volume-button behaviour;
- Android Back behaviour;
- system UI / immersive reading behaviour;
- saving backups to user-selected storage;
- handing external links to Android.

The goal is to keep platform-specific responsibilities at the Android boundary while the reading experience remains consistent inside the main application.

---

# 25. What is mature and what still needs broader testing

The reference build has strong evidence around the core reading experience, especially:

- Library;
- EPUB reading;
- core reflowable Reader behaviour;
- themes;
- bookmarks;
- marks;
- typography;
- metadata organisation;
- local persistence;
- startup and position continuity.

Areas that benefit especially from broader device/content testing include:

- Artefact with very large libraries;
- FB2 from many publishers/encodings;
- unusual or very large PDF files;
- imported font families;
- restore on fresh devices;
- Scriptum editing across different keyboards;
- external Vault folder providers;
- Page Flip hand-feel on different refresh rates;
- tablets, foldables and unusual display sizes.

Some concepts present in historical development material are not active public capabilities and should not be treated as supported simply because related ideas once existed.

---

# 26. Advanced-user tips

## Use Library for retrieval, Artefact for composition

Library is usually faster for finding something specific. Artefact is better when you want a persistent curated arrangement.

## Use typography profiles before per-file overrides

Profiles keep your reading environment coherent. Reserve file-specific settings for genuinely unusual books.

## Do not treat page count as immutable

In a reflowable book, typography defines pagination. The stable concept is your reading location, not the current total number of pages.

## Keep both backup types

Metadata Backup protects your NON·LIBER state. Library Source Backup protects the publications themselves.

## Use Comments and Notes differently

Use a Comment for a thought attached to marked text. Use a Note when the writing should exist as its own object.

## Test external-folder sync cautiously

Android document providers differ. Use non-critical material first when evaluating a new provider or device.

---

# 27. Compact glossary

**Artefact**  
A customisable composition layer over the same underlying Library.

**Capability-based reading**  
Features are enabled according to what the current publication format can genuinely support.

**Fixed-layout**  
A document whose page geometry is intrinsic to the file, such as PDF.

**Journal**  
A writing-oriented classification used for Markdown material.

**Local-first**  
The primary application workflow stores and processes user data on the device rather than requiring a remote account/service.

**Logical reading location**  
A position in the publication that can survive repagination better than a visible page number.

**Metadata Backup**  
Portable backup of app-owned state rather than the original publication files.

**Note**  
A separate user-created writing object, distinct from an annotation comment.

**Reflowable**  
Text whose visible pagination changes with typography and screen geometry.

**Scriptum**  
NON·LIBER's Markdown / Journal / Document / Notes workspace.

**Source Backup**  
Backup of the original publication files themselves.

**Typography profile**  
A reusable reading-appearance configuration that can be applied by content type or to a specific file.

**Vault**  
A Scriptum organisational container that can optionally be associated with user-approved Android folders.

---

# 28. Public documentation boundary

This document intentionally explains **behaviour, product architecture and user-visible system design** without publishing the private implementation blueprint.

It does not include:

- private source-tree paths;
- exact internal module names;
- internal bridge/API identifiers;
- database/store identifiers;
- private build commands;
- internal test harness details;
- exact implementation weaknesses or code-size hotspots;
- security-sensitive implementation specifics;
- contributor/debugging navigation intended for the private project repository.

That information belongs in private engineering documentation.

The public goal is different: an advanced user should be able to understand **why NON·LIBER behaves the way it does**, what its major systems mean, and how to use them intelligently without receiving a map of the proprietary source implementation.

---

# Final summary

NON·LIBER is a local-first Android reading and writing system built around a shared publication library, flexible organisation, format-aware reading, strong typography, annotations, Markdown writing and explicit backup ownership.

Its most important user-facing architectural ideas are:

- **one shared library beneath Library, Artefact, Reader and Scriptum;**
- **format-aware features instead of one fake universal Reader;**
- **logical reading locations instead of trusting reflowable page numbers;**
- **app-owned metadata and marks kept separate from original publication files;**
- **local-first storage and user-approved Android file access;**
- **separate protection for metadata and original publication files;**
- **deep customisation without requiring a remote service or account.**

That is the architecture an advanced user needs to understand.
