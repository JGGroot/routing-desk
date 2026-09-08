# Routing Desk

Planner-style board for routing documents through a squadron, built as a single
self-contained SharePoint Script Editor web part. Replaces a Microsoft Planner
board that could no longer take document uploads through Teams.

Vanilla JS, no dependencies, no build step. One file.

## Deploy

1. Paste `routing-desk.html` into a Script Editor web part. It is a fragment,
   not a full page — no `<html>` / `<body>` wrapper.
2. Reload. The board comes up in "not set up yet" state.
3. Overflow menu (`⋯`) → **Setup / health check** → **Create what is missing**.
   Requires site Owner / Full Control once. Safe to re-run; it only creates
   what is absent.
4. Overflow menu → **Settings** for buckets, labels, thresholds and email
   notifications.

Opened as a local file it runs on seeded sample data so it can be demoed
without touching SharePoint. `?rddemo=1` forces that mode.

**Upgrading from 2.0.0:** re-run Setup. 2.1.0 adds an `RdMentions` column to
`RoutingLog`; the health check will flag it and add it in place. Nothing else
changes and no data moves.

## Mount point

The web part owns `#rtg`. If a loader stub has already put `#rtg` on the page it
adopts that host and drops its own copy, so it works both standalone and behind
a loader. `window.__RTG_DESTROY__()` tears it down for stub-driven unload.

## What it stores

Provisioned automatically into the host site:

| Item | Type | Holds |
|---|---|---|
| `RoutingCards` | list | one item per routing card |
| `RoutingLog` | list | comments and the activity trail, keyed by `RdKey` |
| `RoutingConfig` | list | a single item holding settings as JSON |
| `RoutingDocs` | library | a folder per card, named by `RdKey` |

Card columns are prefixed `Rd` (`RdKey`, `RdBucket`, `RdOrder`, `RdProgress`,
`RdPriority`, `RdStart`, `RdDue`, `RdCompleted`, `RdNotes`, `RdAssigned`,
`RdLabels`, `RdChecklist`, `RdFiles`, `RdComments`, `RdFolder`). Log columns are
`RdKey`, `RdKind`, `RdType`, `RdText`, `RdWhen`, `RdMentions`.

## Features

Buckets with drag-and-drop and in-column reordering. Completed cards collapse
into a per-bucket **Completed** section rather than cluttering the column.
Multi-assignee people picker resolved through `ensureuser`. Labels, checklists,
due and start dates, priority, progress.

Comments take `@mentions` — type `@`, pick from the directory, and that person
gets emailed with a link to the card. Documents upload the moment they are
dropped or picked; on a brand new card, adding a document creates the card and
uploads in one step. Card edits autosave, including when the card is closed
mid-edit.

Board / Grid / Charts views, grouped by bucket, person, progress, priority or
due date — dropping a card on a lane sets that field, so dragging onto a person
reassigns. My-cards filter, CSV export, deep links (`#rdcard=RD-xxxx`).

Email on assignment and mention via SharePoint `SendEmail`, toggleable, with a
**Send a test email** button in Settings and visible errors when the site's mail
service refuses. ETag concurrency with a conflict path. Dark mode, keyboard
navigation, focus-trapped dialogs, and a diagnostics panel holding the last 120
REST calls.

## Keyboard

`Enter` opens a card, `M` moves it, `C` completes it, `Esc` closes dialogs.
