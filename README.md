# daily-organizer

A Claude skill that keeps your personal daily organizer on claude.ai: Today,
Up Next, Waiting On and Done lists that you and Claude both update, with a
morning plan and an end-of-day summary.

Ask Claude to start one and it publishes the page from
[`template.html`](template.html) and gives you the link. A **Days** log keeps
past plans and end-of-day summaries. It installs from the
[claude-tools](https://github.com/primelogiclabio/claude-tools) marketplace.

Tabs show one list at a time, each with a count: **Today**, **Up Next**,
**Waiting**, **Upcoming**, **Someday**, **Ideas**, **Jira** and **Done**. **All** shows
every list stacked on one page. Ideas get their own tab, so Up Next holds only
work to do. **Upcoming** holds tasks planned for later days, and **Someday**
the ones you've set aside with no day. Unfinished tasks from earlier days show
as **Carried over** under Today, marked "from Oct 9 · 23 days"; give one a new
day with **Edit**.

The header shows today in four figures (open today, carried over, to chase,
done today) and a bar for how much of today's plan is done. "To chase" counts
follow-ups and waiting tasks that are 3 or more days old.

A bar above the lists holds the **Day** and **People** filters and a
**List | Table** switch, which shows each section as a list or as a table
with columns for type, priority, ticket, age and comments. Rows light up under
the pointer; click one to open its details: every field, the status note in
the table, and its comments. Opening a row closes the one already open. Each device
remembers its own tab and layout, so a phone can stay on the Today list while
a desktop shows All as tables. The **Add a task** box above the tabs folds
away when you don't need it, and each device remembers that too. Its **Day**
picker starts on today, and can plan a new task for one of the next six days,
**Someday** or no day; **+ Note** adds a status note as you add it. Tasks Claude added from a chat
say "via chat". The bottom of the page shows its version.

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
  too. Each comment has its own **Edit**.
- Claude reads and changes the same records when you ask, without rewriting
  the page, so asking for "add a follow-up to chase the DNS change" is quick
  and can't drop other tasks.
- The records stay when the page itself is updated.
- Only you, and anyone you make an Editor, can see or change them, even if
  you share the page's link.

## The rules it keeps

- Every task has a **type**: `task`, `idea`, `follow-up` or `discussion` (a
  topic to talk through with someone, such as a point for a 1:1). Claude asks
  when it isn't clear. A discussion can say who it's with (see [Team](#team)).
- A task is marked done only with a short **completion comment** saying what
  was done or decided. The page asks for it too.
- A task can have a **ticket**: a key such as `OPS-142`, or a full link. Full
  links are always clickable. To make keys clickable too, set **Ticket link**
  under **Settings** on the page, or tell Claude where your tickets live.
  `https://yourcompany.atlassian.net/browse/` adds the key at the end;
  `https://linear.app/yourteam/issue/{key}` puts it where `{key}` is.
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

## Topics

**Topics** is a screen of its own, for subjects that aren't tasks, such as
"Q4 planning" or "Kubernetes migration". The **Tasks | Topics** switch at
the top of the page swaps between the two. Topics are listed down the left,
under a **New topic** box. The topic you pick shows its **Notes** in the
wide middle pane, dated entries you can edit or remove, and its
**Checklist** on the right, with items to tick off and how many are done.
**Edit** on a topic renames or removes it. On a narrower screen the
checklist goes under the notes; on a phone you see the list, then the topic
you tap, with **‹ All topics** to go back. Each device remembers the screen
and the topic.

Claude keeps the same notes and checklists: "start a topic for Q4
planning", "add a note to Q4 planning: …", "add 'book the room' and 'send
the agenda' to its checklist", "tick off book the room", "what's left on
Q4 planning?". Plan-my-day and wrap-up leave topics out unless you ask.

## Team

Tasks are yours unless you say otherwise. To hand some out:

- Add people under **Settings › Team** on the page, or tell Claude ("add Sam
  to my team").
- Pick who a task is for when you add it, under **For** in a task's **Edit**, or by
  asking ("add a follow-up for Sam to check the logs"). The task then shows
  "for Sam".
- A discussion can say who it's with: pick them under **With** when you add
  it or in the discussion's **Edit**, or ask ("add a discussion with Sam
  about the Q3 budget"). It shows "with Sam" and stays your own task.
- The **People** filter above the lists shows **Everyone**, **Mine** or one
  person. Picking one person shows a **1:1 with** section first, listing the
  open discussions with them, then the tasks they have. Ask Claude "what's on
  my 1:1 with Sam?" for the same list. Each device remembers its choice.
- Removing someone from the team gives their tasks back to you, and their
  discussions no longer say who they're with.

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

The page lists the flags under a **Jira** tab, each with **Add as task**, or
which list it's already on. The tab shows the flags as of the last sync, not
live, because one artifact page can't read another's data. Its **Sync now**
copies "sync my Jira flags" for you to paste to Claude in Claude Code.

The tab is always there. Until a sync connects it, or if Claude finds no Jira
Dashboard, it lists the steps to set one up: add the Atlassian connector on
claude.ai, install the Jira Dashboard skill, ask Claude to start a dashboard
and flag some tickets, then sync.

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

Claude creates the organizer and changes its tasks with the `Artifact` and
`ArtifactData` tools. Claude Code sessions have both. Chat artifacts on
claude.ai have no database.

| Where | Works? |
| --- | --- |
| Claude Code CLI, desktop Code tab, VS Code | ✔ full: starts the organizer, changes tasks, plans and wraps up your day |
| The organizer page, wherever you open it (web, desktop, mobile) | ✔ add, edit, star, finish and remove tasks, add status notes and comments, keep topics with notes and checklists |
| claude.ai Chat, Cowork, mobile chat | ⚠ can't change the organizer's tasks. Claude can still keep an older-style tracker there, with the tasks in the artifact's code and the same rules |
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
