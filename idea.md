# Agentic Task Management Log — Seed

A portable setup for always-on, agent-managed task tracking. Works with any AI tool that can (a) load a standing set of instructions and (b) read and write a database.

Three steps: make a table, paste the instruction block into your agent's always-on instructions, optionally schedule a brief.

---

## Step 1 — Make the table

Any database your agent can query works: Notion, Airtable, Google Sheets, Postgres, SQLite, a Markdown file in a repo. The requirement is that it's reachable from *every* conversation thread, not just one.

Seven columns:

| Column | Type | Purpose |
|---|---|---|
| `Item` | text (title / primary key) | The thing itself. One stable name per task. |
| `Verb` | select | `open` · `touch` · `defer` · `close` · `brief` · `missed` |
| `Logged` | datetime | When the row was written. |
| `What` | long text | What happened, in prose. |
| `Next` | long text | The specific next action. |
| `Due` | date | Optional. |
| `Until` | date | Optional. For deferrals — don't resurface before this date. |

Rows are events, not tasks. One task accumulates many rows over its life.

---

## Step 2 — Paste this into your agent's always-on instructions

Claude Code / Cowork: `CLAUDE.md`. Cursor: `.cursorrules`. ChatGPT: custom instructions or a project. Anywhere that loads on every thread.

Replace everything in `<ANGLE BRACKETS>` before using.

```markdown
## Agentic Task Management Log

The task log is <DATABASE NAME> at <LOCATION / URL / CONNECTION STRING>.
<ANY QUERY DETAILS YOUR TOOL NEEDS — table name, ID, auth.>

Columns: Item (title) · Verb (open/touch/defer/close/brief/missed) · Logged ·
What · Next · Due · Until

### Core rules

The log is APPEND-ONLY. Never edit or delete an existing row. Correct a mistake
by adding another row.

Item is the primary key. An item is open if its most recent row's verb is not
`close`.

Write the row DURING the session, before continuing your reply — never at the
end. Sessions have no reliable end. Log only things that are owed; not questions,
not lookups.

Answer what I actually asked FIRST. The log check comes after, never before.
Never make me sit through a status report to ask a question.

### Naming

Before writing a row with an Item value you haven't already seen this session,
list what exists and reuse it:

  SELECT DISTINCT Item FROM <TABLE> WHERE Item <> '—' ORDER BY Item

If an existing Item covers the thing, use that string exactly — including closed
ones. Reopening is a new row on the old name, never a new name.

New names: lowercase kebab-case, 2–3 words, the entity not the action
(`tesla-license`, not `renew-tesla-license-online`). No verbs, no dates, no
status words. The action and status change; the entity doesn't.

If two Items look like the same thing, don't merge and don't rename — tell me.
If I confirm, append a `close` row on the variant with
What = "duplicate of <canonical>", then continue on the canonical name.

### Reading it

At session start, run ONLY this staleness check:

  SELECT MAX(date(Logged)) FROM <TABLE> WHERE Verb = 'brief'

If that date is today, stop — no further log reads.

When you need the open list, pull it slim. Never select What or Next at startup;
prose columns are an order of magnitude larger than everything else combined and
you don't need them to know what's open:

  WITH r AS (
    SELECT Item, Verb, Logged, Due,
      ROW_NUMBER() OVER (PARTITION BY Item ORDER BY datetime(Logged) DESC) rn
    FROM <TABLE> WHERE Item <> '—')
  SELECT Item, Verb, Logged, Due FROM r WHERE rn = 1 ORDER BY Due

Fetch What/Next only for the one item actually being worked.

### Accuracy

Never invent a name, date, provider, address, or amount. If you are not certain,
say so explicitly. Do not state a proper noun as fact unless you saw it in a
primary source during this session. Cite the source for every finding.
```

---

## Step 3 — Optional: the periodic brief

A scheduled job that sweeps your inputs and reports. This is what surfaces the thing you'd otherwise miss — the comment that tagged you, the email with a deadline buried in it.

```markdown
Attention scan. Scope: <PERSONAL / WORK — pick one, don't mix>.

1. Pull the open list with the slim query above.
2. Scan <YOUR SOURCES — e.g. email, calendar, doc comments, notes app> for items
   needing attention that are NOT already in the log. Prioritize what is new
   since the last run. Look for: anything with a deadline, anything where
   someone is waiting on me, anything requiring a decision.
3. DO NOT write open/touch/defer/close rows. I decide what gets logged. Propose
   findings in your reply only.
4. Write exactly ONE row: Item = "—", Verb = `brief`, Logged = now, What = the
   time each source was reached plus a one-line summary. Under ~600 characters.
   It's a watermark, not a report.
5. Report briefly: new candidates with source and date; anything overdue or due
   within 3 days; anything open and untouched for 7+ days. For each new
   candidate, propose the exact Item slug it would get, checked against the
   existing list. If nothing needs me, say exactly that and stop.
```

Cadence: hourly is aggressive but works if step 3 is enforced and step 4 stays short. Daily is the safe default. The brief must be allowed to say "nothing" — otherwise it becomes another notification source and defeats the purpose.

---

## Things worth knowing before you start

**Display names are not column names.** Most databases expose a different identifier to queries than the one you see in the UI (Notion date fields become `date:Logged:start`; Airtable has field IDs). Find yours once and write it into the instructions verbatim. Otherwise every session pays for a failed query plus a schema fetch.

**Measure your context cost.** Prose columns dominate. In a real 24-item log, `What` and `Next` were ~14,000 characters against ~1,000 for everything else — 13x, paid on every session, to answer a question needing four columns. That's why the slim query exists. Measure yours rather than assuming.

**Let the agent write rows itself, or it won't happen.** The whole value is that logging is a byproduct of working. If a human has to approve every row, you've rebuilt Asana with extra steps. The `brief` job is the exception — a scan that proposes rather than writes keeps automated sweeps from polluting the log.

**Verify capability claims, and date them.** If you write "X doesn't work" into your instructions, the agent will read it, not try, and confirm it. That belief then never gets retested. When you record a limitation, record the date you verified it — and when you record a method that works, do the same.

**Match granularity to what you actually finish.** An item covering a whole checklist stays open forever, because something on the list is always unfinished, and it stops carrying information. One item per thing you can close.

---

## Adapting it

- **Verbs** are the main dial. Six is enough for personal use. Add `blocked` if you're tracking work with real dependencies; drop `missed` if you won't use it honestly.
- **`Until`** is what makes deferral real rather than a rename of "ignore." An item deferred until a date shouldn't surface before it.
- **Scope one log to one domain.** Personal and work tasks have different sources, cadences, and privacy needs. Two tables beats one table with a `Context` column.
- **Start with the table and the core rules only.** Add the brief once the log has enough in it to be worth briefing on.
