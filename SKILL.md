---
name: daily-organizer
description: "Keeps the user's personal daily organizer artifact on claude.ai: Today, Up Next, Waiting On and Done lists in the artifact's database. Use to start it, add, update, finish or review tasks (also when called a workday tracker or task list), plan or wrap up the day, or change the organizer page."
---

# Daily organizer

A personal, day-to-day organizer kept as a claude.ai artifact: a page with **Today**, **Up Next**, **Waiting On**, **Upcoming**, **Someday**, **Ideas** and **Done**, and a **Days** log of past plans and summaries, shown as lists or tables. Tabs show one list at a time, with counts, and the **All** tab shows them all stacked. Open tasks planned for a later day are under **Upcoming**; those planned for a day gone by show as **Carried over** under the day's tasks, marked with the day they were planned for. Tasks set aside with no day are under **Someday**. Tickets flagged on the user's Jira Dashboard show under **Jira** once Claude has copied them in, each with **Add as task**. Open ideas live under **Ideas**, not Up Next or Waiting On, unless they have a day or are set aside for someday. The header shows today's figures (open today, carried over, to chase, done today) and a bar for how much of today's plan is done. A bar above the lists picks the day shown, whose tasks to show (once there's a team) and **List | Table**; the tab, the layout, the person and whether the **Add a task** box is folded are remembered per device. The page is built from `template.html` in this skill's folder. Its tasks aren't in the page's code: they're rows in the artifact's own database, which the page shows live and Claude reads and writes with the `ArtifactData` tool.

It's for one person's own work. Don't apply these rules to a team or project tracker, or to a repo's `PROGRESS.md`; if it's unclear which list the user means, ask.

## Finding the organizer

The organizer is the artifact the user points at: by a link, by name ("my daily organizer", "my workday tracker", "my task list"), or the one already in play in the conversation. If you can't tell which one, list the user's artifacts (`Artifact` `list`) and ask. If they have none and ask for one, start one (below) rather than building a different tracker.

## The data

- `tasks/<id>`, one document per task:
  - `title`
  - `type`: `task`, `idea`, `follow-up` or `discussion`
  - `status`: `next`, `waiting` or `done`
  - `created`: ISO timestamp
  - `ticket`, optional: a ticket key or URL
  - `note`, optional: the current status note
  - `log`, optional: comments, as a list of `{at, text}` entries (ISO timestamp, free text), oldest first
  - `comment`: the completion comment, set when `status` becomes `done`
  - `doneAt`: ISO timestamp, set with `comment`
  - `today`, optional: the `YYYY-MM-DD` date the task is planned for: today, or a later day. The field keeps the name `today` so older rows still work.
  - `owner`, optional: the id of the team member the task is for. No `owner` means it's the user's own.
  - `with`, optional, for discussions: the id of the team member the discussion is with. It doesn't change whose task it is.
  - `priority`, optional: `high` or `low`. No `priority` means Normal.
  - `someday`, optional: `true` for an open task set aside with no day. It has no `today`, and its `status` is `next`.
  - `via`, optional: `"chat"` on tasks Claude added. Tasks added on the page have none.
  - `flag`, optional: the Jira ticket key of the Jira Dashboard flag the task was made from, such as `OPS-142`.
- `team/<id>`, one document per team member: `name`, `created`
- `jira/<KEY>`, one document per ticket flagged on the Jira Dashboard, copied in by Claude (see Jira flags): `key`, `summary`, `status`, `statusCategory`, `assignee`, `url`, `note`, `flaggedAt`, `updated`
- `days/<YYYY-MM-DD>`, one document per day:
  - `plan`: task ids
  - `planNote`, optional
  - `summary`, optional: the end-of-day summary
  - `closedAt`, optional
- `settings/main`, one document:
  - `ticketUrl`, optional: where ticket keys link. Either a URL with `{key}` where the key goes, or a prefix the key is added to the end of.
  - `pageVersion`: the version of the page the organizer runs, written by the page itself when it loads. Missing on pages older than 0.4.0.
  - `pageChanges`, optional: changes Claude made to the page at the user's request, as a list of `{at, text}` entries, so an update can carry them over.
  - `jira`, optional: `{url, syncedAt}`, the Jira Dashboard's artifact link and when its flags were last copied in.

Dates are the user's local date. New task ids are letters, digits and dashes, such as `t-` plus the current time in milliseconds.

## Starting one

1. Copy `template.html` from this skill's folder into the session's scratchpad (or the working directory) as `daily-organizer.html`. Change nothing in it unless the user asked for a change.
2. Publish it with the `Artifact` tool:
   - `icon`: `checklist`
   - `description`: "My tasks for today, what's next and what I'm waiting on."
   - `capabilities`: `{"db": {"rules": [{"path": "", "read": "admin", "write": "admin"}]}}`

   The rule keeps every row to the owner and anyone they make an Editor, even if the page is shared.
3. Check it with one `ArtifactData` `list` of `tasks`. An empty list is right. Never seed sample tasks.
4. Give the user the link. If they listed tasks while asking, add them now (below). The page records its version in `settings/main` the first time it's opened.

## Task rules

1. **Type.** Every task is a `task`, `idea`, `follow-up` or `discussion`. Ask when the user doesn't say. Default to `task` only for a clearly actionable to-do. A stray thought is likely an `idea`, something waiting on another person or system is likely a `follow-up`, and a topic to talk through with someone (a point for a 1:1 or a meeting) is likely a `discussion`: ask rather than guess. When the user says who a discussion is with and they're on the team, set `with` to their id, so it shows on that person's 1:1 agenda. If they aren't on the team, ask whether to add them; if not, name them in the title, such as "Q3 budget with Sam". `owner` still means whose task it is.
2. **Completing needs a comment.** A task is done only with a short comment saying what was done or decided. If the user says "mark X done" without one, ask for it first. The page enforces the same rule.
3. **Ticket, optional.** A ticket key such as `OPS-142`, or a full URL, at creation or later. Never required. A full URL always links. A key links once the organizer has a ticket link (below), and shows as plain text until then.
4. **Status note, optional.** A short, changing note such as "blocked on DNS change", separate from the completion comment. Useful for anything that sits more than a day.
5. **Comments, optional.** Dated entries with more detail on progress: what was tried, what was found, what's next. They build up over time, while the status note stays the one-line current state. When the user shares progress on a task, add it as a comment. Change the status note only when they say where the task stands now.

6. **Owner.** A task is the user's own unless they name a team member ("add a follow-up for Sam to check the logs"). Then set `owner` to that member's id. If nobody on the team has that name, ask whether to add them first. Never guess an owner.
7. **Priority, optional.** Set `high` when the user calls a task urgent, important or top priority, and `low` for "low priority", "whenever" or "no rush". Don't ask when they don't say: Normal is the default. To make a task Normal again, write `priority` as `{"__delete__": true}`. The page lists open tasks High first, then Normal, then Low.
8. **Someday, optional.** When the user says a task is for "someday", "eventually" or "not now", set `someday: true` and `status: "next"`, and remove any `today`. Planning a someday task for a day removes `someday` in the same write.
9. **Via chat.** Every task Claude creates gets `via: "chat"`, so the page marks it "via chat". Don't add it to tasks the user made on the page.

A plain "add a task" with nothing else is the user's own `task` in Up Next, Normal priority, with no day, ticket or note, and `via: "chat"`.

## Team

The page's **Settings** section lists the team, and the add box and each task's **Assign** let the user pick who a task is for; a discussion's **With…** picks who it's with. The page can also show one person at a time: their tasks, with a **1:1 with** section above them listing the open discussions with that person.
- To add someone, `set` `team/<new id>` with `name` and `created`. Ids are letters, digits and dashes, such as `m-` plus the time in milliseconds.
- To assign a task, `update` its `owner`. To give it back to the user, write `owner` as `{"__delete__": true}`. Set or clear a discussion's `with` the same way.
- To remove someone, first give their tasks back to the user and clear `with` on their discussions (one `batch` of `owner` and `with` deletes), then `delete` their `team` document. Say how many tasks came back.
- When the user asks what someone has, `query` `tasks` where `owner` is that member's id.
- When the user asks for their 1:1 with someone, or what to raise with them, list the open tasks whose `with` is that member's id, then the open tasks they own. Older discussions may name the person only in the title: mention those too, and offer to set their `with`.

## Changing tasks

Write rows with `ArtifactData` on the organizer's `url`:
- `set` creates a task.
- `update` changes some fields.
- `delete` removes a task.
- `batch` does several at once. Use it whenever you change more than a couple.

Read with `list` or `query` before acting on a task, so you work from its current state. Match a task by its title, and ask if more than one matches. Every write to an existing row passes the `version` you read as `if_version`. If the row changed in between (say the user ticked it on the page), the write is refused: read it again and redo the write.

Never republish the page to change tasks. Its code changes only when the user asks for a change to the page itself (below).

To finish a task, `update` `status: "done"`, `comment` and `doneAt` together. To reopen one, `update` `status: "next"` and remove the other two by writing each as `{"__delete__": true}`. Remove a status note or a `today` date the same way.

To push a task to tomorrow ("push X to tomorrow"), `update` `today` to tomorrow's date, removing `someday` if it's set. The page's **Push to tomorrow** does the same for carried-over tasks and today's open ones.

To add a comment, `update` `log` with the task's existing entries plus the new one at the end: an `update` replaces the whole list, so leaving the old entries out deletes them.

## Ticket link

The page's **Settings** section has a **Ticket link** field, saved as `ticketUrl` in `settings/main`.
- When the user says where their tickets live (for example "our Jira is acme.atlassian.net"), set it with `ArtifactData`: `https://acme.atlassian.net/browse/` adds the key at the end, and `https://linear.app/acme/issue/{key}` puts it where `{key}` is. Use `set` if the document doesn't exist yet, else `update` with its `if_version`.
- Then show one example link built from a ticket they already have, so they can check it opens the right page.
- If the user adds a ticket key while no ticket link is set, mention once that the page can link keys and ask where their tickets live. Don't ask again in the same conversation.

## Jira flags

The user's **Jira Dashboard** artifact (the jira-dashboard skill's) lets them flag tickets for discussion. The organizer page can't read another artifact's database, so Claude copies the flags in, and the page lists them under **Jira** as of that copy.

- **Finding the dashboard.** Use `settings/main.jira.url`. Otherwise list the user's artifacts and take the one titled **Jira Dashboard**; ask if there's more than one, and if there's none, say there are no flags to bring in. Save its link in `settings/main.jira` on the first sync.
- **Syncing**, when the user asks ("sync my Jira flags", "pull in what I flagged") and at the start of Plan my day once `settings/main.jira` is set:
  1. Read the dashboard's `flags` and `tabs`, and the organizer's `jira` and `tasks`.
  2. For each flag, take the ticket's freshest row: the dashboard tab row with the latest `updated` when the ticket is on a tab, else the flag's own `row`. Build `{key, summary, status, statusCategory, assignee, url, note, flaggedAt, updated}`.
  3. Write in `batch`es of at most 50: `set` `jira/<KEY>` for each flag that's new or changed, `delete` the copies whose flag is gone, and `update` `settings/main` with `jira: {url, syncedAt}` (now).
  4. Never write to the Jira Dashboard: flags are set and cleared there.
- **Offering tasks.** List the flags no task has as its `flag`, each with key, summary, status and note, and offer to add them. Each becomes a `discussion` in Up Next: `title` the summary, `ticket` the url, `note` the flag note if there is one, `flag` the key and `via: "chat"`. Set `with` only when the user names who it's with. Add the ones the user picks in one `batch`.
- **Unflagged tickets.** When a sync finds a task whose `flag` is no longer flagged, say so once and ask whether to finish it (with a comment, as always) or keep it.

The page's **Add as task** on a flag makes the same discussion, without `via`.

## Plan my day

When the user asks to plan the day, or what to work on today:
1. Read `tasks`, `team`, `settings/main`, and the most recent `days` document before today. If `settings/main.jira` is set, sync the Jira flags first (see Jira flags). If the page is older than this skill's template, offer the update (see Updating the page) after the plan.
2. Plan from the user's own tasks. A task assigned to someone else only comes in as a check-in ("ask Sam about the logs") when it's been open 3 or more days. Pick out:
   - open `high` priority tasks;
   - tasks planned for an earlier day and not done (carry-overs, which the page shows under **Carried over**);
   - follow-ups and Waiting On tasks added 3 or more days ago, which may need chasing (the page's **To chase** figure);
   - anything whose ticket, status note or latest comment suggests urgency;
   - Jira flags with no task yet, offered as discussions (see Jira flags).

   Leave `someday` tasks out unless the user asks for them.
3. Propose 3 to 5 tasks for today, with one line each on why. Put high-priority tasks first and say so in their line. Bring in `low` tasks only when there's room. Ask the user to confirm or change the list. Don't write anything yet.
4. On a yes, one `batch`:
   - `update` `today` to today's date on each chosen task, removing `someday` where it's set;
   - `set` (or `update`, if it exists) `days/<today>` with `plan` and any `planNote` the user gave.

   Leave older `today` dates alone: they're history, and the page only shows today's.
5. Tell the user it's on the page under the **Today** tab.

The same works for a later day ("plan tomorrow", "plan Monday"). Use that day's date for `today` and for `days/<date>`. The carry-overs are tasks planned for days before it and not done. The page's day picker and **Today | Tomorrow** tabs show any of the next seven days, and its star plans a task for the day shown.

## Wrap up my day

When the user asks to wrap up, close or summarize the day:
1. Read `tasks`, `settings/main` and `days/<today>`. If the page is older than this skill's template, offer the update (see Updating the page) after the summary.
2. Draft a short summary of:
   - what got done today, with each completion comment;
   - progress on open tasks, from comments added today;
   - which of today's planned tasks are still open, so they carry over;
   - high-priority tasks still open, planned for today or not;
   - what's waiting on others, including tasks assigned to team members, by person;
   - status notes that look stale.
3. Show it, and ask whether anything else got done. Anything the user names as done still needs a comment.
4. On the user's OK, write any completions, then `set` or `update` `days/<today>` with `summary` and `closedAt` (keep `plan` if it's there).
5. Offer to plan tomorrow, starting from what carries over. If the user would rather move today's open planned tasks on as they are, push them to tomorrow in one `batch`.

## Older trackers

A tracker made before this skill used a database keeps its tasks inside the page's code. When the user points at one, offer to move it to the organizer. Don't move it without a yes.
- **A published artifact they can edit** (a claude.ai/artifact link):
  1. Read it with `Artifact` `read`.
  2. Publish `template.html` to the same `url` with the capabilities above, so the link stays.
  3. Write its tasks as rows in one `batch`. Map Up Next to `next`, Waiting On to `waiting` and Done to `done`, and keep any type, ticket, status note, comments and completion comment. Old done tasks stay done even without a comment; the comment rule applies from now on.
  4. `list` the rows back and say how many moved.
- **A chat artifact** (made inside a claude.ai chat, with no artifact link): start a new organizer and copy the tasks into it the same way. Tell the user the new link replaces the old artifact.

## Without the data tool

Making or writing the organizer needs the `Artifact` tool with `capabilities` and the `ArtifactData` tool. Claude Code sessions have both. Chat artifacts on claude.ai have no database. When this session lacks them:
- **For an organizer that already exists:** say you can't change its rows from here. The user can add, complete and plan tasks on the page itself, wherever they open it, or ask from a Claude Code session.
- **For a chat-artifact tracker:** keep working the older way. Tasks stay in the artifact's code, the same task rules apply, and missing fields (type, ticket, status note, comments, completion comment) get added to the artifact's code rather than tracked in chat. Keep its plain look.

## Updating the page

An organizer keeps the page it was published with; a newer `template.html` doesn't reach it on its own. The template's version is in its `<meta name="daily-organizer-version">` tag. The organizer's is `settings/main.pageVersion`; if it's missing, the page is older than 0.4.0 or hasn't been opened since it was last published.

- **When to offer.** When planning or wrapping up the day finds the organizer older than the template, say so once per conversation: its version, the template's, and in a line what the newer page adds. Offer to update it. Also update when the user asks ("update my organizer"). Never update without a yes.
- **Updating, on a yes:**
  1. Read the page with `Artifact` `read`.
  2. Copy `template.html` as in Starting one. Re-apply each entry in `settings/main.pageChanges` to the copy. If the organizer has no `pageVersion`, there's no record of earlier changes: ask once whether Claude ever changed the page's look or features for them, and re-apply what they name. Leave the version meta as the template has it.
  3. Publish the copy to the same `url`. Omit `capabilities` so the database rules carry over.
  4. `list` `tasks` and check the count is the same as before. Tell the user to reload the page. It records its new version when it loads.

## Changing the page itself

Only when the user asks for a feature or a look the page doesn't have:
1. Read the current page with `Artifact` `read`, edit the saved file, and publish it to the same `url`.
2. Omit `capabilities` so the database rules carry over.
3. Keep the data fields above, so existing rows still show. If a change needs a new field, add it to the rows that need it rather than renaming old ones.
4. Add a one-line entry for the change to `settings/main.pageChanges` (`{at, text}`), so a later update can carry it over. Don't change the page's version meta.
