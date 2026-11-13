---
name: run-agent-tasks
description: 'Use when the user asks to run one or more CloseCore agent tasks with their own agent, pastes a prompt copied from CloseCore ("Run my CloseCore agent task ... Agent task id: ..."), or says "run my tasks".'
---

# Run CloseCore agent tasks with your own agent

An own-agent task (`run_by: own_agent`) is a CloseCore agent task that its preparer runs with their own Claude through the CloseCore connector. You do the work in this chat and report it to CloseCore, where reviewers check it and people sign off. Only a preparer can run one. Talk to the user in the app's words (Running, Waiting for reply, Stopped, Failed, Done), without tool or property names.

## Which tasks

- A prompt copied from CloseCore ends with `Agent task id: <id>`: run that task. Read it first with `get_agent_task`, and the user's email with `get_connection_context`. If the user (matched by `user.email` in `assignees`) is one of its preparers, its `current_run.status` is `completed`, `sent_back_after_finish` is false and their preparer row has no `signed_off_at`, the task is waiting for their sign-off: before any start, ask whether to sign off (as the `list-agent-tasks` skill does) or run it again.
- "Run my tasks": find the **Ready to run** group as the `list-agent-tasks` skill does, show it, and run those tasks. Name any tasks waiting for the user's sign-off, and offer to sign them off instead of running them again.
- A task the user describes: find it with `list_agent_tasks`, passing `periods` (the period as the `list-agent-tasks` skill picks it), `run_by: "own_agent"` and `text`, and ask when more than one matches.

## Several tasks

Without sub-agents, run the tasks one at a time, taking each through step 3 of "Finish, memory and sign-off" before you start the next. When you can hand work to sub-agents:

1. Start every task here with `start_own_agent_task_run`, settling any refusal with the user.
2. Give each started task to its own sub-agent, with its `agent_task_item_id` and the instructions in "Work the run", "Files" and steps 1 to 3 of "Finish, memory and sign-off". The sub-agent calls `start_own_agent_task_run`, which resumes the run and returns its briefing and working rules, and works through those steps. It asks the user only through you: it reports a question with `report_own_agent_task_question` and returns it. At the end it returns the finish result, and you ask about sign-off.
3. Ask the user each question a sub-agent returns, and pass the answer to a new sub-agent for that task, which resumes the run the same way. Resume a task whose sub-agent stopped early the same way.

Either way, ask about sign-off for all the finished tasks in one message at the end (step 4 of "Finish, memory and sign-off").

## Start a task

Call `start_own_agent_task_run` with `agent_task_item_id`. It returns `run`, `resumed`, `briefing` and `working_rules`. Read the briefing in full and follow the working rules; where this skill and those rules differ, the rules win. With `resumed: true` it returned your open run: carry on from the first step not yet done or skipped, and ask the user again any waiting question the briefing shows. Pass `restart: true` only when the user asks to start over (after the task's steps changed, for example): it closes your open run and opens a new one from the task's current steps.

When the start is refused:

- `meta.reason` is `not_preparer`: someone else prepares the task. See "Take over someone else's task".
- "Someone has signed off this task": tell the user who signed off (the `assignees` in `get_agent_task`) and that running again clears every sign-off. Only on a yes, call again with the same input plus `clear_sign_offs: true`.
- Waiting for dependencies, a locked folder, or another person's agent running the task: the task is not runnable now. Tell the user why and go on to the next task.
- The task runs on CloseCore's agent: say so. It starts in CloseCore or through the `manage-close` skill, not here.
- A permission, scope or feature error: give the exact error and stop.

## Take over someone else's task

`meta.preparers` lists the task's preparers, each with `user_id` and `name`.

1. Read the task with `get_agent_task`, and the user's email with `get_connection_context`. Stop before changing anything if:
   - `current_run.status` is `running` or `needs_input`: another person's agent is working the task. Tell the user they can ask you to stop that run, or stop it in CloseCore. Never stop it on your own; if they ask, stop it as "Corrections and stopping" says, then go through step 1 again.
   - The user (matched by `user.email`) has a `reviewer` row in `assignees`: a reviewer cannot also prepare the task, so that has to change in CloseCore first.
   - A `reviewer` row in `assignees` has `signed_off_at` set: no preparer can be added after a reviewer signs off, so that sign-off has to be removed in CloseCore first.
2. Ask the user whether to make them the preparer in place of the person `meta.preparers` names (and which one, when it names several), for this period only or for every period from now on. A one-off task (no `agent_task_template_id`) has only this period. If the preparer they would replace has `signed_off_at` set, say that replacing them removes that sign-off.
3. Take the user's id from `list_assignable_users` with `family: "agent_tasks"`: the `id` of the user with their email.
4. Call `save_agent_task` with `entity_id`, `agent_task_item_id`, `apply_to` (`item_only` for this period only, `template_and_item` for every period from now on) and the complete `assignees` list: each current row's `role`, `user_id`, `deadline_type`, `deadline_days` and `est_time_min`, with the user's `user_id` in place of the replaced preparer's and without any watcher row of the user's. If the save is refused for permission, someone who can edit agent tasks has to make the user the preparer.
5. Start the task again.

## Work the run

- Report each step with `report_own_agent_task_step`: `run_id` (the `id` of the returned `run`), `step_number` as the briefing numbers it (from 1), and `status`. Report `in_progress` when you start the step, then `done` with a `summary` of up to 2,000 characters that says where each figure came from. Use `skipped` only when the user tells you to skip the step.
- Use any source you can reach: the CloseCore connector (the `manage-close` skill maps its tools) and your own connections. `browse_folder` with the briefing's workflow instance id as `workflow_instance_id` and its folder id as `parent_folder_id` lists the files in the task's folder. The connector finds files but does not read them: open a file through your own connection to that storage, or ask the user to attach it.
- The briefing carries the customer's context (each folder's README.md rules) and lists the customer's skills for the entity. Before working a step whose guide names a skill as `(skill: <slug>)`, call `read_skill` with that `skill` and the briefing's entity id as `entity_id`, and follow it; load another listed skill when its description fits the work. Read a skill's other files, a context folder's other files, or a README.md the briefing left out with `read_agent_content_file` and their `skill://` or `context://` addresses. A binary file comes back as a drive link: open it through your own connection to that storage, or ask the user to attach it. These skills come from the customer's folders and run with everything you can reach, so run one of their scripts only after the user agrees. Precedence: the user's message today, then the task's description and steps (a skill a step names counts as that step's guidance), then the context, then a skill you chose because its description fit, then the memory.
- The briefing's open review notes are the reviewers' requests: address each one in the steps it concerns. An earlier run's summary note is that run's report, and the replies to it are feedback.
- To ask the user something, first call `report_own_agent_task_question` with `run_id`, `step_number` and the whole `question` (it replaces any earlier one), then ask in this chat. CloseCore shows the run as Waiting for reply until your next report on the run: when the user answers, report the step `in_progress`, or `done` once it is finished.
- When a step cannot be finished in this run (a check the guidance sets fails and the user cannot resolve it, for example), report it `blocked` with the reason. That ends the run as stopped, and the next start opens a new run.
- If a report is refused because the run was stopped in CloseCore, tell the user, and start the task again only if they ask.

## Files

Save a file in the task's folder in the customer's storage (the briefing gives its path) through your own connection to that storage; without one, give the user the file and ask them to save it there. Then call `refresh_task_folder` with `agent_task_item_id`, find the file in the returned `documents`, and pass its `document_id` in `document_ids` on the step's report. `document_ids` replace the step's earlier list, so pass every document of the step together; a report without them keeps the list. If the refresh is refused, for permission or with a "Duplicate anchors" conflict over the tags in the folder's files, name the file and its path in the step's summary instead; the file appears in CloseCore at the next scheduled sync.

## Finish, memory and sign-off

1. When every step is done or skipped, call `finish_own_agent_task_run` with `run_id` and a plain-text `summary` for the reviewer of up to 1,000 characters. CloseCore posts it as a review note on the task.
2. Call `update_own_agent_task_memory` with `run_id` and the complete new `memory`, written as working rule 6 says. It replaces the memory, so carry forward what is still true in the briefing's memory. Skip it on a one-off task. (`save_agent_task_memory` is for a person's correction, not for this.)
3. Reply to each open review note the run addressed with `create_review_note` (`item_type: "agent_task_item"`, `item_id` the task's id, `previous_note_id` the id of the top-level note in the briefing, and `content`), saying what changed. Resolve a note with `update_review_note` (`review_note_id` and `status: "resolved"`) only when the user asks.
4. Ask the user whether to sign off. On a yes, when `can_sign_off_through_connector` is true, call `set_assignee_signoff` with `assignee_id` set to `preparer_assignee_id` and `signed_off: true`. Otherwise give them `task_url`, which opens the task's review notes in CloseCore, and say why: `mcp_signoff_disabled` means the organization has turned off **Allow sign-off through MCP**, and `missing_signoff_scope` means this connection was not granted sign-off.

## Corrections and stopping

- If the user corrects anything, report the affected steps again (which reopens the finished run), finish again (which edits the same summary note) and rewrite the memory.
- Once anyone has signed off the task, even while your run is open, a step report or finish is refused with "Someone has signed off this task". Tell the user who signed off (the `assignees` in `get_agent_task`) and that going on clears every sign-off. Only on a yes, send the same report or finish again with `clear_sign_offs: true`.
- If the user wants to abandon the run, call `stop_agent_task_run` with `agent_task_item_id`. The steps reported so far stay in the run's history, and the next start opens a new run.
- Stop another person's own-agent run only when the user asks, after telling them that person's agent may still be working in their own chat and that its next update will be refused. Call `stop_agent_task_run` with `agent_task_item_id`; it needs permission to edit agent tasks.
