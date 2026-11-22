# daily-organizer

A Claude skill that keeps your personal daily organizer on claude.ai: Today,
Up Next, Waiting On and Done lists that you and Claude both update, with a
morning plan and an end-of-day summary.

Ask Claude to start one and it publishes the page from
[`template.html`](template.html) and gives you the link. A **Days** log keeps
past plans and end-of-day summaries. It installs from the
[claude-tools](https://github.com/primelogiclabio/claude-tools) marketplace.

The page uses the window's full width. A pane down the left lists the lists,
each with a count: **Today**, **Up Next**, **Waiting**, **Upcoming**,
**Ideas**, **People**, **Jira integration** and **Done**, and shows one at a
time. **All** shows every list stacked on one page. The pane stays in
view as you scroll; on a phone it becomes tabs right above the list.
**Settings** sits at the bottom of the pane. Its **Appearance** picks the theme
(System, Light or Dark) and an accent colour; **Same on all my devices**, on by
default, keeps your other devices in step, or untick it to give one device its
own look. Ideas get their own list, so Up
Next holds only work to do, and any other type can have a tab of its own too.
All and Jira integration can be hidden under **Settings › Lists**.
**Upcoming** holds tasks planned for later days; tasks you've set aside with no
day sit at the end of Up Next, marked someday. Unfinished tasks from earlier days show
as **Carried over** under Today, marked "from Oct 9 · 23 days"; give one a new
day with **Edit**.

The header shows today in four figures (open today, carried over, to chase,
done today) and a bar for how much of today's plan is done. It stays at the
top as you scroll; on a phone it shrinks to a short strip. "To chase" counts
follow-ups and waiting tasks that are 3 or more days old.

A bar above the lists holds the **Day** and **People** filters and a
**List | Table** switch, which shows each section as a list or as a table
with columns for the planned day, type, priority, ticket, age and comments;
the columns fit their contents and the task takes the rest. Rows light up under
the pointer; click one to open its details: every field, the status note in
the table, and how many comments and checklist items it has, with **Open**.
Opening a row closes the one already open. **Open**, a row's comments link or
its checklist count (such as ☑ 1/3) opens the task in a larger view over the
page: its comments in the main column and its checklist in a pane beside them,
or under them on a phone. Escape, × or a click outside closes it. Each device
remembers its own tab and layout, so a phone can stay on the Today list while
a desktop shows All as tables. The **Add a task** box above the list starts
folded each time the page opens, leaving the room to the list; click it to
open the form. It starts from the list you're on: today on Today, no day on Up Next, Waiting On on
Waiting, tomorrow on Upcoming, Someday on Someday, an idea on Ideas, and a
discussion with the person you're viewing on People; anything you change
yourself stays until you add the task. Its **Day** picker can plan a new task
for one of the next six days, **Someday** or no day; **+ Note** adds a status note as you add it. Tasks Claude added from a chat
say "via chat". The page's version shows beside the date and at the bottom.

It used to be called **workday-tracker**. See [Renamed](#renamed-from-workday-tracker)
if you had that installed.

## How it keeps your tasks

The tasks aren't written into the page. They're records stored with the
artifact on claude.ai, in the artifact's own database:

- The page shows them live and saves what you do there straight away: add a
  task, star it for today, tick it off, add a comment.
- **Edit** is the one button on each task. It changes any of its fields:
  title, type, list, priority, day (any date, or Someday), ticket, status
  note, who it's for or with, and the day it was added; on a done task, also
  the completion comment and the day it was done. **Remove task** is there
  too. Each comment and checklist item has its own **Edit**.
- Claude reads and changes the same records when you ask, without rewriting
  the page, so asking for "add a follow-up to chase the DNS change" is quick
  and can't drop other tasks.
- The records stay when the page itself is updated.
- Only you, and anyone you make an Editor, can see or change them, even if
  you share the page's link.

## The rules it keeps

- Every task has a **type**: `task`, `idea`, `follow-up`, `discussion` (a
  topic to talk through with someone, such as a point for a 1:1), `github` (a
  GitHub issue or PR to work on, High priority unless you pick another) or
  `jira` (a Jira ticket to work on). GitHub and Jira start off: turn them on
  under **Settings › Types**. Claude asks when it isn't clear. A
  discussion can say who it's with (see [Team](#team)).
- **Settings › Types** lists every type. Untick **Use** on the ones you don't
  need (Task always stays): they leave the add box, and their tasks keep them.
  Each type has the priority its new tasks start at, which you can change,
  and can have its **Own tab**, which holds its open tasks with no day, the way
  Ideas does; without one, they're in Up Next or Waiting.
- Add your own types there too, such as Bug or Review, each with a tag colour,
  or ask Claude ("add a Bug type, high priority, red", "give GitHub its own
  tab", "turn off follow-ups"). **Edit** renames one or changes its colour;
  removing one turns its tasks back into plain tasks until you add it again.
- A task is marked done only with a short **completion comment** saying what
  was done or decided. The page asks for it too.
- A task can have a **ticket**: a key such as `OPS-142`, or a full link. Full
  links are always clickable. To make keys clickable too, set **Ticket link**
  under **Settings** on the page, or tell Claude where your tickets live.
  `https://yourcompany.atlassian.net/browse/` adds the key at the end;
  `https://linear.app/yourteam/issue/{key}` puts it where `{key}` is. A
  GitHub link shows as `owner/repo#12`, and a ticket written as
  `owner/repo#12` links to that issue or PR on github.com.
- A task or topic can have **links**: pages it points to, such as a claude.ai
  artifact, a doc or a design. Add them in **Edit**, one per line with a name
  before the link, with **Add a link** on a topic, or by asking ("link my Jira
  Dashboard to the Kafka topic"; Claude finds the artifact and its name).
  Artifacts are marked ▣ and other links ↗. A task's row shows its one link,
  or how many it has; its details and the task view list them all.
- A task can have a **priority**: High, Normal (the default) or Low. Pick it
  when you add a task, change it with **Edit** on the task, or say so
  ("add an urgent task…", "that one's low priority"). High tasks show a red
  "▲ High" and go to the top of each list; Low ones show "▽ Low" and go to
  the bottom.
- A task can have a **status note** that changes over time, such as "blocked
  on DNS change".
- A task can be set aside for **someday**: say "someday" or "not now", pick
  Someday as the day when you add it or in **Edit**. It stays out
  of your day plans until you plan it for a day.
- A task can collect **comments**: dated entries with more detail on progress,
  such as what you tried and what's next. The status note says where a task
  stands now; the comments keep the history. Say "add a comment to the DNS
  task: …", or open the task on the page and add one there.
- A task can have a **checklist**: the steps left to do, ticked off as you go,
  each with an optional ticket. Say "add 'check the TTL' to the DNS task's
  checklist", or open the task on the page and add items in its checklist
  pane. A task can be finished with items still open.

## Topics

**Topics** is a screen of its own, for subjects that aren't tasks, such as
"Q4 planning" or "Kubernetes migration". The **Tasks | Topics** switch at
the top of the page swaps between the two. Topics are listed down the left,
under a **New topic** box, at the left edge of the window. The topic you
pick shows its **Checklist** in the middle, with items to tick off and how
many are done, and its **Notes** on the right, which take all the width the
window has to spare: bullet points without dates, each a short point or a
paragraph. Click a note to show its **Edit** and **Remove** right under it,
with **↑**, **↓** and **Move to** to move it. Or drag a note by its grip
(⋮⋮, over its bullet) to another place in the list, or onto another
person's group or Main's, even a folded or empty one.
Write new notes in the box under them: Enter adds the note, Shift+Enter
starts a new line.
A checklist item can carry a ticket too, shown as a chip after its text;
set it in the item's **Edit**.
Under the topic's name, its **overview** says what it's about, and its
**Tickets** link to the Jira tickets, Salesforce cases or other pages it
relates to. A Jira key links through your Ticket link setting; a link can
carry a label, such as "Case 00012345 https://…"; plain text such as a
case number shows as it is. Click the overview, or **Add an overview** or
**Add a ticket**, to change them in the topic's **Edit**.
A **With** dropdown under them says whom it's a joint effort or
discussion with, from the same team as your tasks; the Topics screen has its
own **Settings**, with the same Appearance, Ticket link and Team as the Tasks
screen's (types and lists are about tasks, so they stay there), so you can set
them from either screen.
Each person on the topic has a part of their own, for what they're working
on or questions to follow up with them. The checklist and the notes each
show the topic's own part (**Main**) and then each person's as a group,
under a heading with its count; click the heading to fold the group away or
open it again, and each device remembers it per topic. One box at the
bottom of each adds to the part picked beside its **Add** (Main at first).
**Edit** on a topic changes its title, overview and tickets (one per
line), or removes it. **Hide checklist** beside it hides the topic's
checklists so the notes take the whole width; **Show checklist** (with how many
items are ticked) brings them back, and each device remembers it per topic. On a narrower screen the
notes go under the checklist; on a phone you see the list, then the topic
you tap, with **‹ All topics** to go back. Each device remembers the screen
and the topic.

Claude keeps the same notes and checklists: "start a topic for Q4
planning", "the Q4 topic is about …", "link OPS-142 and Salesforce case
00012345 to Q4 planning", "add a note to Q4 planning: …", "add 'book the room' and 'send
the agenda' to its checklist", "tick off book the room", "add to Sam's notes
on Q4 planning: …", "what's left on Q4 planning?". Plan-my-day and wrap-up
leave topics out unless you ask.

## Team

Tasks are yours unless you say otherwise. To hand some out:

- Add people under **Settings › Team** on the page (Settings is the last item
  in the left pane), or tell Claude ("add Sam to my team").
- Pick who a task is for when you add it, under **For** in a task's **Edit**, or by
  asking ("add a follow-up for Sam to check the logs"). The task then shows
  "for Sam".
- A discussion is always yours, with one or more people: tick them in the
  **With** dropdown when you add it or in the discussion's **Edit**, as many
  as it involves, or ask ("add a discussion with Sam and Priya about the Q3
  budget"). It shows "with Sam and Priya", and it's on each one's 1:1 list.
  Discussions don't offer **For**.
- The **People** filter above the lists shows **Everyone**, **Mine** or one
  person. Picking one person shows a **1:1 with** section first, listing the
  open discussions with them, then the tasks they have. Ask Claude "what's on
  my 1:1 with Sam?" for the same list. Each device remembers its choice.
- The **People** item in the left pane lists the open tasks that involve
  your team, for them or discussions with them, grouped by who's involved
  ("Sam", "Priya", "Priya and Sam"). While People is shown, your team is
  listed under it, each person with their count, and it folds away on every
  other list: pick one to see only their tasks, discussions first, and a
  new task starts as a discussion with them. On a phone the names are a
  dropdown at the top of the list. Each device remembers the person.
- Removing someone from the team gives their tasks back to you and takes
  them off the discussions they were part of.

"Plan my day" plans from your own tasks, and suggests checking in on tasks
someone else has had for 3 days or more. "Wrap up my day" lists what's still
with each person.

## Planning your day

- **"Plan my day."** Claude looks at what's open, high-priority tasks first,
  what was planned yesterday and didn't get done, and follow-ups that have
  waited 3 days or more. It proposes 3 to 5 tasks for today and puts them
  under **Today** once you agree.
- **"Wrap up my day."** Claude summarizes what got done (with the completion
  comments), what carries over and what's waiting, asks whether anything else
  got done, and saves the summary to the **Days** log.

You can also plan by hand, and ahead. The **Day** filter's **Today |
Tomorrow** tabs, and the day picker beside them for any of the next seven
days, choose which day the planned section shows. The star on a task plans it
for that day. A task planned for another day goes under **Upcoming** with a
small "★ Tue" marker. Claude can plan ahead too ("plan tomorrow", "plan
Monday", "push the DNS task to tomorrow"), and offers to plan tomorrow when
you wrap up a day.

## Jira flags

If you use the [Jira Dashboard](https://github.com/primelogiclabio/claude-jira-dashboard)
and flag tickets there for discussion, Claude can bring them into the
organizer. Say "sync my Jira flags" (Claude also does it when it plans your
day once you've synced before). Claude copies the flags in and offers each
new one as a discussion task, with its ticket link and your flag note.

The page lists the flags under a **Jira integration** tab, each with **Add as task**, or
which list it's already on. The tab shows the flags as of the last sync, not
live, because one artifact page can't read another's data. Its **Sync now**
copies "sync my Jira flags" for you to paste to Claude in Claude Code.

The tab shows once a sync has connected a Jira Dashboard (you can hide it
under **Settings › Lists**). Until then, or if Claude finds no Jira Dashboard,
**Settings › Lists** lists the steps to set one up: add the Atlassian connector
on claude.ai, install the Jira Dashboard skill, ask Claude to start a dashboard
and flag some tickets, then sync. The Jira task type is separate: it's for
Jira tickets you work on, and can have its own tab like any other type.

## Updating your organizer

Your organizer keeps the page it was published with, so a plugin update
doesn't change it by itself. The page notes its version in the organizer's
settings, and when you plan or wrap up your day, Claude offers the update if
a newer page has shipped. You can also ask: "update my organizer". From 0.10.0
the settings live at `organizer/settings` rather than `settings/main`, so they
can't clash with another skill's; updating an older organizer moves them.

The update republishes the page at the same link. Your tasks stay, because
they're in the organizer's database rather than the page. Changes you asked
Claude to make to the page are carried over to the new one. For a page older
than 0.4.0, Claude asks which changes those were.

The full instructions are in [`SKILL.md`](SKILL.md).

## When it runs

Claude picks it up when you ask to start, add to, update, complete or review
your organizer, or to plan or wrap up your day. Pointing at it by link, or by
name ("my daily organizer", "my workday tracker", "my task list"), is enough.
It doesn't apply to team or project trackers, or to a repo's `PROGRESS.md`
(that's [project-progress](https://github.com/primelogiclabio/claude-tools/blob/main/plugins/project-progress/README.md)); if it's unclear
which one you mean, Claude asks.

To call it directly, type `/daily-organizer` and choose it from the list. When
it's installed as a plugin, its full name is `/daily-organizer:daily-organizer`.

**Older trackers.** A tracker made with workday-tracker keeps its tasks in the
page's code. Point Claude at it and it offers to move it: the page becomes the
new organizer at the same link and the tasks move into the database. Nothing
moves without your yes.

## Where it works

Claude creates the organizer and changes its tasks with claude.ai's artifact
tools, which read and write the artifact's own database. Claude Code sessions
have them, and so do claude.ai chats. An artifact made inside a chat, without
an artifact link, has no database.

| Where | Works? |
| --- | --- |
| Claude Code CLI, desktop Code tab, VS Code | ✔ full: starts the organizer, changes tasks, plans and wraps up your day |
| The organizer page, wherever you open it (web, desktop, mobile) | ✔ add, edit, star, finish and remove tasks, add status notes, comments and checklist items, keep topics with notes and checklists |
| claude.ai Chat | ✔ reads and changes your organizer's tasks: add, update, finish, plan and wrap up from a chat (type `/daily-organizer` or ask by name). Starting a new organizer or updating its page from a chat isn't tested yet |
| Cowork, mobile chat | not tested yet. Where Claude has the artifact tools it works as in Chat; where it doesn't, it says it can't reach the organizer, and the page still works |
| Claude Code cloud sessions (claude.ai/code) | ✔ only if it's on your claude.ai account and the session has the Artifact tools |

Outside Claude Code, skills need a Pro, Max, Team or Enterprise plan, with
**Code execution and file creation** turned on in **Settings › Capabilities**.

## Install

One line installs it, in either of two places:

- **Inside a Claude Code session** (CLI or desktop Code tab, 2.1.275+), type:
  ```
  /plugin install daily-organizer --marketplace primelogiclabio/claude-tools
  ```
- **From a terminal** (PowerShell, bash; CLI 2.1.292+), run:
  ```
  claude plugin install daily-organizer --marketplace primelogiclabio/claude-tools
  ```

Both add the `claude-tools` marketplace first if you haven't added it yet.

The step-by-step instructions for both paths are in the
[claude-tools README](https://github.com/primelogiclabio/claude-tools#installing).
Notes specific to this skill:

- **As a plain skill, without the plugin wrapper.** Clone this repo and link
  it into your skills directory. Put your clone's path in place of
  `C:\src\claude-daily-organizer`:

  ```powershell
  New-Item -ItemType Junction -Path "$HOME\.claude\skills\daily-organizer" -Target "C:\src\claude-daily-organizer"
  ```

  Remove the link only with `cmd /c rmdir "$HOME\.claude\skills\daily-organizer"`.
  Never use `Remove-Item -Recurse` on a junction.
- **Avoid loading it twice.** If this skill is already saved to your claude.ai
  account under **Customize › Skills**, it already reaches Chat, Cowork and
  mobile, and syncs into signed-in Claude Code sessions as `daily-organizer`.
  Installing the plugin as well adds a second copy,
  `daily-organizer:daily-organizer`, and the two compete. Keep one.
- **Your organizer stays yours.** The skill holds no artifact link or data;
  each person's organizer is their own private artifact.

### Renamed from workday-tracker

The plugin and skill are now `daily-organizer`. An existing install keeps the
old name and stops getting updates, so swap it:

```sh
claude plugin uninstall workday-tracker@claude-tools
claude plugin install daily-organizer@claude-tools
```

On claude.ai, remove `workday-tracker` under **Customize › Plugins** or
**Customize › Skills** and add `daily-organizer`. Trackers you already have
keep working: point Claude at one to move it into the new organizer.

### Moved from claude-tools

Until version 0.3.0 this plugin lived in the claude-tools repo, under
`plugins/daily-organizer`. The marketplace now installs it from here, so
installs and updates work as before. If you linked that folder as a plain
skill, point the link at a clone of this repo instead.

## Developing

From this folder:

```sh
claude plugin validate .
```

There are no automated tests. To try a change, start a new Claude Code
session, ask it to start a daily organizer, then add, finish, plan and wrap
up a few tasks. Delete the scratch organizer afterwards.

## Files

```
claude-daily-organizer/
├── .claude-plugin/plugin.json   lets this folder install as a plugin
├── SKILL.md                     the skill: frontmatter + instructions
├── template.html                the organizer page Claude publishes
├── README.md                    this file
└── LICENSE                      MIT
```

## License

MIT, see [LICENSE](LICENSE).
