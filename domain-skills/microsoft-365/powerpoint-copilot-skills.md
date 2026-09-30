# PowerPoint web: Copilot custom skills

## Surface and frame boundaries

PowerPoint web editing commonly lives inside a SharePoint/OneDrive `/_layouts/15/doc2.aspx` page. The editor frame uses `euc-powerpoint.officeapps.live.com/pods/ppt.aspx`; Copilot's custom-skill manager and chat live in a separate Office add-in frame, observed at `fa000000129.resources.office.net`. Locate the live target rather than assuming that the editor frame contains the Copilot controls. Hostnames can vary by deployment.

Coordinate clicks normally cross these frames. After DOM/CDP work on a child target, explicitly switch back to the top-level document before taking a screenshot or continuing coordinate interaction. Re-screenshot to verify the active surface.

## Personal custom-skill management

In Copilot, the plus menu exposes Choose skills and management of plug-ins and skills. The Custom Skills section can create the personal OneDrive Skills folder when it does not exist. Global Custom Skills and individual entries have separate enabled states; uploading alone does not prove a skill is active.

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

## Chat and task completion

The chat editor is a Lexical contenteditable with `aria-label="Type your message"`. Typing `@` opens a skill-selection list; select the visible matching skill and verify that the mention was committed before appending the request. The send button has `aria-label="Send"`.

A message visible in the composer is still unsent. Verify a new “You said” entry and a running Copilot response. Multi-step edits can take several minutes and show changing reasoning statuses. Wait for the finished reply and enabled composer before downloading the result.

When keyboard focus or coordinate sending repeatedly fails after iframe attachment, focus the actual contenteditable in the Copilot frame and dispatch input to that frame's CDP session. Re-check the top-level screenshot after the action. Do not infer submission from a click alone.

## Download and validation

File → Create a copy → Download a copy opens a second confirmation dialog with a Download button. Selecting the menu item alone does not start the file download. Configure browser download behaviour, complete the dialog, and verify the resulting file.

A completed Copilot response is not proof that formatting properties were applied. Inspect the downloaded native PowerPoint and compare relevant font-family, colour, text, shape, notes, character-spacing, line-spacing, and title-placeholder properties against the request. Distinguish font-family definitions from fonts actually available to the cloud renderer. Verify all affected slides visually in the browser as well.
