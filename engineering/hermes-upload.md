---
name: hermes-upload
description: Generate a structured handoff document from the current conversation and upload it to Google Drive. Trigger immediately when the user says "hermes upload" — do not ask for clarification, proceed through all steps. Creates a versioned plain text file in the "Hermes Handoffs" Drive folder, auto-incrementing the version number (v1, v2, v3, etc.) based on files already in the folder with the same chat name. Also use this skill when the user says "export to hermes", "send to hermes", or "hermes handoff".
---

# Hermes Upload

Generates a structured handoff document so the next session can resume with full context, then uploads it to Google Drive with auto-versioned naming tied to the exact chat title.

## Trigger
`hermes upload`

---

## Execution — Run All Steps in Order

### Step 1 — Generate the Handoff Document

Using the full context of this conversation, produce the following document. Write it as if the next session has zero prior context — be specific and complete, not high-level:

```
1. WHAT WE BUILT OR CHANGED
   List every feature, fix, or file created or modified in this conversation.
   Include exact filenames and what each one does.

2. DECISIONS MADE
   Every architecture, design, or product decision reached.
   Include the reasoning and what alternatives were ruled out.

3. CURRENT KNOWN BUGS OR ISSUES
   Anything broken, half-built, or flagged for later.

4. EXACT STATE OF THE CODEBASE
   What is deployed and working vs. WIP vs. planned but not started.

5. OPEN QUESTIONS OR DEFERRED ITEMS
   Things explicitly set aside with "we'll do this later."

6. CONTEXT HERMES NEEDS THAT ISN'T IN THE CODE
   Business logic, user behavior observations, workarounds,
   gotchas in the tech stack, anything implicit.
```

Format: plain text. Actual details, not paragraph summaries.

---

### Step 2 — Determine the Chat Name

Use the **exact title of the current Claude chat conversation** as the file name prefix.

- Do NOT derive or invent a name from the conversation content.
- Do NOT paraphrase or shorten the chat title.
- The chat title is the canonical identifier for this conversation's handoff series.
- Examples of correct usage:
  - Chat titled "Payments API Refactor" → use `Payments API Refactor`
  - Chat titled "Q3 Analytics Dashboard" → use `Q3 Analytics Dashboard`
- If you are uncertain of the exact chat title, ask the user before proceeding.

This ensures that all handoffs from the same chat session share a consistent naming series
(v1, v2, v3...) and are never mixed with handoffs from a differently-titled chat.

---

### Step 3 — Find or Create the "Hermes Handoffs" Folder

Search Google Drive for a folder named exactly **Hermes Handoffs**.

- **Found**: note its folder ID and proceed.
- **Not found**: create it with `Google Drive:create_file` using:
  - name: `Hermes Handoffs`
  - mimeType: `application/vnd.google-apps.folder`

If a Hermes Handoffs folder ID is already configured for this project, use it
directly instead of searching: <drive-folder-id>

---

### Step 4 — Determine the Version Number

Search the Hermes Handoffs folder for files whose names begin with the exact chat name from Step 2.

- **No matches**: assign **v1**
- **Matches found**: read the version numbers in their filenames, take the highest, and increment by 1

---

### Step 5 — Upload the File

Create a new plain text file in the Hermes Handoffs folder:

- **Filename**: `[Chat Name] v[N]`
  - Example: `Payments API Refactor v3`
- **Content**: the handoff document from Step 1
- **Format**: plain text
- **Important**: set `disableConversionToGoogleType: true` to prevent Drive from
  converting to Google Docs format

Use `Google Drive:create_file`.

---

### Step 6 — Confirm to User

After a successful upload, report:
- The exact filename uploaded
- The Drive folder it lives in (Hermes Handoffs)
- The version number assigned
- One sentence summarizing what the next session will find when it opens the file

---

## Error Handling

- **Drive not connected or auth error**: display the handoff document in chat, tell the user Drive isn't reachable, and ask them to check the Google Drive connector in Settings.
- **Folder creation fails**: report the error and show the document in chat so nothing is lost.
- **Version detection returns no results**: default to v1 and note this assumption in the confirmation message.
- **Chat title uncertain**: ask the user for the exact chat title before uploading. Never guess.

## Hard Rules

- Always use the exact chat title — never derive, shorten, or invent a name.
- Always run version detection before assigning a number — never assume v1 without checking.
- Never skip the upload — if something fails, show the document in chat rather than silently failing.
- The handoff document must be written as if the next session is reading a cold brief with no prior context.
