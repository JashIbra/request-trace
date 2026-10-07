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
