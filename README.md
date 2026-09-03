# Agentic Task Management

In a high-velocity company you can spend an entire day feeling productive — clearing Slack, answering pings, putting out fires — and finish with none of your real work done. Legacy communication tools are built to pull you in; the notifications are the product. So high performers spend real effort defending their attention: blocking their calendars, maintaining to-do lists and priority stacks, deciding over and over what deserves a response.

But that defense is itself manual work, and it costs the exact thing it's meant to protect. How many times have you recommitted to tracking your tasks in Asana or Wrike, only to fall off the wagon a few weeks — or a few days — later? You want the *output* of task management. Producing it by hand drains the attention you were trying to save. That gap is the tell-tale sign that a job should be delegated to an agent.

So: how do you stand up an agentic workflow that manages your tasks continuously, without you?

> **Want to build one?** [`idea.md`](idea.md) lays out the concept — the principles the log rests on, the minimal shape it needs, and the decisions you'll make for yourself. It's deliberately tool-agnostic and leaves the implementation to you.

## My principles

- **Work in one place.** I want to spend my time in AI conversation threads, not flipping between Slack, email, and Asana. The system's job is to protect my attention so I can do deep work — not to add another queue I have to check.
- **Near-zero overhead.** I want to spend as little time on task management as possible. The agent should be doing it almost constantly, in the background of whatever else I'm doing.
- **Portable.** It has to work across any LLM system. I use different AI tools at work and at home, and the system shouldn't care which one I'm in.

## The basic components

**A task log.** A headless table the agent reads from and writes to. Any database an AI can query will do; I use Notion for my personal life. The point is a single source of truth reachable from every conversation thread — and one that *you* own. A list that lives inside a legacy tool slowly becomes a list of that tool's notifications.

**Standing instructions.** The rules every thread loads: when to add a row, how to tell what's open, what counts as owed. In my Claude setup these live in Cowork's global instructions. What makes task management always-on rather than a thing I remember to do is that the same instructions and the same log are reachable from everywhere.

**A periodic sweep (optional).** A scheduled job that scans your inputs and reports on a cadence. In practice this is how I learn that something real came in through a channel I wasn't watching — an important Slack, a comment on a doc that tagged me. It proposes; I decide what gets logged.

## How the log works

Each row records which item it's about, what kind of event it is, when it was written, and a note about what happened, with optional dates for when it's due and, for deferrals, when it should resurface. The event kinds are a small fixed vocabulary — mine is `open`, `touch`, `defer`, `close`, `missed` — and an item's name is stable for its whole life.

Two design choices do most of the work.

**The log is append-only.** No row is ever edited or deleted. When something changes, the agent adds a row. An item is open if its most recent row isn't a `close` — which turns "what's on my plate" into a query instead of a judgment call, and preserves the full history of every task for free.

**Rows get written mid-conversation, not at the end.** This is the least obvious rule and probably the most important. Threads have no reliable ending: you get distracted, the session dies, you close the tab. An agent that saves its logging for the end logs nothing a surprising share of the time. So it writes the row the moment a task becomes real, before finishing its reply.

Two smaller rules earn their keep. Only things actually *owed* get a row — questions and lookups produce nothing, because a log that captures everything is a log you stop reading. And the agent answers my question first and checks the log second, so I never sit through a status report to ask something.

## Why I can't go back

- Task management now happens as a byproduct of working. Every thread I'm in has the capability, so my list updates itself while I do something else.
- I've stopped monitoring my communication tools. When I'm tagged into a doc full of to-dos, those items simply appear in my log — I don't go hunting across apps to find out what I owe.
- My work has a home. One environment, one list, one place to look.
- Nothing is ever overwritten, so I can reconstruct why something sat untouched for three weeks. A to-do app that only shows current state can't tell me that.

## Build your own

Start with [`idea.md`](idea.md). It's the concept, not a recipe — the table, the verbs, and the instructions are yours to shape.
