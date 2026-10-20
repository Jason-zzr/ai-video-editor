# Optional local host

The canonical `/` remains the browser editor. The local host is an opt-in Node 24
runtime for durable project files on macOS. It serves an already built editor on
an ephemeral `127.0.0.1` port and opens Chrome. Windows native project storage and
file selection are not implemented. This contribution does not ship installers,
a bundled Node runtime, or model downloads.

## Run from source

With Node 24 selected, Chrome installed, and the macOS command-line developer
tools available:

```sh
npm ci
npm run build
npm run build:storage
npm run host
```

`build:storage` explicitly compiles the included native filesystem helper into
`.cache/local-host/`. It does not install a compiler or runtime. Rebuild it after
updating the helper. Runtime state lives in the current user's
`Library/Application Support/Timeline Studio`; project folders are selected by
the user and are separate from that directory and from Auto-cut's data.

Use File → Save local project to choose a parent folder and create a named
project. Subsequent edits are queued for saving after a short debounce. A saved
status requires a successful durable save receipt. File → Open local project
opens an existing project directory. Reopening a page in the same running host
can restore its remembered project. After stopping and restarting the host,
launch a fresh authenticated page and open the project directory again.

Portable `.timeline` import/export remains available. Complete-project
migration and trash-management interfaces are not part of this integration.
The imported storage implementation retains its supporting record types and
routes; this is not a new public stable API or a promise of cross-platform
project-package compatibility.

## Sessions and recovery

- Launch links contain a short-lived, single-use bootstrap nonce. Browser page
  credentials are retained in session storage, never project files.
- Authenticated activity extends page expiry by 30 minutes. Event heartbeats
  trigger an authenticated refresh at most once per minute. The open stream
  checks the renewed deadline rather than closing at its original deadline.
- One writable editor lease is permitted per project. Leases last up to one
  minute and are renewed every 20 seconds while the page is active.
- An expired lease or a stale **lease** renewal revision permits reacquisition.
  Pending saves are resumed automatically only for a known lease-expiry error.
  Unrelated failures remain explicit. Reacquisition checks the durable editor
  sequence; another page's newer save produces a project conflict and preserves
  the current page's edits rather than overwriting that save.
- Retry preserves the original snapshot and idempotency key when the outcome
  is unknown. Only a confirmed failed save gets a new key. Completed model or
  remote processing jobs are never rerun by lease recovery.
- For a project conflict, export the current work as `.timeline` if needed,
  then explicitly close/open the project. Opening or closing replaces the
  timeline only after confirmation. Stopping the host is explicit and is
  blocked while this page has pending work; other pages must be checked by the
  user before stopping the shared service.

The server accepts exact loopback Host and same-origin authenticated browser
requests. Native file selections are short-lived capabilities. Native storage
uses managed relative paths, project ownership and durable save records; it
rejects unsupported platforms instead of falling back to weaker filesystem
operations.

## Attribution and contribution scope

Adapted from Jason-zzr's MIT-licensed [Auto-cut](https://github.com/Jason-zzr/Auto-cut)
work for [issue #111](https://github.com/MartinDelophy/ai-video-editor/issues/111).
The original author is retained on the imported commits: `6f9fd69`, `cf8e16c`,
`cc21f0c`, `d26fdcd`, `9ad65c0`, and `1a30481`. Compatibility changes retain the
current editor's audio remux path and use the Timeline Studio identity.
Scene segmentation, execution plans, semantic summaries and visual search
(#112–#115) are excluded.

Validation uses `npm run check` (including the host TypeScript project) and
`npm run build:storage`. Session, stream-expiry, lease, retry, real native save /
reopen and HTTP-client checks are run outside the product repository, consistent
with its policy against committed test harnesses.
