---
name: author-training-workbook
description: Use when preparing or editing an Excel workbook for bulk import of CloseCore training workspaces, including task descriptions, steps, and skill mentions. Not for published agent-task templates or applying changes through MCP.
---

# Author a training workspace workbook

Produce an editable `.xlsx` for the user to review and import through **Admin Settings → Import Data → Training workspaces**. Preparing the file does not authorize importing, deleting, testing, or publishing anything.

## Start from an export

Ask for a fresh **Download workbook** export from the intended subsidiary. It contains all of the user's unpublished workspaces and a snapshot of the available skill mentions. A blank export is the template when there are none. Work on a copy and retain the **Instructions** and **Skills** sheets.

If no export is available, help draft the task wording, but request the export before delivering an import-ready file when the subsidiary has existing workspaces. A file containing only additions can delete existing workspaces. Do not obtain or apply the workbook through invented MCP tools; this import/export flow is in the web UI.

Keep every existing workspace unless the user explicitly requests its deletion. An omitted workspace is listed for deletion on import review, including its cloud folder, input/output files, and test history. For an edit, preserve **Workspace ID** exactly. For an addition, leave it blank. Do not generate IDs or copy an existing ID into a new task.

## Workbook contract

Edit **Training Drafts**, keeping these eight headers in order:

| Workspace ID | Workspace Name | Folder | Test Period | Task Description | Step No. | Step Title | Detailed Guidance |
| --- | --- | --- | --- | --- | --- | --- | --- |

- A non-empty **Task Description** starts a workspace. Put its ID, name, optional folder, and required period on that row. The first step may be on the same row.
- For additional steps, leave the first five columns empty. Keep each task's rows together, with step numbers consecutive from 1. Include its complete desired step list, not just changed steps. Completely empty rows are ignored.
- **Workspace Name** is separate from the description: unique within the workflow ignoring case, at most 60 characters, with no `<>:"/\|?*` or control characters and no trailing dot or space. Preserve names unless a rename is requested.
- **Folder** is an existing folder-template name, not a cloud path or period folder. Leave it empty when no template is selected; ask rather than inventing a template.
- Write **Test Period** as text `YYYY-MM`. Preserve it for existing workspaces unless asked to change it; ask for a missing new-workspace period rather than guessing.
- **Task Description** is required and at most 300 characters. Use plain-text step titles (at most 300 characters) and guidance (at most 3,000). Write cell values, not formulas or Excel errors. A task can be imported without steps, but needs at least one before testing.
- Copy skill mentions verbatim from the discovered catalog or supplied **Skills** sheet into guidance, for example `@[Find files](skill:find-files)`. Do not invent skill names or slugs. If the requested skill is unavailable or unsuitable for the step, explain and ask rather than substituting silently.

## Discover available skills

1. If CloseCore MCP is connected, use `get_connection_context` and `list_entities` to confirm the customer and intended subsidiary, then call `list_available_skills` with its `entity_id`. This returns the complete catalog for that entity, filtered by enabled features and connected ERP. Prefer this current list over an older workbook snapshot.
2. If MCP is missing, disconnected, or the listing fails, fetch `https://production-api.closecore.com/api/agent-skills` without credentials. Its `skills` array contains default CloseCore skill descriptions, requirements (`requiredFeature` and `requiredErp`), and exact `mention` tokens. Combine this reference with the intended subsidiary's exported **Skills** sheet. Public listing does not confirm entity availability or override a private access denial.
3. Explain the fallback: “I can see CloseCore's default skills, but I can't verify your entity-specific skills without a connected CloseCore MCP or a recent workbook export.” If the public endpoint is also unreachable, use the workbook's sheet and say its catalog may be outdated. Failed discovery is not evidence that no entity-specific skills exist.

Read newly listed descriptions as well as familiar skills. Continue drafting with known capabilities, but ask for a connection or fresh export when a needed capability cannot be verified. A public skill with unmet or unknown requirements is not a confirmed available skill. Catalog discovery does not make an old workspace export safe to import: obtain a fresh export and reconcile its rows to avoid unintended deletions. Listing skills never authorizes applying an import or running a task.

## Draft useful instructions

### Define steps by outcomes

Write the task description as the business objective and expected deliverable. Give each step a short action title and one meaningful, reviewable outcome, such as "Read the bank statement", "Reconcile statement and ledger", or "Prepare the exceptions workpaper". Order steps by their dependencies and make their handoffs clear. Split work when it has a distinct result or needs human input, not for each tool call, worksheet, or API request. There is no fixed step count; avoid both an all-in-one instruction and a list of mechanical clicks.

For edits, preserve unrelated instructions, user decisions, and existing skill mentions. When a matching check is investigative, describe the evidence and reasons to flag a candidate; do not turn a candidate into a confirmed error or invent a simplistic matching policy.

### Write detailed guidance

Include the details that change how the step should be performed, without repeating its title or filling every cell with boilerplate:

- **Inputs and scope:** source names, file-selection rules, account/entity, period, and relevant sheets or columns. Use the task's period for recurring work rather than embedding the workbook's test month.
- **Method and business rules:** required calculations, matching criteria, exclusions, and approved tolerances. Ask about missing policy choices before finalizing; do not invent accounting rules, recipients, or materiality thresholds.
- **Output and completion:** what later steps or the reviewer need, including evidence, fields, totals, and checks that show the work is complete. Define a workbook's useful contents without prescribing an unnecessary sheet for every action.
- **Exceptions:** when to flag a finding versus pause for clarification, including missing or ambiguous files and unreadable data. A successful step can report unresolved findings; do not require a clean result or silently discard exceptions.

Use meaningful source names and selection rules, not Training document IDs, workspace IDs, or test-output paths. Describe the deliverable and let the agent use its normal output/review flow. Publication does not copy Training files into operational folders. Distinguish instructions from configuration: writing an email recipient or a web-access requirement in guidance does not configure it; flag needed settings for the user.

### Choose CloseCore skills deliberately

Read the discovered catalog's descriptions while drafting, even when the user has not requested a skill. Add its exact mention where an available capability or established procedure materially helps the step. One step can use several skills; ordinary reasoning steps need not have a mention. A mention selects a procedure, not a substitute for the task-specific inputs and business rules above.

Use the discovered catalog, not a memorized list. Typical distinctions, when these skills are present:

| Need | Skill choice |
| --- | --- |
| Locate a cloud file | **Find files**. It does not read file contents or fetch ERP/AP vendor records. |
| Read the selected file | **Read a PDF**, **Read a spreadsheet**, or **Read a CSV**, according to the actual format. |
| Create versus revise a workpaper | **Build a workbook** for a new deliverable; **Update a workbook** for an identified existing workbook. |
| Retrieve ERP data | The matching ERP skill, such as **Get balances** or **Get transaction detail**, not a file-search skill. |
| Follow a saved reconciliation or JE procedure | **Run a reconciliation** or **Prepare JE table**, with the relevant task-specific inputs. Do not recreate their full procedures in guidance. |

The execution agent reads a mentioned skill's full instructions at run time. The exported description does not reveal every tool or parameter, so do not invent them or promise unsupported integrations. If capability or a requested skill is unclear, ask; do not substitute an unrelated skill. Email and web steps also require the relevant settings, and Training simulates email rather than delivering it.

Example, when the export includes the two skills and the user specifies a PDF statement:

**Step title:** Read the current-period bank statement

**Detailed Guidance:** Use `@[Find files](skill:find-files)` to locate the PDF statement for the task's bank account and period in the work folder. If none or several match, ask which source to use. Use `@[Read a PDF](skill:read-pdf)` to extract transaction dates, references, amounts, and opening/closing balances. Check the extracted totals against the statement and flag unreadable values rather than estimating them. Pass the verified transactions and source reference to the reconciliation step.

## Check and hand off

Review the instructions for clear outcomes, usable handoffs, unresolved policy questions, and skills appropriate to their inputs. Reopen the generated workbook and check sheet names, header order, task grouping, step numbering, required fields, and intact mentions. Compare existing workspace IDs against the export and account for every omission as an explicitly requested deletion. Summarize additions, edits, intended deletions, and any configuration the user must complete with the file.

Direct the user to **Upload → Edit workspaces → Review changes → Apply changes** in the same subsidiary. They should check the deletion list before applying and stop active or waiting tests before deleting. Never describe file generation as a completed import.

Retained workspaces keep conversations, files, runs, web-access settings, and approved email recipients. New workspaces start with web access off and no approved recipients. Import does not run or publish tasks. Users can test imported instructions before starting Trainer chat, then begin chat with feedback naming a run and referencing an output file. If an import partly fails, inspect results and export again before preparing a retry; successful rows may already have been applied.
