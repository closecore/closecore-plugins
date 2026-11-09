---
name: author-agent-task
description: Use whenever an agent task's description or steps are written or revised, for an import workbook or for a direct save through the CloseCore connector. Covers the interview, human steps, skill mentions and the agent's limits. Not for checklist or reconciliation rows and not for running tasks.
---

# Author a CloseCore agent task

CloseCore's agent is the preparer on every run of an agent task. It works the steps strictly in order through specialists that read the ERP, work with files, read close data or (with web access on) look up public reference data, and asks in the task's conversation instead of guessing; people sign off before its files reach their work folder.

This skill writes the description and steps and hands them back; it saves nothing. `references/boundary-and-skills.md` holds the boundary, the skill distinctions and the input channels, and `references/archetypes.md` the human-step phrasing and five skeletons.

## Before drafting: look up the customer's setup

Through the connector, call:

1. `get_connection_context` and `list_entities`: the organization, the subsidiary and its ERP.
2. `list_available_skills` with the subsidiary's `entity_id`: its skills and mention tokens.
3. `list_task_folders`: the task folders, and which are restricted.
4. `list_agent_tasks` for the period (slug from `list_periods`): existing tasks to borrow from and not duplicate.
5. `list_assignable_users`: who can prepare or review, by email.
6. `list_accounts`, when a step touches accounts.

Live sources (the connector's skill list, the workbook's **Instructions** and **Skills** sheets, the app's preview) win over the reference files. Without a connection, say so, draft from the **Skills** sheet and the person's answers, and mark every customer-specific value as one to confirm.

## Ask for an example first

Ask whether the person has an example: a workbook of tasks they like, a task they run (for an edit, the task itself), or a before-and-after of a fixed task. Read it first and match its detail, names and conventions over the archetypes; their stated or remembered preferences win over this skill's defaults, so offer to remember new ones. A finished workpaper shows a result, not how it was made, so compare earlier periods' versions and ask where unexplained values come from.

## The interview

Ask these in one message, proposing answers the lookups already give; for an edit, only those the change touches:

1. The deliverable and where it lands: a new workbook, a new version of an existing workbook, a journal entry or Sage Intacct import file, an email, or a conclusion in the conversation.
2. The inputs and where each lives: a file in the task folder or elsewhere, ERP data, an email someone sends, or something only a person knows.
3. For each input the agent cannot fetch, who supplies it and how: in the task's conversation, or by email to a named recipient.
4. The checks that make the result right (totals that tie, ranges, counts) and what happens when one fails (usually, block the step and describe why).
5. Who reviews, how often the task recurs, and when it is due.

Non-ERP data is not automatically manual: files in connected cloud storage, last period's workbook and emailed replies are within reach, so check before making an input a human step. Never invent policy, tolerances, recipients or file names; ask.

## Two kinds of step

**An agent step**: the agent does the work, with the CloseCore skills it relies on named in the guidance. Anything on the agent's "cannot" list becomes a human step or leaves the task; quote the boundary in `references/boundary-and-skills.md` so everyone hears the same reason.

**A human step**: a person supplies what the agent cannot fetch. Place it before the step that uses it and phrase it with a template from `references/archetypes.md`: what to request from whom, to ask and wait, and how to check the answer. A human step written as an instruction to a person ("The controller uploads the statement") may not park the run, because nothing in it tells the agent to ask, wait and check the answer. Name its channel: the task's conversation (`@request-clarification`; the run shows **Waiting for reply** until someone answers), or an email to one approved recipient (`@send-email`, then `@read-email-thread`).

Review and approval are not steps; the final sign-off covers them. Steps run strictly in order; the agent never reorders them and skips one only with a person's say-so. So: input checks first, human steps before the steps that need them, the deliverable last. A step phrased as verification ("Confirm the trial balance ties") checks evidence and never does the work, so never phrase work as a check.

## Write the guidance

Each agent step's guidance covers, without repeating the title:

- **Inputs and scope**: each source by a meaningful name and how to pick this period's version ("the bank statement PDF for the task's period in the task folder"), plus the accounts, sheets or columns that matter. Use the task's period, never a test month, and no document IDs, links or folder paths: each period runs in a new folder.
- **Method and business rules**: calculations, matching rules, exclusions and tolerances, as the person stated them.
- **Output and completion check**: what the step leaves for the next step or the reviewer, and the check showing it is done.
- **Exceptions**: when to flag a finding and carry on, and when to stop and ask.

A step can mention several skills, and a summary or reasoning step none. In a workbook, copy the mention from the **Skills** sheet's **Mention to copy** column (`@slug`, lowercase), keeping any full `@[Name](agent-rec:slug)` token whole, with no backticks or leading apostrophe (whatever the **Instructions** sheet suggests), which stay as text. A capital letter, a typo, or a letter, digit, hyphen or underscore touching the mention leaves it plain text, with no error. Through the connector a bare `@slug` stays plain text: use the `mention` token `list_available_skills` returns, and for an agent reconciliation, which no tool lists, build `@[Name](agent-rec:slug)` from its name and slug, which an existing task's steps, a **Skills** sheet or the person (on **Agent Reconciliations**) can give.

Guidance configures nothing: a recipient, web access or a trigger written there also goes into the settings. When editing, keep unrelated instructions, the person's decisions and existing mentions as they are. Limits for every task: 1 to 30 steps, a title on each step of up to 300 characters, guidance of up to 3,000 characters, a description of up to 3,000 characters (300 for a training draft), and no HTML. The description's first line is the task's name in notifications and, cut to 60 characters, its folder under **Agent Files**, so keep that line short.

For a direct save with a separate title and briefing, pass the title, a newline, and the
briefing together in `description`. New tasks show the full briefing. Pass
`collapse_description: true` to `save_agent_task` only if the user asks to show just the
title initially. A run always receives the entire briefing.

## Rehearse the run

Before handing back, narrate the run, two or three lines per step: which specialist acts, or the agent itself for the conversation and email, and with which skill; the inputs and where each comes from; what the step leaves behind; and where the run would park (**Waiting for reply**, **Email approval needed**, **Waiting for email**, or a step blocked by a failed check). Rewrite or split any step the narration cannot explain; an input with no source is a missing human step.

## Defaults to suggest, not impose

Suggest each once; drop it when the person works differently, since their safeguards may live outside CloseCore:

- A human reviewer with a due date.
- **Email policy** set to **Approval** until the task has earned trust.
- **Web access** off unless a step needs public reference data.
- A **Manual** start for the first period, so a person starts and reviews the first run before a date or email trigger takes over.
- A training workspace only while a task is still being shaped, before its template row exists; never publish a draft duplicating a template task, which would create a second recurring task.

## Hand the steps back

Return to the skill that called you:

1. The description: the objective and deliverable, as the agent's assignment.
2. The numbered steps, each with a title and guidance.
3. Suggested settings: folder, reviewer and due date, frequency, start mode, dependencies, web access, approved email recipients (name and email), email policy.
4. Each human step with its channel and, for email, its recipient.
5. Anything still to confirm.

Reached directly, continue with the `author-import` skill for a workbook or the `manage-close` skill for a direct save.
