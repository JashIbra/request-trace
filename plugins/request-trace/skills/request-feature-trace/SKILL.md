---
name: request-feature-trace
description: Feature walkthrough — one user action followed through EVERY call in order, changed or not, with each function, variable and command explained the way a senior colleague explains it to a junior, and the work this feature added marked along the way. Use when the user types /request-feature-trace, or asks for the whole path of a feature "from the button press", "every function in order", "explain it like to a junior". For a review of a branch's changes only, use `request-trace`.
---

# Feature trace — the whole path, for someone new to it

The reader presses a button and wants to know **everything that runs after it, in order**, and what each
piece is. They are not reviewing a diff; they are learning how the feature works. The trace still shows
which parts this feature added, so they also learn what the feature consisted of.

## What to do

1. **Load the `request-trace` skill and follow it**, with the rules below taking precedence wherever the
   two disagree. Use the Skill tool with `request-trace`. Everything about the page, the links, the
   footnotes, the language, naming things instead of pointing, explaining a name where it first appears,
   and opening every bullet with what happens there stays in force.
2. **Then apply the rules below.**

Do not copy or paraphrase the other skill's rules into your answer. Do not mention that there are
variants. Do not ask which one the user wants.

## What changes against `request-trace`

- **Every call on the path is a step, changed or not.** Start at the control the person presses, named
  as the screen shows it ("Включить режим киоска"), and follow the code in the order it runs: the
  handler, each function it calls, each step of a sequence, each command sent to another system, until
  the visible result. An unchanged step is not cut and not shortened to one line; the reader needs it to
  understand what comes next. Stop at the platform's or a library's boundary, and say what happens
  beyond it in one sentence.
- **Each function gets the questions a junior asks.** What it does, in one plain sentence. Who calls it
  and at what moment — what the person sees on screen at that moment. What it receives and what it
  returns. What it does with the result of what it calls.
- **Each variable, parameter, field and constant in a step gets a line**: what it holds, where its
  value comes from, what reads it later. Skip only loop counters and values whose name and use need
  nothing more (`val result = shell(...)` checked on the next line).
- **A field of a state object gets every place that writes it.** When a step reads a field of a screen's
  state — `running`, `downloading`, `finished` — name each function that sets it, the action that runs
  that function, and the value, with its line. Find them with a search for every assignment, not from
  memory — a missed writer is the bug the reader is hunting: "`true` when the download starts (:238), `false` when it ends or fails (:256, :265, :272)". If
  the state is an immutable data class updated through `copy(...)`, say so once, so the reader does not
  take a `val` with a default for a constant.
- **Commands sent to another system are shown as they are sent**, assembled from the code: the exact
  `adb shell pm grant com.app.tahfeez.kiosk android.permission.READ_CONTACTS` line, the exact HTTP
  request. Then say which part of the command comes from which variable.
- **Write as a senior colleague explains to a junior.** Use the real terms — Device Owner, coroutine,
  adb, app op — and explain each one the first time in plain words. Assume the reader knows the
  language, not this project or this platform's corners.
- **Mark what this feature added.** Keep the state chips from `request-trace` on every bullet, decided
  from git against the base branch. "New" and "changed" show what the feature consisted of; "unchanged"
  is no longer a reason to cut a step, only a label.
- **Open the page with what the feature did**, under the title: three to six plain lines, one per piece
  of work the feature added — the facts, no commentary, no count of steps. This is the one exception to
  the no-preamble rule.
- **Length follows the path.** There is no limit of thirty steps; a long path gets more steps, grouped
  under headings by stage. Do not pad: a step still has to be a place where something happens.

## Before the page goes out: a junior reads it, a senior answers

Every step is checked by two agents before the page is published. Do not skip this and do not do it
yourself in your head — your own text always reads clear to you.

1. **The junior.** Spawn an agent (Agent tool, general-purpose) that plays a junior: about half a year
   of the language, new to this project and this platform, **with no access to the code** — it gets only
   the text of the steps, rendered as plain text, and the step before them for context. Paste the text
   into the prompt itself; never send a placeholder. It answers, per step: what it understood in its own
   words, whether it sees why the step is in the walkthrough, and the questions it would ask, most
   important first.
2. **The senior.** Spawn a second agent that plays a senior mentor **with read access to the code**. It
   gets the same steps and the junior's answer, checks every answer against the code, answers the
   junior, and then says what in the text caused each question and rewrites the step so the question
   would not arise. It also reports anything the code shows that the text got wrong or left out — a
   missed writer of a field, a condition with no value, a real bug.
3. **Apply and repeat.** Fix the steps from the senior's report — after checking its claims against the
   code yourself, since it is a model too — and run the junior again on the changed steps. A step is done
   when the junior can say what happens there and why the step is in the walkthrough, and its remaining
   questions are about depth rather than meaning.

Batch the steps, about six to eight per junior run, so a long walkthrough does not take one agent per
step; send the batches in parallel. Bugs the senior finds in the code go into the chat for the user, not
silently into the page.
