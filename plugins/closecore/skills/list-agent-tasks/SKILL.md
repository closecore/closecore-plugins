---
name: list-agent-tasks
description: 'Use when the user asks which CloseCore agent tasks they can run with their own agent, or what is waiting on them. Lists their own-agent tasks for a period in three groups: Ready to run, Waiting for your sign-off, Not runnable now.'
---

# List your own-agent tasks

An own-agent task is a CloseCore agent task set to run by **Your own agent** (`run_by: own_agent`): its preparer runs it with their own Claude through the CloseCore connector. This skill lists the tasks the user prepares and changes nothing; running them belongs to the `run-agent-tasks` skill.

## Find the user and the period

1. Call `get_connection_context` for the user's email, then `list_assignable_users` with `family: "agent_tasks"` and take the `id` of the user with that email. A user who is not listed cannot prepare agent tasks: say so and stop.
2. Call `list_periods`. Use the period the user names; otherwise the current close, the period for the previous calendar month. Pass the returned `slug`, never one built from the user's words, and say which period you used. If that month is not returned, show the returned periods and ask.
3. Call `list_entities` to name each task's entity.

## Group the tasks

Call `list_agent_tasks` with `periods: [<slug>]`, `run_by: "own_agent"` and `preparer_user_ids: [<the user's id>]`. In each item, the user's preparer row is the `assignees` entry with their `user_id` and `role: "preparer"`, and the run is open when `current_run.status` is `running` or `needs_input`. Each task goes in the group of the first rule that fits, with that rule's reason:

1. `status` is `complete`: **Not runnable now**, reason "Complete".
2. `blocked` is true, because a task it depends on in this period is not complete: **Not runnable now**, reason "Waiting for a dependency".
3. The run is open and `current_run.started_by_user_id` is someone else's: **Not runnable now**, reason "<name>'s agent is running it" (the runner's name from `assignees`, or "Another person's agent is running it").
4. `sent_back_after_finish` is true, because someone other than the runner left a review note after the last finish and the last preparer sign-off: **Ready to run**, reason "Sent back". If the user has signed off, its start asks to clear the sign-offs.
5. The user's preparer row has `signed_off_at` set, so the task waits on others: **Not runnable now**, reason "You signed off".
6. `current_run` is null: **Ready to run**.
7. The run is open and the user started it: **Ready to run**; a start resumes it.
8. `current_run.status` is `stopped` or `failed`: **Ready to run**; a start opens a new run.
9. Any other task, whose run is `completed`: **Waiting for your sign-off**.

`sent_back_after_finish` can stay true once the task is complete, which is why rule 1 comes first. `blocked` misses a dependency that has no task in this period, so a Ready to run task can still be refused at its start for dependencies; the `run-agent-tasks` skill then reports it as not runnable now.

## Show them

Show a short table per group, leaving out empty groups, with the columns Task (the first line of `description`), Entity, Status, Reason and ID (`id`, which a run needs). Status is the app's label for the current run, whoever ran it: Not started when there is none (Blocked when `blocked` is true), otherwise Running, Waiting for reply (`needs_input`), Stopped, Failed or Done (`completed`). Reason is the rule's reason, blank where the rule gives none. Leave property names and enum values out of the reply.

Then offer to run the Ready to run tasks with the `run-agent-tasks` skill, and to sign off the ones waiting for the user's sign-off. If the user wants to check the work first, show each step's summary from `current_run.steps`. Sign off only the tasks the user confirms: call `set_assignee_signoff` with `assignee_id` set to the `id` of their preparer row and `signed_off: true`. If the call is refused because the organization has turned off sign-off through the connector (`mcp_signoff_disabled`) or the connection lacks the sign-off scope, tell the user to sign off on the task in CloseCore.
