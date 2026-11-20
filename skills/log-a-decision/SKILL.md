---
name: log-a-decision
description: Record a decision or change as a Puzzle changelog entry through the Puzzle MCP, with the rationale, the specifications, the status, the owner, and links to the steps and tools it touches. Load when the user asks to "log this decision", "add a changelog entry", "record this in Puzzle", "memorialize this", "write up why we did this", or after any session that scoped a process, picked or dropped a tool, changed the data model, chose an architecture, redesigned a process, planned a build, or answered substantive questions about how the operation works. Also load to update an existing entry's status or links. The changelog is the record of why the map looks the way it does; an entry without rationale is not an entry.
---

# Log a decision

The map shows what the operation looks like. The changelog shows why. Each entry records a decision or a change: what was decided, why, what it touches, who owns it, and where it stands. Entries outlive the conversation they came from, so someone opening the workspace months later can understand the reasoning behind the current state, and the cost tab on an entry is where a change's financial case lives.

Load `read-the-map` first.

## When to write one

Any time a meaningful operational or technical decision has been worked through:

- A process was mapped, scoped, redesigned or retired.
- A tool was chosen, added, replaced, deprecated or migrated away from.
- The data model changed: new records, new fields, a cross-system reference.
- An architecture was set ("HubSpot for demand, Workiz for delivery").
- A build phase was planned or finished.
- A step changed executor (a person's work became an automation or an agent).
- Substantive questions about how the operation works were answered.
- An audit found and fixed gaps.

If the user has just worked through something like this with you, offer: "Want this as a changelog entry on the workspace?" Don't wait to be asked.

## Procedure

### 1. Confirm scope and workspace

Say back the decision in one or two sentences so the user can correct the scope. Name the workspace if more than one is connected.

### 2. Title

The first H1 of the body. Three to eight words, specific enough to identify the decision, never a sentence. "Clio to CasePeer migration", "Intake triage moves to an AI agent", "Order-to-cash current state mapped". Not "Tech stack update", not "We decided to move intake triage to an agent because...".

The title text sits on the `#` line itself: `# Intake triage moves to an AI agent`. A body that starts with `# ` and puts the words on the next line saves an empty heading, and the entry shows as "Untitled". Check this every time a body is rewritten, above all one read back from the API and edited.

### 3. Owner

`list_people` with `query` for the person responsible for the decision or the build. Use their `id` as `person_id`. If the obvious owner isn't on the People canvas, ask; a changelog entry with the wrong owner misleads later readers. When an outside party (a consultant, a vendor) operates the workspace, they may exist as a person record under their company name.

### 4. Status

Exactly one of:

- `idea`: being explored, nothing committed.
- `planned`: decided and scoped, not started.
- `in_progress`: being built.
- `completed`: shipped and running. Set `completed_at` to the date it happened when that isn't today.
- `archived`: superseded or abandoned, kept for the record.

Ask when it isn't obvious. Entries written after the fact are `completed`; entries capturing an option for later are `idea`.

### 5. Steps to link

Every workflow step the decision touches, so anyone viewing those steps sees the entry behind them. If the steps were built or changed in this conversation, their ids are at hand. Otherwise `list_sections` and `list_steps` with `query`, and confirm ambiguous matches with the user before linking. A step id that no longer exists is skipped and reported, not a blocker.

### 6. Tools to link

Every workspace tool the decision involves: introduced, extended, bridged, deprecated. `list_tools` scope `workspace` for the `account_tool_id` (not the catalogue `id`).

### 7. Current state and future state (when the decision changes cost)

On the Optimizer plan an entry can carry its own cost case. When the decision changes how work is done (a redesign, an automation, a tool swap, a removed step), name both sides:

- **Current state:** the sections, or whole Workflow tabs, that run today (typically the section on the Current State tab).
- **Future state:** the sections or tabs that will run after the change (typically its counterpart on the Future State tab).
- **Exclusions:** each side has its own filter, `current_state_exclude` and `future_state_exclude`, each `{statuses, tag_ids}`. Steps that will not run after the change are left out of the future cost, not deleted. Exclude by status first (Archived, Draft); exclude by tag only for a cut no status expresses, such as a phase mapped but not in this change's scope. Leave the current filter empty unless the current side needs one (next point).
- **One map, both sides.** When the change was drawn into the same section or tab as today's work (the proposed steps added in Draft beside the steps that run now), name that section or tab on both sides with different filters: the current side excludes the proposed steps (by their Draft status, or a tag), the future side excludes the steps the change retires (Archived). The MCP accepts a source on both sides only while the two filters differ, because otherwise it would cost the same on each. Separate Current State and Future State tabs need no filter on the current side.

Resolve the ids with `list_sections` (or `list_tabs` for a whole tab) and put them in the proposal. A decision that changes no cost (a naming rule, a shared view, a data field) skips this step.

### 8. Body

Markdown, rendered in the changelog. Use what applies; skip sections that don't:

```markdown
# <Title>

**Status:** <Idea | Planned | In progress | Completed>
**Owner:** <name>
**Decision date:** <Month YYYY>
**Stakeholders:** <who else has a stake>

---

## Summary
Two to four sentences: what was decided and why it matters. Readable on its own.

## The principle (or the core mechanic)
One paragraph: the rule the decision sets. "X owns Y, Z owns W." The before and after of a redesign. Skip if there isn't one.

## Decisions and the rationale behind each
### 1. <Decision>
Context and the decision, two to four sentences.
**Rationale:** why this was the right call, with the trade-off named.
### 2. ...

## Specifications
Field mappings, data model details, system behaviours, the handle table of a build, tables and lists. Skip for a purely strategic decision.

## Open items
Numbered. Who resolves each and what blocks it.

## Phases (if more than one)
- **Phase 1 (<status>):** <scope>
- **Phase 2:** ...

## References
Map sections, documents, recordings, related entries by title and id.
```

Content rules:

- Every decision has a rationale. A list of what changed with no why is a status update, not a changelog entry.
- Name the trade-off. If an option was rejected, say which and why.
- Concede what was conceded. If a concern turned out to be right, record it.
- Capture durable answers. Questions answered in the session belong here if the answers will matter later.
- Reference the map: section names, step names and ids, entities, tools.
- Don't fabricate. Unknown is "unknown"; a gut call is "a judgment call".
- Plain language. No filler, no marketing words.

### 9. Create

`create_changelog_entries` with `body`, `person_id`, `status` (and `completed_at` when backdating). Note the returned `id`.

### 10. Link

`update_changelog_entries` with `add_step_ids` and `add_account_tool_ids`. Report any id that failed; don't block on it.

When step 7 applies, the same call sets `add_current_state_section_ids` or `add_current_state_tab_ids` and `add_future_state_section_ids` or `add_future_state_tab_ids` (create takes no sources), plus `future_state_exclude` and, when the current side needs one, `current_state_exclude`, each with `statuses` and `tag_ids`. Both filters can also go on the create call itself. Each replaces its whole filter; `{}` clears it. For one section or tab on both sides, send the filters in the same call as the sources, or first: a source added to the second side while the two filters still match is refused.

Then read it back: `list_changelog_entries` with the `id` and `include: ["current_state", "future_state", "cost_analysis"]`. `has_current` and `has_future` should both be true. A side that totals zero while it holds steps is missing inputs (durations, frequencies, role rates), not free: report which steps lack them and leave the figure as it is. Never fill an input to make the number move; `cost-a-process` finds the gaps.

When the decision answers an open question or resolves a pain point recorded as a comment on the map, reply on that thread with one line and the entry's title and id, then resolve it (`create_comments` with `parent_id`, then `update_comments` with `resolved: true`). A pain point is resolved only when the process owner (on a client engagement, the client) has given the proposed solution in the thread and the change is Live; until then, reply with the pointer and leave it open. The team that built the change resolves it. On steps the decision changes where a reader would look for the reason, a `decision` comment with the same one line is optional, and only after the user approves it. The comment points; the entry is the record.

### 11. Confirm

Tell the user the title and id, the URL (`/workspaces/<workspace>/changelog/<id>/edit` on the app origin), the status, the owner, the counts of linked steps and tools, and the sections the body contains. When both states are set, add the net from the cost analysis, monthly and annual, and any side with missing inputs.

## Updating an entry

`list_changelog_entries` with `query` or `id` and `include: ["body", "status", "associated_steps.step_id", "associated_tools.account_tool_id"]`. Then `update_changelog_entries` with the fields that change: `status` (set `completed_at` when it becomes completed), `body` (pass the whole body; keep what was there word for word and add the new section), `add_step_ids`, `remove_step_ids`, `add_account_tool_ids`, `remove_account_tool_ids`, and the state fields (`add_` and `remove_` for current and future state sections and tabs, `future_state_exclude`). Add `current_state`, `future_state` and `cost_analysis` to the include before changing them. Never shorten an existing body to make room, and keep the title on the `#` line.

## The quality bar

Someone landing on the workspace six months from now should read the entry and understand not just what was decided but why each piece was decided that way, the trade-offs accepted, and the questions still open. If it doesn't pass, expand the rationale.

## Not for

GitHub release notes, product changelogs meant for customers, status updates with no decision in them, notes to self, client-facing communication. Those have their own channels.
