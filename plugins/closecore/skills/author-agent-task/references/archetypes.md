# Human step phrasing and archetypes

Mentions appear in backticks here; write them without backticks, in the form the `author-agent-task` skill gives for a workbook or the connector.

## Human step phrasing

Use one of these templates for every step in which a person supplies something. Fill each angle bracket with the customer's own words.

For an answer or a file through the task's conversation:

```text
Request <what> from <who> in this task's conversation: <what to ask, with the period and the expected form>. Wait for the answer. Verify <checks> before continuing; if it is missing or fails the checks, leave this step blocked and say what is wrong. Use @request-clarification.
```

For a reply by email:

```text
Email <Name> (an approved recipient) to request <what>, wait for the reply, and read the thread before continuing. In the email, ask for <what to ask, with the period and the expected form>. Verify <checks> before continuing; if it is missing or fails the checks, leave this step blocked and say what is wrong. Use @send-email and @read-email-thread.
```

Filling them in:

- **Who**: for the conversation, someone on the task as a co-preparer, reviewer or watcher.
- **What and the expected form**: the item and its period, and how it should arrive (a figure with its basis, a PDF, or an xlsx or CSV file with named columns).
- **Checks**: concrete tests the agent can run, such as the period on the document, a total that ties to another source, a value inside a range, or required columns present.
- **No fallback values**: the agent never substitutes last period's value or a default unless the person has confirmed a standing rule that allows it. Write that rule into the step when it exists.
- **Placement**: before the first step that uses the answer. When one person supplies several things, ask for them in one step.

A filled-in conversation step, for example:

> Request this period's approved bonus accrual rate and its basis from the controller in this task's conversation: the rate as a percentage of eligible salaries and the document that approves it. Wait for the answer. Verify that the rate is for the task's period and between 0% and 25% before continuing; if it is missing or fails the checks, leave this step blocked and say what is wrong. Use @request-clarification.

## Archetypes

Five skeletons drawn from the skill catalog. Each step shows its kind (agent step or human step) and the skills it relies on. Adapt every step; never paste a skeleton unchanged.

### Roll forward and refresh from the ERP

1. **Roll forward the <workbook>** (agent step; `@roll-forward-workbook`). Copy last period's <workbook> from the prior period folder into this period. Clear <the input cells, for example the Current Month column on the Detail sheet>. With nothing named, nothing is cleared.
2. **Pull this period's balances** (agent step; `@get-balances`, plus `@balances-by-dimension` when the workbook needs a balance split by department or location). The period-end balances of <accounts, by number> for the task's period. They go into the workbook as individual figures; a schedule that lists transactions needs a ledger export saved as a file.
3. **Update the workbook** (agent step; `@workpaper-formulas`). On the rolled-forward copy, <carry last period's closing balances to this period's opening balances>, enter this period's figures on <sheet>, and keep every derived figure a formula.
4. **Tie the workbook to the ledger** (agent step; `@tie-out-tags` when the workbook has #cc tags). Confirm that <the workbook's total for each account> equals the ERP balance within <tolerance>. If a difference remains, leave this step blocked and describe it.
5. **Report** (agent step). The balances, the tie-out result and anything still open.

### Reconcile an account to a statement

1. **Get this period's statement** (human step, conversation or email; `@request-clarification`, or `@send-email` and `@read-email-thread`). Use a template above to request <the bank> statement for <account> for the task's period as a PDF, checking its period and account number. When someone always saves the statement to the task folder, make this an agent step instead: find it with `@find-files`, and ask in the conversation if it is missing. Where agent email is set up, a statement that a bank emails can also start the task (**Start Mode**: **Automatic when an email arrives**); its attachments are saved to the task folder before the run begins.
2. **Read the statement** (agent step; `@read-pdf`, or `@read-spreadsheet` or `@read-data-file` for an export). The opening and closing balances and the transactions. Check that the transactions take the opening balance to the closing balance, and flag any figure that cannot be read.
3. **Pull the ledger balance** (agent step; `@get-balances`). The balance of <GL account> at period end.
4. **Reconcile** (agent step). To match the statement line by line against the ledger, mention the saved agent reconciliation for <account>. Without one, use `@build-workbook` or `@update-workbook` with `@workpaper-formulas` to tie the statement's closing balance to the ledger balance, finding the reconciling items by matching the statement against <a ledger detail export a person saves in the task folder> by <rule, for example amount and a date within 3 days>, listing the reconciling items and computing the unexplained difference. With neither an agent reconciliation nor an export, list the ledger entries behind the difference in the step with `@transaction-detail` and ask the person which are reconciling items.
5. **Check and report** (agent step). The unexplained difference is <zero, or within a stated tolerance>. When the reconciling items are rows in a workbook (an agent reconciliation's unmatched sheets, or the unmatched lines of a ledger export), also check that none is older than <n> days. If a check fails, leave this step blocked and describe the problem; otherwise report the reconciled balances and the reconciling items.

### Prepare a journal entry from a schedule

1. **Read the schedule** (agent step; `@find-files`, `@read-spreadsheet`). This period's <schedule, for example the prepaid amortization schedule> in the task folder, and its amounts for the task's period by <account and department>.
2. **Confirm the accounts** (agent step; `@list-accounts`, `@list-dimensions`). Every account number and dimension value the entry will use exists in the ERP. If one does not, leave this step blocked and list it.
3. **Tie the schedule to the ledger** (agent step; `@get-balances`). <The ERP balance of the account, adjusted by this entry, equals the schedule's closing balance> within <tolerance>. If it does not, leave this step blocked and describe the difference.
4. **Prepare the entry** (agent step; `@prepare-je`, or `@prepare-intacct-je-import` on Sage Intacct). Debit <account> and credit <account> for <the period's amounts>, dated <the period end>, with the description <text>. Debits must equal credits.
5. **Report** (agent step). The entry's lines and total, and the tie-out result.

The agent never posts: after sign-off, a person posts the journal entry draft or imports the Intacct file.

### Request data by email, validate it, then compute

1. **Check the inputs on hand** (agent step; `@find-files`, `@read-spreadsheet`). The <workbook> and <report> for the task's period are in the task folder. If either is missing, ask in the conversation.
2. **Request the <data>** (human step, email; `@send-email`, `@read-email-thread`). Use the email template: <Name>, the <data> for the task's period as a CSV or XLSX attachment with <columns>, and the checks that the attachment is present, readable, for the task's period and has those columns. Use the attachment, not the message text, as the source.
3. **Import and compute** (agent step; `@update-workbook`, `@workpaper-formulas`). Import the data into <sheet> of the <workbook> without retyping rows, recalculate, and check <for example: dates inside the period, totals that tie to <source>, every <employee or item> matched>. If a check fails, leave this step blocked and describe it; never change source data to make a check pass.
4. **Report** (agent step). <The totals>, each check's result and anything still open.

### Review a workbook and flag anomalies

1. **Find the workbooks** (agent step; `@find-files`, `@read-spreadsheet`). This period's <workbook> in the task folder and last period's version in the prior period folder.
2. **Tie the key figures** (agent step; `@get-balances`, `@tie-out-tags`). <The totals that matter> equal the ERP balances for the task's period, and every #cc tag resolves.
3. **Flag anomalies** (agent step, written as a check). Compare with last period and flag <for example: changes over 10% and $5,000, typed-in figures where a formula is expected, dates outside the period, figures without support>. Report each finding with its sheet and cell, and do not correct the workbook.
4. **Write up the findings** (agent step; `@build-workbook` when the reviewer wants a findings workbook). One line per finding with what to check, or a summary in the conversation.
