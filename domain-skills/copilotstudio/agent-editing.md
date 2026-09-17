# Microsoft Copilot Studio — agent editing & workflows

URL base: `copilotstudio.microsoft.com`. Heavy SPA: `wait_for_load()` returns long
before the app renders. After `goto_url`, sleep 10–15s and poll for a concrete
selector instead of trusting readyState.

## Auth

Unauthenticated hits redirect to `login.microsoftonline.com`. With cached sessions
you get a "Pick an account" tile list — clicking a tile completes SSO without
credentials. If a password/MFA form appears instead, stop and hand off to the user.

## URL patterns

- Agent overview (Build tab): `/environments/<env-id>/agents/<agent-id>`
- Workflows list: `/environments/<env-id>/workflows`
- Workflow designer: `/environments/<env-id>/flows/<flow-id>` (title "Agentic Automations")
- Appending `/tools` or similar to an agent URL just re-renders the overview — the
  Build-tab sections (Skills/Tools/Knowledge) are not separately routable.

## Agent instructions editor

- Selector: `[aria-label="Agent instructions"]` — a Lexical-based contenteditable
  SPAN (`fai-EditorInput__input`), directly editable inline on the Build tab. No
  separate "Edit" mode needed.
- **Replace content**: `el.focus()` + `Range.selectNodeContents(el)` into the live
  selection, then CDP `Input.insertText` (helpers' `type_text`) with the full new
  text. Reliable for ~6k chars; newlines become paragraphs.
- The editor appends zero-width chars (`​‌`) per block — tolerate them in
  `innerText` comparisons, and expect innerText length to differ a few % from your
  source (list markers normalize).
- The editor is NOT virtualized: if `innerText` ends mid-sentence after scrolling
  the editor to the bottom, the cloud copy really is truncated — a known state when
  agents built on the `cliagent` template were touched with `pac copilot get`
  (instructions are UI-only for that template; Get can clobber them).
- Save: top-bar `Save` button. After a successful save the button's tooltip/aria
  reads "No unsaved changes" — that, not a toast you may have missed, is the
  reliable saved-state check. Toast `[role="status"]` shows "Saving agent" briefly.
- Verify persistence by re-navigating to the agent URL and re-reading the editor.

## Publish

`Publish` button top-right on the agent page. Toast sequence: "Publishing your
agent…" → "Agent published successfully"; the button label goes `Publishing…` →
back to `Publish`. Takes ~10–20s.

## Workflows (Agentic Automations)

- List rows: single click only selects the row; click the **name cell text** to
  navigate into the designer.
- Designer canvas has nodes (e.g. `Recurrence`, `Agent`); clicking a node opens its
  settings panel on the right. Recurrence panel exposes Frequency/Interval, day
  chips (Sun–Sat), hours/minutes inputs and Time zone.
- Day-chip selection state is not exposed via aria-pressed/checked — read the run
  history instead: the Activity panel lists runs with aria-labels like
  "Select Succeeded run from 9/17 8:00 AM", which tells you the effective schedule
  (and failures: "Select Failed run from …").
- An `Agent` node's config (Connection, target agent, fixed Message prompt) is
  visible in `document.body.innerText` once the node is selected.
