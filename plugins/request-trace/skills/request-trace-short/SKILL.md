---
name: request-trace-short
description: A short request trace — the same numbered walk of one user action through the code as `request-trace`, cut to about ten steps with one-sentence bullets, for a reader who wants the route at a glance rather than every mechanism. Use only when the user types /request-trace-short or asks for a "short", "brief" or "condensed" request trace. For the full trace use `request-trace`.
---

# Request trace — short

This is the **same trace** as `request-trace`, cut down. The route, the order, the honesty rules stay;
what goes is depth.

## What to do

1. **Load the `request-trace` skill and follow it**, with the limits below taking precedence wherever
   the two disagree. Use the Skill tool with `request-trace`.
2. **Then apply the limits below** to what you write.

Do not copy or paraphrase the other skill's rules into your answer. Do not mention that there are
variants. Do not ask which one the user wants.

## The limits

- **About ten steps, twelve at most.** Keep the steps that carry new or changed code, plus an unchanged
  step only where the new behaviour depends on it. Merge neighbouring places in one class into one
  step when they serve one purpose; drop the rest instead of summarising them.
- **Two to five bullets a step, one sentence each.** A bullet that needs a second sentence to stand up
  says too much; keep the fact a reviewer acts on and drop the mechanism behind it.
- **Steps grouped under three or four stage headings**, not more.
- **Two or three screen fragments in total**, only where a step changes what is drawn. No object taken
  apart with callouts, and no diagram unless the step cannot be understood without its number.
- **Footnotes only for names a step is built on, eight to ten at most.** Each note is one or two
  sentences: what it is and where it comes from, then what it does.
- **The page's head is the title and one line of facts** — branch and base, the counts and limits a
  reviewer will check.
- **The chat gets one line** with the file's path, as in the full trace.

## What does not change

- The subject, the order, and the first-step rule: start at the human's action and follow it to the end.
- The state label on every bullet, decided from git; a changed or removed thing still says why, in the
  same bullet.
- File-and-line links on every step, checked against the live file, never on a blank line.
- The numbers: every count, limit and timeout that stays in the text stays exact.
- The language: the reader's, with symbols, paths and status codes as the code spells them.
- The HTML page, published as an Artifact and opened.
