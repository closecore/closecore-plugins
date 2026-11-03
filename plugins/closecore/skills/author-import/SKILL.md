---
name: author-import
description: "Use when preparing, editing or checking an Excel workbook for CloseCore's bulk imports: the template configuration workbook (folders, checklists, reconciliations, recurring agent tasks) or the training workspaces workbook, including agent task rows and upload errors. Not for changing tasks directly through the connector; the manage-close skill does that."
---

# Author a CloseCore import workbook

CloseCore has two Excel import workbooks, each downloaded for one subsidiary from **Admin Settings → Import Data**: the template configuration workbook (**Update existing templates**: folders, checklists, reconciliations and, where enabled, recurring agent tasks) and the training workspaces workbook (**Training workspaces**: the person's own unpublished drafts).

You prepare the file; the person uploads, reviews and applies it in the app. Preparing a workbook authorizes nothing: never say an import happened, and never look for a connector tool for these workbooks (none exists).

## Operating rules

- Start from a fresh download made today, from the right subsidiary: an older file can hold IDs that no longer exist and miss newer templates, which the upload would delete. Without one, draft wording only.
- Keep every sheet, its name and every existing row. Keep each **Template ID** and **Workspace ID** as downloaded; a new row's ID stays blank, never generated or copied.
- Write plain text and numbers, never formulas, date cells or true/false cells; the one formula allowed is a template sheet's **Folder** reference.
- A batch, or folders, frequencies and assignees across the board, belongs in the workbook. One task or a handful, or a change to a live task, is often quicker through the connector with the `manage-close` skill; offer that, then follow the person's choice.
- Read the person's example workbook or before-and-after first and match its detail and conventions. Their stated or remembered preferences win over this skill's defaults; offer to remember new ones.
- Live sources win over the reference files: the workbook's **Instructions** and **Skills** sheets, the connector and the app's preview.

## Template configuration workbook

1. The person downloads it: **Admin Settings → Import Data → Update existing templates**, the subsidiary, then **Download workbook**. Without agent tasks enabled and permission to read them, it has no **Agent Task Templates** or **Skills** sheet.
2. Read the **Instructions** sheet, which records whether the downloader may change folders and create tags, and the **Skills** sheet.
3. Agree every row with the person, including each new template's folder, frequency, assignees, due dates, tags and dependencies. `list_assignable_users` gives assignee emails; the **Accounts** sheet lists account numbers.
4. Write the rows as `references/template-workbook.md` describes. Write each new **Folder** cell as the folder's exact **Folder Name** in text, adding a new folder's row first with **Template ID** blank.

## Training workspaces workbook

1. The person downloads it: **Admin Settings → Import Data → Training workspaces**, the subsidiary, then **Download workbook**. It holds only their own unpublished workspaces there.
2. Edit only **Training Drafts**, as `references/training-workbook.md` describes: a first row per draft, then its steps.
3. **Test Period**, per row: the month that draft tests against, as text `YYYY-MM`. Keep existing ones; ask for a new draft's, never guess.
4. Every draft needs at least one step.
5. Each of the person's unpublished workspaces there that the sheet lacks is deleted on apply; a header-only sheet deletes them all.

## Agent task rows

Load the `author-agent-task` skill first and follow it; never draft an agent task's steps without it. Then fill the rows:

- **Task Description** on the task's first row, then one row per step: **Step No.** from 1, **Step Title** and **Detailed Guidance**. Step 1 shares the task's row; each later step row leaves every other cell blank.
- Mentions and length limits follow that skill's "Write the guidance" section.
- Template workbook settings, on the task's row: **Folder**, **Frequency**, CloseCore's own deadline in **Agent Due Date**, the human preparers and reviewers with their due dates (CloseCore is always a preparer), the start mode in **Trigger**, **Web Access**, **Approved Email Recipients** as `Name <email>` (everyone an email step writes to), **Email Policy**, existing Template IDs in **Dependencies**, and **Tags**.
- A training draft takes only the description and steps; the person sets web access and approved recipients in the workspace.

## Check before hand-off

Check the file against the matching reference (its rules and Error catalog) and report a check, result and fix table quoting the app's message.

## Round-trip safety

- Change only the rows and cells the person asked about; never sort or reorder rows; keep every ID.
- Add new rows below the last used row of their sheet, and a new subtask directly under its new task. On **Folder Templates**, clear a removed folder's cells rather than deleting its row (**Folder** references point at row numbers).
- Before hand-off, check that every downloaded **Template ID** or **Workspace ID** the person did not ask to delete is still on its sheet (the review lists a dropped row under **Deleted**, or as Delete), then list the rows added, changed (each cell's old and new value) and removed, with what removal does: a template is deleted from the effective period on (an agent task's active runs stop), and a training workspace loses its test history, cloud folder and files.
- Drop an existing row only once the person confirms that row, even after "clean up" or "start fresh".

## Hand off

- Template configuration workbook: in **Admin Settings → Import Data → Update existing templates**, choose the same subsidiary and, at the **Upload** step, the file. In **Edit templates**, fix highlighted rows, then **Continue**. Under **Apply changes from**, choose the first period to change (earlier periods stay as they are), then **Review changes**. Compare **Added**, **Edited** and **Deleted** with your summary, clear **Fix these before applying** and **Missing permissions**, then **Apply changes**.
- Training workspaces workbook: in **Admin Settings → Import Data → Training workspaces**, choose the same subsidiary and, at the **Upload** step, the file. In **Edit workspaces**, select **Review changes** and fix any row errors; the review lists each row as Add, Edit or Delete. Stop active or waiting tests in a workspace before it is deleted, then **Apply changes**.
- If **Apply changes** fails, download again before retrying, since part of the change may already be saved.
- Automatic triggers can start runs as soon as template changes apply.
