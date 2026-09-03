# A Commitment Log for an AI Agent

Most writing about "memory" for AI agents is about recall: how to give the model a place to stash context so it can find it again later. This is a different idea. It's a log of things you owe, kept by the agent as a byproduct of working with you, and read back by the agent so it can tell you what's slipping.

The distinction matters. A memory store is something the agent dumps into and searches. A log is a record the agent has to keep faithfully: every commitment gets a row, every row stays, and the state of the world is whatever the rows add up to. The agent doesn't get to remember selectively, and it doesn't get to tidy up history.

The rest of this document is the handful of principles the idea rests on, the minimal shape it needs, and the decisions you'll make yourself when you build one.

---

## The principles

**Rows are events, not tasks.** A task is not a row that gets updated as it progresses. It's a name that accumulates rows over its life: opened, touched, deferred, closed, maybe reopened. Nothing is ever edited or deleted. If a row was wrong, the correction is another row. This is what makes the log trustworthy: the history can't be rewritten by the agent, and anything you see in it actually happened in that order.

**Every thing has one stable name.** The name is the key that ties the rows together, so it has to be the *entity* rather than the action or the status. The action changes, the status changes, the entity doesn't. Before the agent coins a new name it should check what already exists and reuse it, including names that were closed long ago. Reopening something is a new row on the old name, never a new name. When two names look like the same thing, the agent flags it and you decide; it doesn't merge or rename on its own.

**Open means the latest row doesn't say closed.** There's no status column to keep in sync. The current state of any item is derived from its most recent row. This falls out of the append-only rule and it's what keeps the log honest: you can't mark something done without leaving a record of having done so.

**The agent writes the row itself, during the work.** The whole value is that logging costs you nothing. If you have to approve every row, you've rebuilt a task manager with extra steps. So the agent writes the row the moment a commitment appears, before it continues its reply, not at the end of the session, because sessions don't reliably end. And it logs only things that are owed. Questions, lookups, and conversation aren't commitments.

**Answer first, log second.** The log exists to serve the conversation, not to interrupt it. The agent answers what you asked, then attends to the log. You should never have to sit through a status report to get a question answered.

**Read as little as possible.** The agent doesn't need the whole log to know what's open. It needs the name, the latest verb, and the dates. The prose, what happened and what's next, is only needed for the one item actually being worked, and it's by far the most expensive part of the log to read. A log that gets loaded in full on every session becomes a tax on every session.

---

## The minimal shape

You need surprisingly little. Each row records which item it's about, what kind of event it is, when it was written, and a note in prose about what happened. Two optional dates are useful: when the item is due, and, for deferrals, a date before which it shouldn't resurface. That last one is what makes deferral mean something rather than being a polite word for ignoring.

The event kinds are a small fixed vocabulary. Something like open, touch, defer, close is enough to start; you'll know within a few weeks whether you need more. The vocabulary is a dial, not a spec.

Where the rows live doesn't matter much, as long as the agent can read and write it from every conversation, not just one. A Notion database, a spreadsheet, a real database, a file in a repo all work. Pick whatever your agent can already reach.

---

## Decisions you'll make

The principles above are the parts worth keeping. Everything else is yours to steer, and most of it you'll only get right by watching what your own log does.

**Granularity.** One item per thing you can actually finish. An item that covers a whole checklist stays open forever, because something on the list is always unfinished, and it stops carrying information. Where you draw the line depends on how you work.

**Verbs.** Start small. Add one only when you catch yourself wanting to say something the existing verbs can't. If you track work with real dependencies you might want a way to say blocked; if you'll never honestly record that you missed something, don't give yourself a verb for it.

**Scope.** Personal and work commitments have different sources, cadences, and privacy needs. One log per domain tends to beat one log with a context column, but that's a preference, not a rule.

**How much the agent reads at startup.** The cheapest workable pattern is a single check of when the log was last reviewed, and nothing more unless that's stale. How stale is too stale, and what the agent does about it, is up to you.

**Whether anything sweeps for you.** Some people add a scheduled pass that scans their inbox, calendar, and notes for things that ought to be in the log and aren't. If you do, the useful constraint is that the sweep proposes and you decide; an automated job that writes commitment rows on its own will fill the log with things you didn't agree to. It must also be allowed to report nothing, or it becomes one more notification source.

**What you write into the instructions.** Whatever standing instructions your agent loads become beliefs it acts on. If you write down that something doesn't work, the agent will read that, not try, and confirm it, and the belief never gets retested. Date the things you assert, both the limitations and the methods that worked.

---

## Why bother

You could keep a task list by hand. The reason to hand it to the agent is that commitments mostly surface inside conversations, and the moment they appear is the only moment they're cheap to capture. A log the agent maintains as it goes captures them at that moment, refuses to forget them, and can be asked, at any point, what you owe and to whom. That's a different thing from a to-do app, and it's a different thing from memory.
