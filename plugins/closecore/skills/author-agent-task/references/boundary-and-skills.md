# The agent's boundary, skills and input channels

Quote the two lists below when you turn part of a request into a human step or leave it out of the task, so every person hears the same reason. They describe a run of an agent task, which always works in one subsidiary. A training test sees less: only its workspace and the earlier periods' folders for its folder template, with email simulated.

## What the agent can do

- Read files in the task folder and elsewhere in the subsidiary's connected cloud storage, including earlier periods' folders: PDFs and images (scans and handwriting too), xlsx files, Google Sheets, CSV and text files.
- Read accounts, dimensions, balances and transaction detail through the subsidiary's ERP connection, and fetch standard reports, the ones the ERP calculates itself, such as a P&L, balance sheet or aging as of a date, where the integration offers them.
- Build new workbooks, and new versions of existing xlsx workbooks, with formulas a reviewer can trace. They wait in the run's folder under **Agent Files** until people accept them at sign-off.
- Prepare a journal entry as a table that becomes a draft for a person to review and post, or a Sage Intacct import file for the accountant to import.
- Run a saved agent reconciliation, which matches two sources line by line, each a file or the ledger, and review its matched and unmatched results.
- Read close data: other tasks and their status, review notes, custom reports built in CloseCore, journal entry drafts, close progress and task history.
- Email one approved recipient at a time, wait for the reply, and read it and its attachments.
- Ask in the task's conversation and wait for the answer, including files a person attaches.
- Remember a settled convention for the next period of a recurring task.
- Look up public reference data, such as an FX rate, when the task has **Web access** on.

## What the agent cannot do

- Sign in to another system, portal or website, or reach any system that has no CloseCore connection.
- Read bills or vendors from an AP connection.
- Post or change anything in the ERP. The one exception: on Sage Intacct, with posting enabled on the connection, it can attach task files to a journal entry that already exists.
- Email anyone who is not an approved recipient, copy anyone in, or send attachments.
- Move, delete or rename existing files, or change one other than as a new version that a person accepts.
- Change other CloseCore records: other tasks, review notes, settings, other people's sign-offs, or its own steps.
- Work in another subsidiary.
- Decide a value, policy or threshold it was not given. It asks instead.
- Use the web when **Web access** is off, or send anything of the customer's out to the web.

Anything on the second list becomes a human step or leaves the task.

## How a run works

The agent reads the description and steps, checks the task's conversation, and works the steps strictly in order. It hands each piece of work to a specialist, which matters when you rehearse a run:

- **ERP**: accounts, dimensions, balances, transaction detail and standard reports, read-only. Balances, transaction lines and reports reach it as files that it analyzes with code; it reports the figures and hands transaction lines or a report meant for a workbook to the workbook specialist as a file.
- **Workbook**: finds, opens and writes files, and is the only specialist that produces them. It has no tool that returns ERP data; transaction lines and reports reach it as the ERP specialist's file, which it copies into the workbook with code. It moves rows from a stored CSV, TSV or xlsx file only by importing them, never by retyping; lines it reads itself from a PDF or image it can write.
- **Close data**: CloseCore's own tasks, notes, custom reports, journal entry drafts, progress and users. A custom report's figures reach it as a file that it analyzes with code. It changes nothing.
- **Web**: public reference data only, on a task with **Web access** on.

The agent itself asks questions, sends email, runs agent reconciliations, keeps memory and signs off as preparer once every step is done. A step it cannot complete or check parks the run with a question; it never reports progress it did not make.

## Choosing skills

`list_available_skills`, or the workbook's **Skills** sheet, is the catalog: each skill's name, what it does, what it needs and its mention. The distinctions it does not draw:

- **Update a workbook** makes a new version of an existing xlsx file under its name, which a person accepts at sign-off; **Build a workbook** makes a new file; **Roll forward a workbook** only copies and relabels last period's file, so later steps do the real work.
- **Read a spreadsheet** reads xlsx files and Google Sheets, **Read a CSV** reads CSV, TSV and text files, and **Read a PDF** reads PDFs and images, scans and handwriting included.
- **Get transaction detail** puts up to 50,000 lines in a file the agent works on with code, to total, group and check them, pick out the lines a step asks for, or copy them into a workbook; name the account and narrow by dimension or amount when the volume is large.
- **Fetch standard reports** gets a standard report, one the ERP calculates itself, such as a P&L, balance sheet, cash flow statement or aging as of a date. When a step needs one, mention it rather than **Get balances** or **Get transaction detail**: on an ERP that offers standard reports, the agent never rebuilds one from balances or transactions. A custom report, one the team built on CloseCore's **Reporting** page, needs no skill: name the report in the step.
- **Run a reconciliation** comes with any mention of an agent reconciliation, so mention the agent reconciliation itself.
- **Request clarification** belongs on every human step that uses the conversation.
- The ERP skills stay listed even without an ERP connection, so check the subsidiary's ERP in `list_entities`; a skill whose feature, or whose specific ERP, the subsidiary lacks is not offered.
- A mention of a skill that no longer exists makes the agent stop and ask at run time.

## Channels for a person's input

### The task's conversation

- The step tells the agent to ask (`@request-clarification`). The agent posts all its open questions in one message, the task's assignees are notified, and the run shows **Waiting for reply**.
- Ask someone on the task as a co-preparer, reviewer or watcher. Nobody else is notified that the run is waiting.
- A person answers in the task's **Conversation** tab, or through the CloseCore connector. The reply resumes the run at once.
- A reply can carry attached files. The agent reads PDFs, images, spreadsheets and CSV files, so ask for one of those.
- Use it for what someone on the team knows or holds: an approved rate, a judgment call, a report from a system the agent cannot reach.
- Review notes are not a channel. The agent reads them only as information about the close, so instructions belong in the steps or the conversation.

### Email

- The step tells the agent to email a named approved recipient (`@send-email`), wait for the reply, and read the thread (`@read-email-thread`) before continuing.
- It needs the agent email feature and the recipient on the task's **Approved email recipients** with a contact name and email.
- Each email goes to one recipient, with no CC or BCC and no attachments. Attachments on the reply are saved to the task and can be read.
- The task's **Email policy** decides whether a person sees the draft first. **Approval** means someone approves each draft (the run shows **Email approval needed**); **Auto** sends to approved recipients without review. While the agent waits for the reply, the run shows **Waiting for email**.
- The agent sends a follow-up only on a date the guidance gives; otherwise it keeps waiting.
- Use it for someone outside CloseCore, such as a bank contact, a vendor or a manager who works in another system.
