# Training workspaces workbook

What the app enforces when this workbook is uploaded in **Training workspaces**.

## Sheets

- Instructions: the rules, by topic.
- Training Drafts: the drafts, each a task row followed by its step rows.
- Skills: the skills and agent reconciliations that guidance can mention, each with its **Mention to copy**.

The download holds the person's own unpublished workspaces in the subsidiary, or a blank **Training Drafts** sheet when there are none. The upload reads only **Training Drafts**, and only its first eight columns, whose headers must be exactly, in order: Workspace ID, Workspace Name, Folder, Test Period, Task Description, Step No., Step Title, Detailed Guidance.

## Rows

- A non-empty Task Description starts a draft. Its row also carries Workspace ID, Workspace Name, Folder and Test Period, and may hold step 1.
- Each further step takes the next row with only Step No., Step Title and Detailed Guidance filled. Keep a draft's rows together and never sort them; completely blank rows are ignored.
- Number each draft's steps 1, 2, 3, with a title on every step, and give the complete step list, not only the changed steps. Every draft needs at least one step.
- Write values in the eight columns: a formula or Excel error there fails, Folder included.

## Workspace ID

As downloaded, it edits that draft; blank adds a new one, which needs permission to create training workspaces.

## Test Period

Each draft's own month to test against, as text `YYYY-MM`, such as `2026-10`; an Excel date on the first day of a month also works. Tests read the earlier periods of the draft's folder template, never the test period's own work folder.

## Workspace Name

Unique in the workbook and among all the subsidiary's training workspaces, ignoring case; at most 60 characters; no `< > : " / \ | ? *` or control characters; no trailing dot or space. Keep names unless a rename is asked for.

## Folder

Required: the name of one active folder template in the subsidiary, matched ignoring case. It is the work folder the task belongs to, not a cloud path or a period folder. A restricted folder template needs access to it. Keep existing values, and ask which template a new draft belongs to.

## Descriptions and steps

Length limits and mentions follow the `author-agent-task` skill (a training draft takes the shorter description); a full mention token for an unknown skill fails.

## What applying does

- Each of the person's unpublished workspaces in the subsidiary whose ID is missing from the sheet is deleted, with its test history and its cloud folder, including all input and output files. A sheet with only headers deletes them all.
- The whole apply is refused while any row has an error. Rows then save one at a time and deletions last, and a failure in one does not undo the others.
- An edit replaces the name, folder, test period, description and steps, and keeps files, conversations, test runs, web access and approved email recipients.
- A new draft starts with web access off and no approved recipients; the person sets them in the workspace.
- Importing never runs or publishes a draft. Publishing later creates a separate recurring task.

## Error catalog

The upload stops at its first message, most of which start `Excel row <n>: `. The review shows each row's error under that row, and nothing applies while one remains. Each entry gives the cause, its messages and the fix.

- Wrong sheet or headers: `The workbook needs a Training Drafts sheet.`, `Expected column <n>: <header>. Use the exported template.` Restore the sheet and headers from a fresh download.
- Too large: `The Training Drafts sheet exceeds 10,000 rows.`, `Choose an Excel file smaller than 10 MB.` Delete empty rows, or split the drafts across two imports.
- A formula or error cell: `replace formulas and errors with values.` Write the values.
- Rows out of shape: `Enter a Workspace Name.`, `Enter Task Description before adding steps.`, `Task fields belong on a row with Task Description.`, `Enter a Step Title.`, `Step No. must be consecutive, starting at 1 for each task.` Regroup and renumber the rows.
- A test period in another form: `Test Period must be YYYY-MM or a date on the first day of the month.` Write `YYYY-MM`.
- A name the rules refuse: `Duplicate workspace name in this workbook.`, `Enter a workspace name.`, `Workspace names must be 60 characters or fewer.`, `Workspace names cannot contain folder path characters.`, `Workspace names cannot end with a dot or space.`, `This training workspace name is already in use in this workflow. Choose another name.` Choose a name that follows Workspace Name.
- An ID the person cannot edit: `Workspace ID appears more than once in this workbook.`, `Training workspace not found.`, `Only the creator can change this training workspace.`, `Workspace is not in this workflow.`, `This draft has already been published.` Keep one row per ID, copied from a fresh download of this subsidiary; remove the rows of a draft that is published or not the person's, which deletes nothing.
- A folder that does not resolve: `Folder is required.`, `Folder "<name>" must match one active folder template in this workflow.`, `You do not have access to this folder template.` Use an active folder template's name the person can access.
- Lengths and steps: `description must be at most 300 characters`, `Agent tasks require at least one step`, `Step <n>: unknown skill "<id>".`, `HTML markup is not allowed`. Shorten the description, add a step, copy the mention from **Mention to copy**, or remove the markup.
- Errors on apply, or changes since the review: `Excel row <n>: <error>`, `Excel row <n>: the draft changed. Preview the workbook again.`, `Workspace "<name>" changed. Preview the workbook again.` Fix the row, or select **Review changes** again.
- A deletion refused: `Stop active or waiting tests before deleting this workspace.` Stop the tests, download again, and remove that workspace's rows again.
