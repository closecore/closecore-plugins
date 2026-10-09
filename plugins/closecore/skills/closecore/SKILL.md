---
name: closecore
description: Use when working through a CloseCore MCP connector with close tasks, review notes, reports, Flux analyses, ERP balances, or connected cloud storage.
---

# CloseCore

Use CloseCore MCP for close data and workflow changes. `CloseCore:tool_name` means the connected server's namespace.

## Operating rules

- Treat current tool schemas as the input contract. Never invent IDs, enum values, or fields.
- Before analyzing a close, require the user to select one entity or all entities and one period. Use `CloseCore:list_entities` and `CloseCore:list_periods` to present exact options when either choice is missing; do not infer the entity or period from task results. If the user already named both unambiguously, treat that as their selection.
- When the user says "current close" without naming a period, suggest the returned period for the previous calendar month as the likely choice, while still asking them to select the entity and confirm the period. If that month is not returned by `CloseCore:list_periods`, present the available periods without guessing.
- If the user selects all entities, tell them the analysis may take a little longer and begin immediately. Do not ask for another confirmation.
- Never construct a period value from the user's wording. Use the returned `slug` for task-list `periods`, storage `period_slug`, and `get_report_data.period_month`. Use the returned `month` for date-valued Flux `period_month` inputs.
- Ask the user to choose among ambiguous matches before mutation. Read the current item first and report the mutation's returned result.
- On connection, authentication, scope, or permission failure, give the exact error and ask the user to connect or re-authenticate. Do not switch data sources silently.

## Present results

Write for accountants and administrators. Lead with the business outcome and translate tool responses into natural language. Keep tool names, response property names, enum values, database terminology, and implementation details internal.

- `signed_off_at: null` becomes "You have not signed off."
- `require_explanation: true` becomes "An explanation is required."
- `creation_source: "mcp"` becomes "Created through MCP."
- `status: "pending"` becomes "Awaiting sign-off."
- An unchanged `updated_at` becomes "The note's last-updated time did not change."
- A write response without `assignees` becomes "The update response did not include the new assignee," followed by a read-back when confirmation matters.

For normal close work, omit database row IDs, audit-log IDs, payload commentary, and unsolicited connector diagnostics. When the user explicitly asks to debug or verify the integration, explain the observed behavior in plain language first; include exact tool names or properties in `code` only when they help diagnose the issue.

## Workflow map

| Goal | Tool sequence |
| --- | --- |
| Supporting IDs | `list_projects`, `list_task_folders`, `list_assignable_users`, `list_tags` |
| Checklist | `list_checklist_items` -> `save_checklist_items` |
| Reconciliation | `list_reconciliations` -> `save_reconciliation` |
| Agent task | `list_agent_tasks` -> `save_agent_task` |
| Journal entry task | `list_journal_entry_items` (read-only in MCP) |
| Review notes | Find task -> `get_review_notes` -> `create_review_note` or `update_review_note` |
| Report or Flux | `list_reports` -> `get_report` -> `get_report_data`; then `list_flux_items` -> `save_flux_item` when requested |
| ERP detail | `list_accounts` / `list_dimensions` -> applicable balance tool |
| Storage | Entity `workflow_instance_id` -> `search_storage`; `browse_folder` only to drill in |

## Analyze a close

After the entity and period are selected:

1. Call `get_close_stats` for the selected period, passing the entity ID for one entity or omitting `entity_ids` for all entities.
2. Include prior periods only when the user requests a trend or comparison. Use returned period slugs rather than constructing them.
3. Investigate meaningful signals with `get_close_stats` grouped by entity, folder, or user. Do not group every dimension by default.
4. Call `list_review_notes` for open blockers in the selected period and entity scope.
5. Use the task-family list tools only to identify the specific checklist, reconciliation, agent, or journal-entry tasks behind an aggregate issue.
6. Report overall progress, overdue and unassigned work, blockers, material concentration by user or folder, and specific evidence-backed actions. Reconcile counts before calculating percentages, and do not dump every task when a smaller set explains the result.

Task lists contain only note summaries. Use `get_review_notes` before replying or resolving. Create a reply with `previous_note_id`; update the exact note ID for content, assignment, tags, or status.

`get_report_data` reads cached v2 data. For another report type, use existing Flux data only if it answers the request; otherwise report the limitation. If several comparison types fit the request, ask which one. Update a Flux item with its returned ID and required identity/configuration fields. Never create a duplicate for an uncertain match.

Storage returns metadata and nullable `cloud_url`, not contents. Verify name, path, and document type; narrow multiple matches or ask. If no result matches, retry with a shorter identifying term. Give an available link and ask the user to drag the file into the session. Otherwise report that no link exists. Never claim MCP downloaded or parsed it.

## Signoff

Call `CloseCore:set_assignee_signoff` only on an explicit request. Resolve the exact assignee row from its task or Flux result. An unambiguous request needs no second confirmation. Signoff changes CloseCore review state only; it never posts to the ERP. Return any `next_action` link.

## Example

To reply to and resolve a reconciliation note: resolve entity and period, list reconciliations, get notes with `item_type: "rec_item"`, create the reply using the root `previous_note_id`, then update that root to `status: "resolved"` only after creation succeeds.
