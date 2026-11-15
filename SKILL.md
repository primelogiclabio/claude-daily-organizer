---
name: daily-organizer
description: "Keeps the user's personal daily organizer artifact on claude.ai: Today, Up Next, Waiting On and Done lists in the artifact's database. Use to start it, add, update, finish or review tasks (also when called a workday tracker or task list), keep topics with notes and checklists, plan or wrap up the day, or change the organizer page."
---

# Daily organizer

A personal, day-to-day organizer kept as a claude.ai artifact: a page with **Today**, **Up Next**, **Waiting On**, **Upcoming**, **Ideas** and **Done**, and a **Days** log of past plans and summaries, shown as lists or tables. The page takes the window's full width. A pane down the left lists them, with counts, and shows one at a time, or **All** of them stacked; on a phone the pane becomes tabs above the list. **Settings** sits at the bottom of the pane, apart from the lists. Its **People** item lists the open tasks that involve team members (for them, or discussions with them), grouped by the people each involves, and while People is shown the team is listed under it, each member to show only theirs; on every other list the team folds away. Open tasks planned for a later day are under **Upcoming**; those planned for a day gone by show as **Carried over** under the day's tasks, marked with the day they were planned for. Tasks set aside with no day sit at the end of Up Next (or their type's tab), marked someday. Once Claude has copied in the tickets flagged on the user's Jira Dashboard, they show under **Jira integration**, each with **Add as task**; that tab isn't there before. Open ideas live under **Ideas**, not Up Next or Waiting On, unless they have a day or are set aside for someday; any other type can have a tab of its own that works the same way. A **Tasks | Topics** switch in the header swaps the lists for the **Topics** screen, for subjects that aren't tasks, such as "Q4 planning": topics down the left; for the picked topic, its overview and tickets under its name, then a row for its own part (**Main**) and one for each person working on it, each with its checklist in the middle and its notes (bullet points) on the right, which take the rest of the window's width; a note's grip drags it to another place in its list or into another person's notes or Main's. Clicking a row opens its details (every field, then how many comments and checklist items it has, with **Open**; in the table, which has no note column, the status note too), and closes any other row that was open. **Open**, a row's comments link or its checklist count (such as ☑ 1/3) opens the task in a larger view over the page: its comments in the main column and its **Checklist** in a pane beside them (under them on a phone). Each task's **Edit** changes any of its fields on the page, and each comment and checklist item has its own **Edit**. A task's links show on its row (one link's name, or how many it has) and in its details and task view; a topic's show under its tickets, with **Add a link**; a claude.ai artifact is marked ▣, any other link ↗. A task's type shows as a tag; besides the built-in types (task, idea, follow-up, discussion, GitHub and Jira), **Settings → Types** lists every type with **Use** (Task is always on), the priority its new tasks start at and **Own tab**, and adds, edits (name, colour) and removes the user's own. **Settings → Lists** hides All or Jira integration; Today, Up Next, Waiting On, Upcoming, People and Done always show. A GitHub issue or PR link shows as `owner/repo#12`, and a ticket written that way links to it on github.com. The add box starts from the list shown: the day shown on Today, no day on Up Next, Waiting On on Waiting, tomorrow on Upcoming, Someday on Someday, an idea on Ideas, and a discussion with the person viewed on People; its priority starts at the type's (High for GitHub); a field changed by hand keeps the pick until the task is added. Claude's own adds don't follow it: they get no day unless the user names one. The header shows today's figures (open today, carried over, to chase, done today), a bar for how much of today's plan is done and the page's version beside the date, and stays at the top as the page scrolls. A bar above the lists picks the day shown, whose tasks to show (once there's a team) and **List | Table**; the tab, the layout, the person, the screen and the topic picked are remembered per device; the **Add a task** box starts folded each time the page opens. The page is built from `template.html` in this skill's folder. Its tasks aren't in the page's code: they're rows in the artifact's own database, which the page shows live and Claude reads and writes with the `ArtifactData` tool.

It's for one person's own work. Don't apply these rules to a team or project tracker, or to a repo's `PROGRESS.md`; if it's unclear which list the user means, ask.

## Finding the organizer

The organizer is the artifact the user points at: by a link, by name ("my daily organizer", "my workday tracker", "my task list"), or the one already in play in the conversation. If you can't tell which one, list the user's artifacts (`Artifact` `list`) and ask. If they have none and ask for one, start one (below) rather than building a different tracker.

## The data

- `tasks/<id>`, one document per task:
  - `title`
  - `type`: `task`, `idea`, `follow-up`, `discussion`, `github` or `jira`, or the id of one of the user's own types (`types/<id>`). A type the page doesn't know, such as one removed, shows there as a task; the task keeps its `type`.
  - `status`: `next`, `waiting` or `done`
  - `created`: ISO timestamp
  - `ticket`, optional: a ticket key or URL
  - `links`, optional: pages the task points to (see Links), a list of strings, one per link: a name then the link (`Daily Organizer https://claude.ai/artifact/KxoB7…`), or a bare link
  - `note`, optional: the current status note
  - `log`, optional: comments, as a list of `{at, text}` entries (ISO timestamp, free text), oldest first
  - `checklist`, optional: the task's steps, in a topic checklist's format (see `topics` below): `{id, text}` entries in the order added; a ticked item also has `done: true` and `doneAt`, and an item can have a `ticket`
  - `comment`: the completion comment, set when `status` becomes `done`
  - `doneAt`: ISO timestamp, set with `comment`
  - `today`, optional: the `YYYY-MM-DD` date the task is planned for: today, or a later day. The field keeps the name `today` so older rows still work.
  - `owner`, optional: the id of the team member the task is for. No `owner` means it's the user's own.
  - `with`, optional, for discussions: who the discussion is with, as one team member's id, or a list of ids when it's with more than one of them. Older rows hold one id. A discussion is always the user's own, with those people; `with` doesn't make it theirs.
  - `priority`, optional: `high` or `low`. No `priority` means Normal.
  - `someday`, optional: `true` for an open task set aside with no day. It has no `today`, and its `status` is `next`.
  - `via`, optional: `"chat"` on tasks Claude added. Tasks added on the page have none.
  - `flag`, optional: the Jira ticket key of the Jira Dashboard flag the task was made from, such as `OPS-142`.
- `team/<id>`, one document per team member: `name`, `created`
- `types/<id>`, one document per type the user added (see Types): `name`, `created`, and optionally `priority` (`high` or `low`: where its new tasks start) and `color` (`grey`, `blue`, `teal`, `green`, `amber`, `red`, `purple` or `pink`; grey when missing), plus `off: true` when the user turned it off and `tab: true` when it has a tab of its own. The id is the name as a slug, such as `bug` or `code-review`, with `-2` and up when that's taken. A row whose id is a built-in type's, such as `types/github`, holds only what the user changed about it: `off`, `tab` (`true` or `false`) and `priority` (`high`, `low` or `normal`).
- `topics/<id>`, one document per topic (see Topics):
  - `title`
  - `created`: ISO timestamp
  - `with`, optional: the team members the topic is a joint effort or discussion with, as for a discussion's `with`: one id, or a list of ids for two or more
  - `overview`, optional: what the topic is about, in the user's words: plain text, a sentence or a paragraph, with line breaks (`\n`) if the user wrote them
  - `tickets`, optional: the tickets the topic relates to, as a list of strings, one per ticket: a Jira key (`OPS-142`), a link, a label then a link (`Case 00012345 https://acme.lightning.force.com/...`), or plain text such as `SF case 00012345`
  - `links`, optional: pages the topic points to, as for a task's `links`
  - `notes`, optional: the topic's notes, written as bullet points: a list of `{id, text}` entries in the order added, with no dates. A note can be a short point or a paragraph, with line breaks (`\n`) if the user wrote them. Note ids are like item ids, such as `n-` plus the time in milliseconds.
  - `checklist`, optional: items in the order added, as a list of `{id, text}` entries; a ticked item also has `done: true` and `doneAt` (ISO timestamp), and an item can have a `ticket`, in the same forms as the topic's `tickets`. Item ids are letters, digits and dashes, such as `c-` plus the time in milliseconds.
  - `people`, optional: each person's own part of the topic, keyed by team member id: `{"<member id>": {"notes": [...], "checklist": [...]}}`, both lists shaped as above. `notes` and `checklist` on the topic itself are its main part.
- `jira/<KEY>`, one document per ticket flagged on the Jira Dashboard, copied in by Claude (see Jira flags): `key`, `summary`, `status`, `statusCategory`, `assignee`, `url`, `note`, `flaggedAt`, `updated`
- `days/<YYYY-MM-DD>`, one document per day:
  - `plan`: task ids
  - `planNote`, optional
  - `summary`, optional: the end-of-day summary
  - `closedAt`, optional
- `organizer/settings`, one document:
  - `ticketUrl`, optional: where ticket keys link. Either a URL with `{key}` where the key goes, or a prefix the key is added to the end of.
  - `pageVersion`: the version of the page the organizer runs, written by the page itself when it loads. Missing on pages older than 0.4.0.
  - `pageChanges`, optional: changes Claude made to the page at the user's request, as a list of `{at, text}` entries, so an update can carry them over.
  - `jira`, optional: `{url, syncedAt}`, the Jira Dashboard's artifact link and when its flags were last copied in; or `{status: "none", checkedAt}` when a sync found no Jira Dashboard.
  - `hiddenTabs`, optional: the lists the user hid, of `all` and `jira` (Jira integration).

  Organizers from before 0.10.0 keep these fields in `settings/main`, a name another skill's artifact might also use. Read it when `organizer/settings` doesn't exist; updating the page moves them (see Updating the page).

Dates are the user's local date. New task ids are letters, digits and dashes, such as `t-` plus the current time in milliseconds.

## Starting one

1. Copy `template.html` from this skill's folder into the session's scratchpad (or the working directory) as `daily-organizer.html`. Change nothing in it unless the user asked for a change.
2. Publish it with the `Artifact` tool:
   - `icon`: `checklist`
   - `description`: "My tasks for today, what's next and what I'm waiting on."
   - `capabilities`: `{"db": {"rules": [{"path": "", "read": "admin", "write": "admin"}]}}`

   The rule keeps every row to the owner and anyone they make an Editor, even if the page is shared.
3. Check it with one `ArtifactData` `list` of `tasks`. An empty list is right. Never seed sample tasks.
4. Give the user the link. If they listed tasks while asking, add them now (below). The page records its version in `organizer/settings` the first time it's opened.

## Task rules

1. **Type.** Every task is a `task`, `idea`, `follow-up`, `discussion`, `github` or `jira`, or one of the user's own types (see Types). Ask when the user doesn't say. Default to `task` only for a clearly actionable to-do. A stray thought is likely an `idea`, something waiting on another person or system is likely a `follow-up`, and a topic to talk through with someone (a point for a 1:1 or a meeting) is likely a `discussion`: ask rather than guess. When the user says who a discussion is with and they're on the team, set `with` to their id, or to a list of ids when it's with more than one of them, so it shows on each one's 1:1 agenda. If someone isn't on the team, ask whether to add them; if not, name them in the title, such as "Q3 budget with Sam". A discussion is always the user's own: don't give it an `owner`. A GitHub issue or pull request to work on is `github`, with its link as `ticket` (or `owner/repo#12`); a Jira ticket to work on is `jira`, with its key as `ticket`. Both start off: while one is off, use `task` with the same ticket and mention once that the user can turn that type on. A Jira flag to talk through stays a `discussion` (see Jira flags). Use one of the user's own types when they name it ("add a bug: …") or when the task plainly is one; read `types` to know them, and don't use one that's off (see Types).
2. **Completing needs a comment.** A task is done only with a short comment saying what was done or decided. If the user says "mark X done" without one, ask for it first. The page enforces the same rule.
3. **Ticket, optional.** A ticket key such as `OPS-142`, or a full URL, at creation or later. Never required. A full URL always links. A key links once the organizer has a ticket link (below), and shows as plain text until then.
4. **Status note, optional.** A short, changing note such as "blocked on DNS change", separate from the completion comment. Useful for anything that sits more than a day.
5. **Comments, optional.** Dated entries with more detail on progress: what was tried, what was found, what's next. They build up over time, while the status note stays the one-line current state. When the user shares progress on a task, add it as a comment. Change the status note only when they say where the task stands now.
6. **Checklist, optional.** Steps to tick off on the way to finishing a task ("add 'check the TTL' and 'update the doc' to the DNS task's checklist"). Comments say what happened; checklist items are what's left to do. When "add X to the DNS task" could be either, ask. Finishing a task doesn't need its items ticked; if some are still open, say how many when confirming. When the user asks about a task, give its open items and how many are done.

7. **Owner.** A task is the user's own unless they name a team member ("add a follow-up for Sam to check the logs"). Then set `owner` to that member's id. If nobody on the team has that name, ask whether to add them first. Never guess an owner.
8. **Priority, optional.** Set `high` when the user calls a task urgent, important or top priority, and `low` for "low priority", "whenever" or "no rush". Don't ask when they don't say: Normal is the default, except that a new task starts at its type's priority (see Types) unless the user says otherwise, as the page's add box does. To make a task Normal again, write `priority` as `{"__delete__": true}`. The page lists open tasks High first, then Normal, then Low.
9. **Someday, optional.** When the user says a task is for "someday", "eventually" or "not now", set `someday: true` and `status: "next"`, and remove any `today`. Planning a someday task for a day removes `someday` in the same write.
10. **Via chat.** Every task Claude creates gets `via: "chat"`, so the page marks it "via chat". Don't add it to tasks the user made on the page.

A plain "add a task" with nothing else is the user's own `task` in Up Next, Normal priority, with no day, ticket or note, and `via: "chat"`.

## Types

The built-in types are `task`, `idea`, `follow-up`, `discussion`, `github` and `jira`; they can't be renamed or removed. The user's own are rows in `types`. **Settings → Types** on the page lists them all.
- **On or off.** `github` and `jira` start off, and are on once their row has `off: false`; every other type is on unless its row has `off: true`; `task` is always on. Don't use a type that's off: when the user names one, say it's off and offer to turn it on. A type turned off keeps its tasks, which show in Up Next or Waiting On.
- **Priority.** A type's priority is its row's `priority` (`normal` meaning Normal), else its default: High for `github`, Normal for the rest. A new priority applies to tasks added from then on.
- **Own tab.** A type with a tab holds its open tasks that have no day and aren't set aside for someday, as **Ideas** does for ideas; say that tab's name when telling the user where such a task is. Idea's tab is on unless its row has `tab: false`; other types have one only with `tab: true`; `task` never has one, Up Next being its list.
- **Changing them.** To turn a type off or on, give it a tab or take it away, or change its priority: for the user's own, `update` `off`, `tab` or `priority`, writing `{"__delete__": true}` to clear one. For a built-in, `set` `types/<id>` if there's no row yet, else `update` it, keeping only what differs from its defaults (GitHub and Jira off, Idea's tab on, GitHub at High): a value set back to its default is deleted.
- To add one ("add a Bug type, high priority, red"), check that no type has that name, built-in or the user's own, ignoring case and punctuation; then `set` `types/<slug>` with `name`, `created`, and `priority`, `color` and `tab: true` if the user gave them. Use the next free `-2`, `-3`… slug when one is taken.
- To rename one or change its colour, `update` `name` or `color`. The id stays, so its tasks keep their type.
- To remove one, say how many tasks use it and offer to change them to another type first; then `delete` its row. Tasks left with it keep their `type` and show as tasks until a type with that name is added back.

**Lists.** Today, Up Next, Waiting On, Upcoming, People and Done are always shown. The user can hide **All** and **Jira integration** (`hiddenTabs` in `organizer/settings`): to hide or show one, `update` `hiddenTabs` with the list as it should be. There's no Someday list: tasks set aside for someday sit at the end of Up Next, or of their type's tab, marked someday.

## Links

Tasks and topics can have `links`: pages they point to, such as a claude.ai artifact, a doc or a design. Tickets stay in `ticket` (tasks) or `tickets` (topics).
- To link an artifact ("link my Jira Dashboard to the Kafka topic"), list the user's artifacts to find it and add `<its title> https://claude.ai/artifact/<id>`. If more than one could be meant, ask.
- For any other link, take the name from the user's words, or add the bare link.
- `update` the whole list with `if_version`, keeping the links already there; to remove one, write the list without it, or `{"__delete__": true}` for none.
- When the user asks about a task or topic, mention its links by name.

## Team

The page's **Settings**, the last item in its left pane, lists the team (the Topics screen has a **Settings** with the same team, so people can be added from either screen), and the add box and each task's **Edit** let the user pick who a task is for (**For**) and who a discussion is with (**With**, any number of people). Discussions have no **For**: they're always the user's own. The page can also show one person at a time: their tasks, with a **1:1 with** section above them listing the open discussions with that person.
- To add someone, `set` `team/<new id>` with `name` and `created`. Ids are letters, digits and dashes, such as `m-` plus the time in milliseconds.
- To assign a task, `update` its `owner`. To give it back to the user, write `owner` as `{"__delete__": true}`.
- To add someone to a discussion, `update` `with` to the list of everyone it's with (one id becomes a list). To take someone off, write who's left: a list for two or more, one id for one, or `{"__delete__": true}` for no one.
- To remove someone, first give their tasks back to the user and take them off their discussions' and topics' `with` (one `batch`), then `delete` their `team` document. Leave their parts of topics (`people`) in place: the page still shows those, as a former member's. Say how many tasks came back.
- When the user asks what someone has, `query` `tasks` where `owner` is that member's id.
- When the user asks for their 1:1 with someone, or what to raise with them, list the open tasks whose `with` is or includes that member's id, then the open tasks they own. Older discussions may name the person only in the title: mention those too, and offer to set their `with`.

## Topics

A topic is a subject the user keeps notes and a checklist on, apart from the task lists: a project to think through, a meeting series, a list of things to check. It isn't a task: it has no type, status, day or owner, and plan-my-day and wrap-up leave topics out unless the user asks about one. The page's **Topics** screen lists them down the left, newest first, under a **New topic** box; the topic picked shows its overview and tickets under its name, then its checklist in the middle and its notes as bullet points on the right, and its **Edit** changes its title, overview and tickets, or removes it. A click on a note shows its **Edit**, **Remove**, **↑**, **↓** and **Move to** under it, and its grip drags it within its list or into another part's notes. Under the topic's name, **All** shows every part, and **Main** and a button per person narrow it: a click shows only that part, further clicks add others. On a phone the screen shows the list, or one topic with a way back. **Hide checklist**, beside a topic's **Edit**, hides its checklists on that device so the notes take the width, and **Show checklist · 2/6** brings them back; it's a view choice kept in the browser, so Claude adds and ticks items as usual.

- **People.** When the user says who a topic involves ("the Q4 topic is with Sam and Priya"), set `with` from the team as for a discussion, and change it the same way. Someone not on the team: ask whether to add them first. The page shows a **With** dropdown under the topic's name.
- **Creating.** "Start a topic for Q4 planning": `set` `topics/<new id>` with `title` and `created`, and `overview` and `tickets` when the user says what it's about or names tickets. Ids are like task ids, such as `p-` plus the time in milliseconds.
- **Overview.** When the user says what a topic is about ("the Q4 topic is about the budget and headcount"), `update` `overview` with it in their words, replacing the one there; add to it rather than replace it only when they say so. To remove it, delete the field.
- **Tickets.** When the user links a ticket to a topic ("link OPS-142 to Q4 planning", "add this Salesforce case to it: …"), `update` `tickets` with the topic's tickets plus the new one at the end, in one write, as with notes; skip one that's already there. A Jira key goes in as the key. A link goes in as given, after a short label when the user names it ("Case 00012345 https://…"), since a bare Salesforce link only shows as "Salesforce Case". A Salesforce case given only by its number goes in as `SF case <number>`. Remove a ticket by writing the list without it. The page links Jira keys through the Ticket link setting, never other text.
- **Finding one.** Match the topic by its title, and ask if more than one matches or none does. When the user says "add a note to Q4 planning" and there's both a topic and a task by that name, ask which.
- **Notes.** Notes are bullet points without dates, each a short point or a paragraph. To add points, `update` `notes` with the topic's existing points plus each new `{id, text}` at the end, in one write; when the user gives a paragraph, keep it as one note, and split it into points only if they wrote it as points. Change or remove a point the same way as a checklist item (below). An `update` replaces the whole list, as with comments, so never leave out the other points. To reorder notes, write the list in the new order. To move a note to another person's part or to Main ("move the vendor note to Priya"), take it out of one list and add it to the other in one `update` (`notes` and/or `people`), keeping its `id`, and drop a person's part that's left empty.
- **Checklist.** To add items, `update` `checklist` with the existing items plus each new `{id, text}` at the end, in one write. To tick an item, write the list with `done: true` and `doneAt` (now) on it; to untick, write it without those two. Find items by `id`; an item without an `id` is found by its place in the list. Rename or remove an item the same way, keeping the rest. An item can carry a `ticket` when the user names one ("add 'send the numbers' to Q4 with OPS-142"), written as for the topic's tickets; remove it by writing the item without `ticket`.
- **Each person's part.** A topic has its main notes and checklist, and each person working on it can have their own, for the part they're on or follow-up questions for them. When the user names the person ("add to Sam's notes on Q4 planning", "put 'send the numbers' on Sam's checklist for Q4"), write to that person's part: `update` `people` with the whole map as it is, changing only that person's `notes` or `checklist` in the same way as the main lists, so other people's parts stay. Someone not on the team: ask whether to add them first. Without a person, notes and items go to the main part. A person's part shows on the page while they're in the topic's `with` or have notes or items in it; when the user names someone for a part who isn't in `with` yet, add them to `with` in the same write.
- **Reviewing.** When the user asks what's on a topic, start with its overview and tickets, then give its main notes as bullet points and its open checklist items with how many are done, then the same for each person with a part, under their name. When they ask about one person's part, give just that.
- **Removing.** Only when the user asks: `delete` the topic's document. It takes all its notes and checklists with it, people's parts included.

Every write to an existing topic passes its `if_version`, as with tasks.

## Changing tasks

Write rows with `ArtifactData` on the organizer's `url`:
- `set` creates a task.
- `update` changes some fields.
- `delete` removes a task.
- `batch` does several at once. Use it whenever you change more than a couple.

Read with `list` or `query` before acting on a task, so you work from its current state. Match a task by its title, and ask if more than one matches. Every write to an existing row passes the `version` you read as `if_version`. If the row changed in between (say the user ticked it on the page), the write is refused: read it again and redo the write.

Never republish the page to change tasks. Its code changes only when the user asks for a change to the page itself (below).

To finish a task, `update` `status: "done"`, `comment` and `doneAt` together. To reopen one, `update` `status: "next"` and remove the other two by writing each as `{"__delete__": true}`. Remove a status note or a `today` date the same way.

To push a task to tomorrow ("push X to tomorrow"), `update` `today` to tomorrow's date, removing `someday` if it's set. On the page, the user does the same with the day in the task's **Edit**.

To add a comment, `update` `log` with the task's existing entries plus the new one at the end: an `update` replaces the whole list, so leaving the old entries out deletes them. To change a comment, write the whole list the same way, with only that entry's `text` changed and its `at` kept.

A task's `checklist` is written as a topic's is (see Topics, Checklist): new `{id, text}` items at the end of the whole list in one `update`, a ticked item with `done: true` and `doneAt` (now), an unticked one without those two, and items found by `id`, or by their place in the list when they have none.

The page's **Edit** is the one button on a task row. It changes the title, type, list, priority, day (any date, or Someday), ticket, links, status note, `owner` and `with`, and the date in `created`; on a done task, also `comment` and the date in `doneAt`. It writes only the fields the user changed, and holds **Remove task**. When the user asks Claude for the same changes ("rename X", "make it a follow-up", "move it to Friday"), `update` those fields with the rules above: a day removes `someday`, and Someday removes `today` and sets `status: "next"`.

## Ticket link

The page's **Settings** has a **Ticket link** field, saved as `ticketUrl` in `organizer/settings`.
- When the user says where their tickets live (for example "our Jira is acme.atlassian.net"), set it with `ArtifactData`: `https://acme.atlassian.net/browse/` adds the key at the end, and `https://linear.app/acme/issue/{key}` puts it where `{key}` is. Use `set` if the document doesn't exist yet, else `update` with its `if_version`.
- Then show one example link built from a ticket they already have, so they can check it opens the right page.
- If the user adds a ticket key while no ticket link is set, mention once that the page can link keys and ask where their tickets live. Don't ask again in the same conversation.

## Jira flags

The user's **Jira Dashboard** artifact (the jira-dashboard skill's) lets them flag tickets for discussion. The organizer page can't read another artifact's database, so Claude copies the flags in, and the page lists them under **Jira integration** as of that copy. That tab shows once a sync has connected a Jira Dashboard, unless the user hid it; until then **Settings → Lists** explains how to set one up, and its **Sync now** only copies "sync my Jira flags" for the user to paste to Claude. The `jira` task type is separate: it's for Jira tickets the user works on.

- **Finding the dashboard.** Use `organizer/settings.jira.url`. Otherwise list the user's artifacts and take the one titled **Jira Dashboard**; ask if there's more than one.
  - **None found:** `update` (or `set`) `organizer/settings` with `jira: {status: "none", checkedAt}` (now), so the page says Jira flags aren't available. Tell the user the setup steps: add the Atlassian connector on claude.ai (Settings › Connectors), install the Jira Dashboard skill (https://github.com/primelogiclabio/claude-jira-dashboard#try-it), ask Claude to start a Jira dashboard and flag tickets there, then sync again.
  - **Found:** save its link with the first sync's write (step 3).
- **Syncing**, when the user asks ("sync my Jira flags", "pull in what I flagged") and at the start of Plan my day once `organizer/settings.jira.url` is set:
  1. Read the dashboard's `flags` and `tabs`, and the organizer's `jira` and `tasks`.
  2. For each flag, take the ticket's freshest row: the dashboard tab row with the latest `updated` when the ticket is on a tab, else the flag's own `row`. Build `{key, summary, status, statusCategory, assignee, url, note, flaggedAt, updated}`.
  3. Write in `batch`es of at most 50: `set` `jira/<KEY>` for each flag that's new or changed, `delete` the copies whose flag is gone, and `update` `organizer/settings` with `jira: {url, syncedAt}` (now). An `update` merges nested objects, so also write `status` and `checkedAt` inside `jira` as `{"__delete__": true}` if an earlier sync left them.
  4. Never write to the Jira Dashboard: flags are set and cleared there.
- **Offering tasks.** List the flags no task has as its `flag`, each with key, summary, status and note, and offer to add them. Each becomes a `discussion` in Up Next: `title` the summary, `ticket` the url, `note` the flag note if there is one, `flag` the key and `via: "chat"`. Set `with` only when the user names who it's with. Add the ones the user picks in one `batch`.
- **Unflagged tickets.** When a sync finds a task whose `flag` is no longer flagged, say so once and ask whether to finish it (with a comment, as always) or keep it.

The page's **Add as task** on a flag makes the same discussion, without `via`.

## Plan my day

When the user asks to plan the day, or what to work on today:
1. Read `tasks`, `team`, `organizer/settings`, and the most recent `days` document before today. If `organizer/settings.jira.url` is set, sync the Jira flags first (see Jira flags). If the page is older than this skill's template, offer the update (see Updating the page) after the plan.
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
1. Read `tasks`, `organizer/settings` and `days/<today>`. If the page is older than this skill's template, offer the update (see Updating the page) after the summary.
2. Draft a short summary of:
   - what got done today, with each completion comment;
   - progress on open tasks, from comments added today and checklist items ticked today;
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

Making or writing the organizer needs claude.ai's artifact tools: publishing with `capabilities` (`Artifact`) and reading and writing rows (`ArtifactData`). They may go by other names, such as artifact database tools for listing, getting and writing rows; use whichever this session has. Claude Code sessions have them, and so do claude.ai chats. An artifact made inside a chat, with no artifact link (a chat artifact), has no database. When this session has no way to write the organizer's rows:
- **For an organizer that already exists:** say you can't change its rows from here. The user can add, complete and plan tasks on the page itself, wherever they open it, or ask from a claude.ai chat or a Claude Code session, which have the tools.
- **For a chat-artifact tracker:** keep working the older way. Tasks stay in the artifact's code, the same task rules apply, and missing fields (type, ticket, status note, comments, completion comment) get added to the artifact's code rather than tracked in chat. Keep its plain look.

## Updating the page

An organizer keeps the page it was published with; a newer `template.html` doesn't reach it on its own. The template's version is in its `<meta name="daily-organizer-version">` tag. The organizer's is `organizer/settings.pageVersion`, or `settings/main.pageVersion` on an organizer whose settings haven't moved yet; if both are missing, the page is older than 0.4.0 or hasn't been opened since it was last published.

- **When to offer.** When planning or wrapping up the day finds the organizer older than the template, say so once per conversation: its version, the template's, and in a line what the newer page adds. Offer to update it. Also update when the user asks ("update my organizer"). Never update without a yes.
- **Updating, on a yes:**
  1. Read the page with `Artifact` `read`.
  2. Copy `template.html` as in Starting one. Re-apply each entry in `organizer/settings.pageChanges` to the copy. If the organizer has no `pageVersion`, there's no record of earlier changes: ask once whether Claude ever changed the page's look or features for them, and re-apply what they name. Leave the version meta as the template has it.
  3. If `settings/main` exists and `organizer/settings` doesn't, move the settings in one `batch`: `set` `organizer/settings` with the organizer's fields from `settings/main` (`ticketUrl`, `pageVersion`, `pageChanges`, `jira`, whichever are there), and `delete` `settings/main` with its `if_version`. Carry no other fields across.
  4. Publish the copy to the same `url`. Omit `capabilities` so the database rules carry over.
  5. `list` `tasks` and check the count is the same as before. Tell the user to reload the page. It records its new version when it loads.

## Changing the page itself

Only when the user asks for a feature or a look the page doesn't have:
1. Read the current page with `Artifact` `read`, edit the saved file, and publish it to the same `url`.
2. Omit `capabilities` so the database rules carry over.
3. Keep the data fields above, so existing rows still show. If a change needs a new field, add it to the rows that need it rather than renaming old ones.
4. Add a one-line entry for the change to `organizer/settings.pageChanges` (`{at, text}`), so a later update can carry it over. Don't change the page's version meta.
