# PowerPoint web: Copilot custom skills

## Surface and frame boundaries

PowerPoint web editing commonly lives inside a SharePoint/OneDrive `/_layouts/15/doc2.aspx` page. The editor frame uses `euc-powerpoint.officeapps.live.com/pods/ppt.aspx`; Copilot's custom-skill manager and chat live in a separate Office add-in frame, observed at `fa000000129.resources.office.net`. Locate the live target rather than assuming that the editor frame contains the Copilot controls. Hostnames can vary by deployment.

Coordinate clicks normally cross these frames. After DOM/CDP work on a child target, explicitly switch back to the top-level document before taking a screenshot or continuing coordinate interaction. Re-screenshot to verify the active surface.

When several PowerPoint tabs are open, a global first-URL match can select another tab's Copilot iframe. The observed `Target.getTargets` entries expose `parentFrameId`: the Copilot target's parent is the editor iframe, whose parent is the top-level document target. Follow that ancestry to the intended page before attaching to a child. The top-level `Page.getFrameTree` did not include these out-of-process descendants in the observed session.

## Personal custom-skill management

In Copilot, the plus menu exposes Choose skills and management of plug-ins and skills. The Custom Skills section can create the personal OneDrive Skills folder when it does not exist. Global Custom Skills and individual entries have separate enabled states; uploading alone does not prove a skill is active.

Some pane versions put **Manage plug-ins & skills** under the header's **More options** menu; the composer plus button may expose file attachment instead. Use the observed visible menu rather than requiring the plus route. The editor's compact **Chat with Copilot** control was observed with `id="CopilotDAB"`, while the chat add-in may already be preloaded before the pane is visible. Attaching to that preloaded frame does not prove that its UI has opened; verify the rendered pane before composing.

Custom-skill switches were observed as native `input[role="switch"]` elements. Their live `checked` property establishes state; `aria-checked` may be absent even on an enabled switch. Confirm both the global switch and the intended individual entry visually and through the actual control state.

The skill upload file input is hidden:

```css
input[data-testid="skill-upload-input"]
```

It was observed accepting `.zip`, `.skill`, and `.md`. Use `DOM.setFileInputFiles` on this input in the Copilot frame after opening the visible upload workflow. Do not select the last file input generically: a separate replacement input may be present even during upload.

Replacement uses:

```css
input[data-testid="manage-skills-replace-input"]
```

First click Replace on the intended listed skill. Setting files without choosing a target produces “Replace target is unavailable.” To avoid opening a native file dialog during CDP automation, enable `Page.setInterceptFileChooserDialog` for the frame session before the visible Replace click, set the file on the hidden replacement input, then disable interception. Wait for “Uploading skill” to disappear and verify the visible entry and error state. Validate the selected package before replacing it.

Each Replace button has an observed `aria-label="Replace <skill-name>"`. If coordinate interaction repeatedly misses the intended row after frame attachment, this exact label identifies the replacement target without relying on row order or accidentally choosing Delete. Verify that the target-selection error cleared before treating the upload as successful.

## Chat and task completion

The chat editor is a Lexical contenteditable with `aria-label="Type your message"`. Typing `@` opens a skill-selection list; select the visible matching skill and verify that the mention was committed before appending the request. The send button has `aria-label="Send"`.

Custom skill suggestions were observed as `[role="option"][value="<skill-name>"]`. A committed mention becomes a noneditable Lexical decorator. If coordinate selection repeatedly fails, use the visible option's stable value in the Copilot frame, then place the caret at the end of the contenteditable before inserting the remaining prompt. An uncommitted partial mention can otherwise leave the skill-name suffix at the end of the prompt.

On a newly opened Copilot pane, the suggestions may initially show loading placeholders. Wait for the actual matching label and description before selection. Verify a `[data-lexical-decorator][contenteditable="false"]` child in the composer after selecting, before adding the task text. An early selection attempt can leave plain `@name` text that sends without a committed skill chip. Enabled skills may still be selected automatically, but that is different from proving explicit selection in a test.

A message visible in the composer is still unsent. Verify a new “You said” entry and a running Copilot response. Multi-step edits can take several minutes and show changing reasoning statuses. Wait for the finished reply and enabled composer before downloading the result.

When keyboard focus or coordinate sending repeatedly fails after iframe attachment, focus the actual contenteditable in the Copilot frame and dispatch input to that frame's CDP session. Re-check the top-level screenshot after the action. Do not infer submission from a click alone.

The local daemon's selected CDP session is shared by clients with the same `BU_NAME`. If multiple harness processes may be active on the machine, use a task-specific daemon name and explicitly switch to the task's existing tab. This isolates harness session selection while keeping the user's browser/profile. Avoid changing download behaviour for unrelated work; restore the browser default when finished.

## Download and validation

File → Create a copy → Download a copy opens a second confirmation dialog with a Download button. Selecting the menu item alone does not start the file download. Configure browser download behaviour, complete the dialog, and verify the resulting file.

A completed Copilot response is not proof that formatting properties were applied. Inspect the downloaded native PowerPoint and compare relevant font-family, colour, text, shape, notes, character-spacing, line-spacing, and title-placeholder properties against the request. Distinguish font-family definitions from fonts actually available to the cloud renderer. Verify all affected slides visually in the browser as well.



## Brand Kit manager and persistence

The separate Microsoft 365 Copilot kit manager is at `https://m365.cloud.microsoft/create/brandkit`. Asset categories include Logos, Templates, Fonts, Colors, Images, Icons, Brand voice and Skills. Create/save a kit before filling these sections. Ownership, sharing and organization publication are separate states; adding assets does not publish an official kit.

In the observed Brand Kit skill editor, a ZIP import populated only the main SKILL.md in the Instructions field, whereas a self-contained Markdown import retained its full embedded references. This differs from personal skill-package installation. Inspect the actual imported Instructions body before saving; reload, reopen each saved skill's Edit form, and compare the full body to the intended export. A listed skill name alone does not establish retained resources.

An already-open PowerPoint tab can cache the kit picker. Refreshing the presentation exposed a kit created later in the manager. Select brand first, then Choose skills. The attached brand reference exposes an accessible Remove label containing the kit name. Identically named personal and kit skills may appear deduplicated; their presence alone does not prove exclusive kit retrieval.

The observed manager font role selector offered Heading, Subheading and BodyText, with no dedicated accent-label role. Preserve the detailed brand rules when the UI taxonomy is broader. Editing and saving a font role can reorder cards; re-inspect the target after every save instead of relying on previous row positions.

Hidden image inputs shared `data-testid="image-upload-input"` across asset categories. Inspect the enclosing accordion's category text to choose the correct input; do not assume one global upload input. Template inputs accepted presentation files, and custom font inputs accepted .ttf/.otf. Open the visible workflow and verify the saved cards after setting files.

PowerPoint downloads can complete after the confirmation dialog disappears. A short fixed sleep followed by an empty directory check is insufficient evidence of failure. Poll for the complete file and validate the archive before concluding the transfer failed.


### Long Brand Kit instructions in the Edit form

The manual Brand Kit Instructions textarea was observed with HTML `maxlength="4000"`, even when its saved Markdown import contained much more text. Reading its value can return the full saved body; typing a replacement through native input can silently truncate it. Check the live maxlength and compare the complete pending value before Save. A saved card and the opening paragraph do not establish completeness.

In the observed authorized update, removing the frontend maxlength during native input allowed the full body to be retained, saved and verified after reload. Avoid passing one huge Input.insertText request through a harness transport with a smaller line limit; send bounded chunks and compare the final value. Do not save partial instructions on failure. Backend persistence must be checked separately from local input success.

Some kit card menu interactions completed with native pointer input after a semantic DOM click failed to open the intended Edit form. Re-screenshot the actual popup and verify the edit form's skill-name field before filling it; do not continue editing based on a menu click alone.


### Layout copies and desktop export verification

A completed web edit was observed displaying the intended repositioned panel and original SVG pattern, while desktop PowerPoint rendered the downloaded cover using the older composition. The new local layout was correctly registered in the master and assigned to the slide, but it retained the source layout name, its `p14:creationId` value and several shape creation GUIDs. A local copy with a distinct layout name and fresh layout/shape creation identifiers rendered the intended composition in desktop PowerPoint; all other package parts were retained. This demonstrates the repair as a group of changes, not which single identifier drives the desktop resolution.

For layout edits, verify both the web preview and the actual downloaded deck when desktop tools are available. A valid slide-layout relationship and unchanged SVG bytes alone do not establish that the intended composition renders. Prefer native layout cloning and preserve the original source definitions. If generating OOXML layout copies directly, use distinct names/creation identifiers and valid master registration. Keep original export evidence separate from a repaired local artifact.


### Brand Kit image upload completion and pagination

The Images counter can include pending cards immediately after file selection while a toast still says `Uploading 0/N images`. Do not treat the new count as completed persistence. Wait for the upload toast to finish, close any remaining Add images dialog, and reload the kit before checking the total.

The image grid was observed paginating at 20 assets per page. A saved total above 20 can therefore coexist with only 20 `img[alt="Uploaded Asset"]` nodes. Use the visible page controls to review the remaining cards; verify loaded thumbnails across pages instead of waiting for all assets to exist in the first page's DOM. Asset counts for other accordions may finish loading later than Images after reload; wait for each required category rather than assuming all state is ready from one counter.
