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

**Upgrading from 2.0.0:** re-run Setup. 2.1.0 added an `RdMentions` column to
`RoutingLog`; the health check will flag it and add it in place. Nothing else
changes and no data moves. 2.2.0 changes how email is sent — see below.

## Email

Microsoft retired SharePoint's own `SP.Utilities.Utility.SendEmail` API for
SharePoint Online on **31 October 2025**, so on SPO it returns HTTP 400 and no
mail is sent. Notifications go through a Power Automate flow instead.

Build the flow once:

1. Create a flow with the **When an HTTP request is received** trigger.
2. Give it this request body schema:

   ```json
   {
     "type": "object",
     "properties": {
       "to":      { "type": "array", "items": { "type": "string" } },
       "subject": { "type": "string" },
       "html":    { "type": "string" },
       "from":    { "type": "string" },
       "card":    { "type": "string" },
       "link":    { "type": "string" },
       "site":    { "type": "string" }
     }
   }
   ```

3. Add **Office 365 Outlook → Send an email (V2)**: *To* = `join(triggerBody()?['to'], ';')`,
   *Subject* = `subject`, *Body* = `html`, and turn the body's rich-text switch on.
4. Save, copy the trigger's **HTTP POST URL**, and paste it into
   Settings → *Power Automate HTTP POST URL* with the transport set to
   **Power Automate flow**.
5. Press **Send a test email to myself**. It sends using the values on screen,
   so you can prove the wiring before saving.

The trigger URL carries its own signature, so anyone who can read the board's
config list can also read the URL and make the flow send mail. Keep the flow
limited to sending, and treat the URL as semi-public. If that is not acceptable,
drive notifications from a flow triggered on **When an item is created** in
`RoutingLog` instead — every action and comment writes a row there, mentions
included, so no secret has to live in the page at all.

On-premises farms can still select **SharePoint SendEmail (on-prem only)**,
where the API continues to work.

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

Email on assignment and mention through a Power Automate flow (see **Email**
above), toggleable, with a **Send a test email** button in Settings and visible
errors when mail is refused rather than a silent failure. ETag concurrency with
a conflict path. Dark mode, keyboard
navigation, focus-trapped dialogs, and a diagnostics panel holding the last 120
REST calls.

## Keyboard

`Enter` opens a card, `M` moves it, `C` completes it, `Esc` closes dialogs.
