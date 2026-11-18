---
name: explain-page
description: Turn a lot of information into a plain, top-to-bottom explainer page with diagrams, written for a reader who lacks the context
argument-hint: "[topic or focus, optional]"
disable-model-invocation: true
---

Make an explainer page about: `$ARGUMENTS` (empty means the subject of this conversation).

The reader is a **cold reader**: smart, short on time, switching in from other work, and missing the context on these systems, tables and acronyms. Every choice below serves that reader.

## Steps

### 1. Gather the facts

Collect what the page will say from the source the user is reacting to (an email, a Slack thread, a ticket, a PR) and from this conversation's findings. When the facts are not established yet, read the source and check each claim against the code or live data first.

Done when every fact you plan to put on the page traces to something read or measured in this session, and every claim you could not check is on a list of unknowns.

### 2. Make the name list

Before writing, list every thing the page will mention. For each one write:

- **Page name**: the name the source uses (the table name in the email, the job name in the ticket). The source's name is always the page name, so the reader can match the page to what they already read.
- **Plain line**: one short sentence saying what it is, shown beside the name the first time it appears.
- **Engineer name**: the internal term (layer names, repo names, DAG names), used only in the glossary.

Use one page name per thing, the same every time.

Done when every system, table, job and term on the page has a row.

### 3. Build the page

1. Run `Artifact` with `action: "quickstart"` and `intent: "other"`, then load the `artifact-diagramming` skill.
2. Write one HTML page that follows **Page sections** in order, using the name list and **Writing rules**.
3. Publish it.

When the agent has no `Artifact` tool, write the same page as one self-contained HTML file (inline CSS and SVG, no external requests) and give its full path instead of a link.

Done when the page is published (or the file is written) and each section that has something true to say is on it.

### 4. Reply

Reply with the link, one line per section saying what it covers, and a note that the page is private until the user shares it. When the user needs to decide something, end with that one question.

## Page sections

Follow the data or the events from where they start to where people see them. Leave out a section that has nothing true to say.

1. **The short answer.** Two to five rows, each a status pill (for example Safe, Act, Bug, Unknown) and one sentence.
2. **What happened.** The source message in its own words, and the decision it asks for.
3. **How it works.** A flow diagram that follows the path from origin to where people use it. Label each arrow with what moves and when. Color only the parts where a risk sits.
4. **The key concept.** One real example taken apart piece by piece (an id split into its fields, a request split into its hops).
5. **Why it is or is not a problem.** The answer first, then the reason.
6. **What can go wrong.** One card per risk: a severity pill, a plain explanation, a diagram when it shows a mechanism, one real example with real values, and the fix in one line.
7. **What we don't know.** Each unknown, why it matters, and who can answer it.
8. **Background.** Why the other party is doing this, when that helps the reader follow.
9. **Next steps.** Numbered asks grouped by who to ask, a box with the user's decision, and a ready-to-send message with a copy button.
10. **Glossary.** Three columns: page name, engineer name, what it is.

## Writing rules

- Stay inside the source's scope: the tables, dates and changes it names. Each risk and each ask follows from a line in the source; a past event earns a place only when it shows how this change can fail.
- Define each thing in its plain line the first time it appears.
- Write short sentences in the active voice, with plain everyday words (use, help, start, stop, check).
- Use real values and real examples from the investigation. Label any made-up illustration "Example figures".
- Write "we don't know" where the evidence is thin, and say who can find out.
- Keep lists flat: one level.
- Show ticket numbers, job IDs and file paths only where the reader needs them to act.

## Diagram rules

- Draw one diagram per mechanism: a flow for a path, a timeline for "when", a before/after for a change, to-scale bars for sizes and durations.
- Label every arrow with what moves and when.
- Draw times and sizes to scale.
- Color carries meaning only: green safe, orange risk, red broken, one accent for the thing under discussion.
