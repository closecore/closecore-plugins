---
name: author-agent-task
description: Use whenever an agent task's description or steps are written or revised, for an import workbook or for a direct save through the CloseCore connector. Covers the interview, human steps, skill mentions and the agent's limits. Not for checklist or reconciliation rows and not for running tasks.
---

# Author a CloseCore agent task

Unless a task is set to run by **Your own agent**, CloseCore's agent is the preparer on every run of an agent task. It works the steps strictly in order through specialists that read the ERP, work with files, read close data or (with web access on) look up public reference data, and asks in the task's conversation instead of guessing; people sign off before its files reach their work folder. For a task its preparer's own agent runs (`run_by: own_agent`), "Steps for the preparer's own agent" below takes the place of the skill mentions and input channels the sections before it describe.

This skill writes the description and steps and hands them back; it saves nothing. `references/boundary-and-skills.md` holds the boundary, the skill distinctions and the input channels, and `references/archetypes.md` the human-step phrasing and five skeletons.

## Before drafting: look up the customer's setup

Through the connector, call:

1. `get_connection_context` and `list_entities`: the organization, the subsidiary and its ERP.
2. `list_available_skills` with the subsidiary's `entity_id`: its skills and mention tokens, the customer's own skills first. For a task the preparer's own agent runs, only the customer's own skills (category `custom`) apply.
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
6. Who runs it: CloseCore's agent or, where the organization has the option, the preparer's own agent (and then which person prepares it). This is fixed once the task is created, so for an edit it is already decided.

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

A customer's own skills, kept in their cloud folders, are mentioned like the built-in ones, and one whose name gives a built-in's mention replaces it. Every run has every skill the subsidiary has, so the agent also applies a skill whose description fits a step that does not mention it, but that use ranks below the customer's context (the rules in each context folder's README.md, which the agent reads on every run), so mention the skill a step must follow.

Guidance configures nothing: a recipient, web access or a trigger written there also goes into the settings. When editing, keep unrelated instructions, the person's decisions and existing mentions as they are. Limits for every task: 1 to 30 steps, a title on each step of up to 300 characters, guidance of up to 3,000 characters, a description of up to 3,000 characters (300 for a training draft), and no HTML. The description's first line is the task's name in notifications and, cut to 60 characters, its folder under **Agent Files**, so keep that line short.

For a direct save with a separate title and briefing, pass the title, a newline, and the
briefing together in `description`. New tasks show the full briefing. Pass
`collapse_description: true` to `save_agent_task` only if the user asks to show just the
title initially. A run always receives the entire briefing.

## Steps for the preparer's own agent

A task set to run by **Your own agent** is worked by its preparer's own Claude through the CloseCore connector, from the preparer's own chat. The steps are its briefing, so write them for that agent:

- Mention only the customer's own skills (category `custom` in `list_available_skills`). The preparer's agent loads one with `read_skill` before its step. Built-in skills and agent reconciliations are CloseCore's agent's procedures, and the preparer's agent sees only their names, so write that method out instead; drop those mentions when you adapt an archetype.
- Any data source is fair: what the CloseCore connector reads (ERP balances and detail, close data, the connected storage's folders) and whatever the agent reaches through its own connections. Name each source, so its reports can say where each figure came from.
- A human step says what the agent asks its user in the chat and how it checks the answer. These tasks have no conversation in CloseCore and send no email from it.
- Files the task produces go in the task's folder in the customer's cloud storage, and the agent refreshes the folder through the connector. Nothing waits under **Agent Files** for acceptance.
- The preparer is a person, and the task always starts manually, with no web access or email settings; of the defaults below, only the human reviewer applies.

Rehearse the run the same way, naming each step's source and where the agent would ask its user. `references/boundary-and-skills.md` says what else changes for these tasks.

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
3. Suggested settings: who runs it (and, for the preparer's own agent, the preparer), folder, reviewer and due date, frequency, dependencies and, for CloseCore's agent, start mode, web access, approved email recipients (name and email) and email policy.
4. Each human step with its channel and, for email, its recipient.
5. Anything still to confirm.

Reached directly, continue with the `author-import` skill for a workbook or the `manage-close` skill for a direct save.
