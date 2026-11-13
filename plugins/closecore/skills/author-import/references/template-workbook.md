# Template configuration workbook

The rules the app enforces on upload in **Update existing templates**.

## Sheets

1. Instructions
2. Folder Templates
3. Checklist Templates
4. Reconciliation Templates
5. Agent Task Templates
6. Accounts
7. Skills

The first four must keep these names; **Agent Task Templates** and **Skills** come only with agent tasks enabled and read permission. Edits to **Accounts** and **Skills** are ignored, as are blank rows. Each header row holds exactly the headers below, in order, with nothing after them; only the agent sheet's last four may be missing.

Headers, in order:

- **Folder Templates**: Template ID, Folder Name
- **Checklist Templates**: Template ID, Folder, Task Description, Subtask Description, Frequency, Preparer Emails, Preparer Due Dates, Reviewer Emails, Reviewer Due Dates, Tags, Dependencies
- **Reconciliation Templates**: Template ID, Folder, Name, Type, Accounts, Frequency, Fixed Balance, Threshold Amount, Preparer Emails, Preparer Due Dates, Reviewer Emails, Reviewer Due Dates, Tags, Dependencies
- **Agent Task Templates**: Template ID, Folder, Task Description, Step No., Step Title, Detailed Guidance, Frequency, Agent Due Date, Preparer Emails, Preparer Due Dates, Reviewer Emails, Reviewer Due Dates, Tags, Dependencies, Trigger, Web Access, Approved Email Recipients, Email Policy, Run By
- **Accounts**: Account Number, Account Name
- **Skills**: Name, Type, Description, Mention to copy

## Cells, lists and limits

- Required: Folder Name; Folder and Frequency on every template; Type, and Name for a group reconciliation; an agent task's Task Description and each step's Step Title.
- Write text, or numbers where a number is meant; a required name or title, or an agent Task Description, holding only a number counts as missing. No formulas except Folder references, and no error, date or true/false cells.
- Emails, due dates, tags, accounts, dependencies and recipients are lists separated by semicolons. Inside an item, write `\;` for a semicolon and `\\` for a backslash; any other backslash, or an empty item, fails.
- Limits: 10 MiB per file, 10,000 used rows per sheet, 1,000 rows on each folder, checklist and reconciliation sheet, and 1,000 agent tasks.

## Folder cells

The export writes each Folder cell as a direct reference to the folder's row on **Folder Templates**, such as `='Folder Templates'!B4`; the upload reads that row's Folder Name. Point at column B of a filled row from row 2 down (`$` signs are fine), or write the exact folder name as text; either must match, ignoring case, an existing folder template or a **Folder Templates** row. Folder Names are unique, ignoring case, with no `/ \ ? # %`.

## Matching and deletion

- A blank Template ID adds a template; any other ID edits its template, which must still exist, once per sheet.
- An existing template missing from its sheet is deleted from the effective period on; a deleted agent task's active runs stop.
- An **Agent Task Templates** sheet with only headers deletes all the subsidiary's recurring agent tasks. Without the sheet, agent tasks stay as they are, and no folder holding one can be deleted.
- A folder can be deleted only once its templates are deleted or moved in the same workbook.
- Added folders, checklists and reconciliations save before the other changes.

## Checklists and reconciliations

- A checklist row fills Task Description or Subtask Description, not both. A subtask belongs to the nearest task above it in its folder; a new subtask needs a new parent, so add one to an existing task in the app. Only checklists take Weekly and Bi-weekly, and an existing checklist cannot switch between them and other frequencies: delete and re-add it.
- A single reconciliation has one account and takes that account's name, so leave Name blank; a group has a Name and one or more accounts. Write accounts as a number or "number - name" from the **Accounts** sheet, each on one reconciliation row only. Type, and a single reconciliation's account, cannot change: delete and re-add the row. Fixed Balance is optional; Threshold Amount is 0 or more (blank is 0); amounts may use `$`, commas, or parentheses for a negative.

## Assignees, due dates and tags

- Assignees are active people with access to the subsidiary (`list_assignable_users`), none both preparer and reviewer on a row. Due dates pair with emails by position: `alex@example.com;sam@example.com` with `5;15*`.
- A due date is a business-day offset after month-end (negative before it, never 0) or `N*` for day N of the month, 1 to 31. Weekly and Bi-weekly checklists use 0 (Sunday) to 6 (Saturday) or a weekday name, without `*`. Agent Due Date takes the same forms for CloseCore's own deadline; blank keeps it, or uses the default on a new task.
- Tags match existing names, ignoring case; a new name creates a tag, which needs the Manage tags permission.

## Dependencies

Existing Template IDs of checklists, reconciliations or agent tasks, separated by semicolons, without prefixes, from any subsidiary the person can access; blank removes them. A template added in this workbook can be a dependency only after it is applied and downloaded again. No task depends on itself, on a task being deleted, or in a circle; a task others depend on can be deleted only if those dependencies go in the same workbook, and a dependent in another subsidiary must change there first.

## Agent task rows

A non-empty Task Description starts a task, and its row holds all the task's columns. Each further step takes the next row with only Step No., Step Title and Detailed Guidance filled. Steps run 1, 2, 3, each with a title; length limits and mentions follow the `author-agent-task` skill, and the upload never checks that the subsidiary can use a mentioned skill. The email columns manage people, and watchers are kept; on a task CloseCore runs, CloseCore is always a preparer as well.

## Trigger cell

On the task's row, in a form from Accepted values, up to 4,000 characters. Blank keeps the current trigger, or makes a new task Manual. Email settings go on separate lines of the same cell:

```text
Email
Mailbox: bank-statements
Senders: statements@bank.com; treasury@bank.com
Source: Attachment
Pattern: {{MMMM}}_{{YYYY}}
Offset: -1
```

Take every value from the person's real emails, never this example.

- Mailbox: the task's email address name; 3 to 40 lowercase letters, digits or hyphens, no hyphen at either end, used by no other task.
- Senders: 1 to 20 full email addresses separated by semicolons, separate from Approved Email Recipients.
- Source: Attachment (its file name), Subject, or Received (the date received).
- Pattern: required for Attachment and Subject, left out for Received; up to 200 characters, no `*`, with a month or period token ({{MMMM}}, {{MMM}}, {{MM}}, {{M}}, {{PP}}, {{P}}, {{Q}}) and a year token ({{YYYY}}, {{YY}}, {{FFFF}}, {{FF}}).
- Offset: -2 to 2 months from the recognized month to the task's period; default 0.

A bare `Email` keeps saved settings; a new email task needs the whole block and the agent email feature. A weekday start set in the app exports blank and is kept. Every trigger waits for the task's dependencies to complete.

## Option columns

A missing Web Access, Approved Email Recipients or Email Policy column keeps its setting, and a value under its missing header fails. In a present column, blank means web access off, no approved recipients, or Approval. Recipients are `Name <email>` entries separated by semicolons, such as `Alex Smith <alex@example.com>;Pat Lee <pat@example.com>`.

## Run By

Column 19 (S), the last, says who runs the task with a value from Accepted values; letter case does not matter. Without its header every row reads as blank, and a value under the missing header fails. Who runs a task is set when it is created: a new template takes the row's value, or CloseCore when blank, or Own agent for an organization that has only that kind. An existing template keeps its kind, so its row must be blank or match it; to move a task to the other agent, add a new row for a new task.

An Own agent row needs at least one Preparer Email, keeps Trigger as Manual or blank, and leaves Web Access, Approved Email Recipients and Email Policy blank. It has no Agent Due Date, so a value there is ignored. The export writes Own agent rows this way.

## Accepted values

### Frequency

- Weekly
- Bi-weekly
- Monthly
- Quarterly
- Yearly
- Custom: then the months it runs in, full English names separated by semicolons, such as `Custom: January; July`.

### Trigger

- `Manual`: a person starts the task.
- `Date:<n>`: business day n after period-end, such as `Date:5`; negative is before it, never 0.
- `Date:<n>*`: day n, 1 to 31, of the month after period-end, such as `Date:15*`.
- `Dependencies`: once its dependencies are complete.
- `Email`: when a matching email arrives, with the lines Mailbox, Senders, Source, Pattern and Offset.

### Web Access

- Yes
- No

### Email Policy

- Approval: a person approves each draft before it is sent.
- Auto: drafts to approved recipients go without review.

### Run By

- CloseCore: CloseCore's agent runs the task.
- Own agent: the preparer's own agent runs it through the CloseCore connector.

## Error catalog

Upload messages read `<Sheet>, Excel row <n>: <message>`, and the first one stops the upload. **Edit templates** marks problem cells, says `Fix the highlighted rows before continuing.` and disables **Continue**; review messages appear under **Fix these before applying** or **Review changes**. Each entry gives the cause, its messages (long ones by their opening words) and the fix.

- Sheets or headers changed, or a file over 10 MiB: `This does not look like a template configuration workbook`, `header <column>1 must be "<header>"`, `header contains an unexpected extra column`, `Missing header for <header>`. Restore them from a fresh download.
- Too large: `<Sheet> has more than 10000 rows`, `the import supports at most 1000 rows`, `Too many agent tasks`, `Template configuration request is too large`. Delete empty rows, or split the import.
- Agent tasks unavailable: `This workbook includes agent tasks`, `Agent tasks are not enabled.` Remove the agent sheet, or have someone with access upload.
- Formulas and cell types: `formulas are not permitted`, `formula must directly reference a Folder Name cell`, `formula references a blank Folder Name cell`, `contains an Excel date cell`, `has an unsupported Excel cell type`. Write text, or a Folder reference to a filled row.
- Missing or malformed values: `<Header> is required`, `<Header> contains an empty list item`, `<Header> contains an invalid escaped list`, `Template configuration document is malformed: <detail>`, such as `HTML markup is not allowed` or a length limit. Correct the cell or shorten the text.
- IDs: `Template ID must be a UUID`, `duplicate Template ID <id>`, `Template ID <id> no longer exists`. Copy IDs from a fresh download, or leave them blank.
- Frequencies: `Frequency "<value>" is not recognized`, `Custom frequency requires at least one month`, `Custom frequency lists a month more than once`. Use an accepted value.
- Cells highlighted in **Edit templates** on folder, checklist and reconciliation rows: `User not found`, `User without entity access`, `Folder "<name>" not found`, `No match for <account>`, `already has a reconciliation configured`, `can have only one account`, `At least one GL account is required`, `Threshold cannot be negative`, `Duplicate folder name in import`, `Name cannot contain special characters`. Fix the cells there or in the workbook.
- Folders: `No folder template named "<name>"`, `A folder selected for deletion contains recurring agent tasks`, `Keep the folder for "<task>"`, `A folder with an active checklist or reconciliation cannot be deleted.` Point the Folder cell at a real folder, or keep the folder until its templates are gone.
- Row shape: `exactly one of Task Description or Subtask Description is required`, `subtask is orphaned`, `Add subtask "<description>" from the task itself`, `A Task Description is required on the first row`, `Step numbers must start at 1`. Regroup and renumber the rows; add a subtask to an existing task in the app.
- Reconciliations: `Type "<value>" is invalid`, `No active account matches "<account>" for this entity`, `The account on single reconciliation "<name>" cannot be changed`, `Reconciliation type cannot be changed.` Use single or group and accounts from the **Accounts** sheet; delete and re-add a row to change either.
- Assignees, also for Reviewer and Agent: `Preparer Emails and Preparer Due Dates must have matching counts`, `Preparer email "<email>" is invalid`, `Preparer due date "<value>" must be a whole number`, `Preparer: Business-day offset cannot be zero`, `Preparer: Day of month must be between 1 and 31`, `Preparer: Weekday must be between 0 (Sunday) and 6 (Saturday)`, `A person cannot both prepare and review "<task>".`, `Assignee <email> is not available for this subsidiary.`, `These assignees are not available for this entity`. Give each email from `list_assignable_users` one due date in an accepted form.
- Dependencies: `Dependencies must contain existing Template IDs`, `Dependencies contains a duplicate Template ID`, `Dependency <id> for "<task>" is not an available existing task`, `"<task>" cannot depend on itself.`, `"<task>" depends on a task selected for deletion`, `Cannot delete "<task>" while another task depends on it.`, `Dependencies for "<task>" form a circular dependency.` Follow Dependencies.
- Options: `Web Access must be Yes or No`, `Email Policy must be Approval or Auto`, `Use Name <email@example.com> for each approved recipient`, `Each approved email recipient needs a contact name.` Follow Option columns.
- Run By: `Run By must be CloseCore or Own agent.`, `An own-agent task needs at least one Preparer Email.`, `An own-agent task starts manually.`, `An own-agent task has no web access or email settings.`, `Agent tasks run by your own agent are not enabled for this organization.` (or `run by CloseCore`), `Run By can't change on an existing task. Create a new task instead.` Follow Run By; for the last, clear the cell or set it back to the template's kind.
- Triggers, the first eleven shown after `Trigger: `: `Trigger must be Manual, Date:5, Date:15*, Dependencies, or Email`, `Trigger settings are only supported for Email`, `unknown email field`, `Trigger repeats the <field> field`, `Trigger Mailbox: Use 3-40 lowercase letters`, `Allowed senders must be full email addresses.`, `Trigger requires between 1 and 20 allowed senders`, `Trigger Source must be Attachment, Subject, or Received`, `Trigger Offset must be`, `Trigger Pattern is only used with Attachment or Subject`, `Trigger Pattern: <problem>`, `Email requires Mailbox, Senders and Source`, `Email requires a mailbox and complete email settings.`, `Email-triggered start requires the agent email feature.`, `is used by more than one imported task`, `is already used by another task`. Follow Trigger cell with a free mailbox, or choose another trigger.
- Permissions: **Missing permissions** in the review. Folder changes need Manage structure, new tags Manage tags and agent task changes Agent tasks Write; someone with them applies.
- No differences: `No changes detected`. Check that the edits were saved.
- A period folder missing, partway through applying: `The folder for "<task>" is missing in <Month YYYY>. Sync folders and retry.` Sync folders in the app, then download again and reapply what did not save.
