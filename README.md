# request-trace

Two Claude Code skills for reading unfamiliar code before you review it:

- **`request-trace`** walks **one user action through the code** — from the tap in the client to the
  response, and on into whatever happens after it — and spells out the mechanics of every call along
  the way.
- **`explain-line`** explains **one line of that code by question and answer**: a one-sentence answer
  first, then only what you ask next. See [Going deeper](#going-deeper-explain-line).

Both come in one plugin.

`request-trace` exists for code review. A list of changed files tells you where to look; it does not tell you
what runs after what. This produces the route, and under each stop it answers, in advance, the
questions a reviewer is about to ask: what is this function, where did this variable come from, why
is the code here at all.

## What it produces

The sample below is a Kotlin client and an ASP.NET backend, but the skill works with any language;
on another stack the notes explain that stack's libraries instead.

````markdown
## 6. Validator `CreatePostRequestValidator` → 400

- File introduced in this branch, a FluentValidation[¹²](post-scheduling/12-FluentValidation.md) validator class; the rule sits at
  [CreatePostRequestValidator.cs:14](../../Application/Validators/CreatePostRequestValidator.cs#L14).
- No code in the project calls it — ASP.NET Core[¹³](post-scheduling/13-ASP.NET-Core.md) does: `AddValidatorsFromAssemblyContaining`[¹⁴](post-scheduling/14-AddValidatorsFromAssemblyContaining.md)
  registers it at startup, and `AddFluentValidationAutoValidation`[¹⁵](post-scheduling/15-AddFluentValidationAutoValidation.md) makes the framework run it on
  each request, before the controller's first line.
- `RuleFor(x => x.PublishAt)`[¹⁶](post-scheduling/16-RuleFor.md) — the checks chained after it apply to the `PublishAt` field. The
  compiler hands the library a description of the lambda rather than its code — an expression tree[¹⁷](post-scheduling/17-Expression-tree.md) —
  so the library reads the field's name without running anything and names it in the 400 body,
  telling the client which field is wrong.

…
````

And one of its note files, `post-scheduling/12-FluentValidation.md`, opened from the ¹²:

````markdown
# ¹² FluentValidation

FluentValidation — a third-party .NET library for validating objects: the rules are written as
code in a validator class rather than as attributes on the properties.
````

Every entry carries a file and a line as a link, so the reader jumps straight to the code instead of
scrolling for it. Every name the project did not write carries a small superscript number, a link to
a note that says what kind of thing it is, where it comes from and why that source exists, and what it
does — each note readable on its own. The note opens as a small file in its own tab, beside the trace.

The trace goes into a Markdown file, not the chat — `.claude/traces/<branch>.md` when `.claude/` is
git-ignored — so it never shows up in the diff and reads comfortably next to the editor. The chat
gets only the file's path. Links are relative to that file, with the line as a `#L124` anchor. Each
footnote is also written to its own small file beside the trace, and the marker links there.

## What it insists on

- **The unit is a step, not a file.** One method can produce five consecutive entries; an untouched
  file still appears if the request goes through it.
- **Every bullet opens with its state, in bold** — new in this branch, unchanged, changed, *now
  meaning something else*, or removed — decided from the git diff, not from memory. The fourth is
  what review misses most often: the diff is tiny or absent while the meaning has moved.
- **Unchanged code only when it matters.** An untouched place stays only if the new behaviour
  depends on it, a refusal happens there, or the route would break without it — and then in one
  line. Everything else is noise a reviewer has to read past.
- **Everything the project did not write gets a footnote.** `RuleFor`, `flatMapLatest`,
  `encodeDefaults`, `timestamptz` — a superscript number at the first mention, and a numbered note at
  the end: what kind of thing it is, which library it belongs to and what that library is for, and
  what it does. Libraries themselves are named in the text and get notes too. The bullet keeps to
  how this code uses the thing.
- **Refusal points are flagged with their status codes**, and so is anything running before the
  controller, which is easy to miss because nothing calls it explicitly.
- **No diagrams, no preamble, no summary.** A sequence diagram of the same trace holds less and
  cannot be pasted into a ticket.

Both skills are written in English so they can be shared; what they produce follows the language the
person asked in.

## Going deeper: `explain-line`

The plugin ships a second skill for the moment a trace entry is not enough. It explains **one line
of code by question and answer**: a one-sentence answer first, then only what you ask next, so you
choose how deep to go.

```
/explain-line 6.3
```

In a conversation that already has a trace, that drills into the third bullet of step 6 — the
`RuleFor` line shown above. It also takes a pasted line of
code or a `file:line`, with no trace at all.

What it insists on:

- **The direct answer first** — "Yes", "No", "In `Kind`" — then one to three lines.
- **A wrong restatement is corrected with the first word.** "So it checks that the time did not
  come?" gets "No", not a polite detour.
- **Real values from your project**, not categories: "20:00 in Berlin goes out as `18:00Z`".
- **The equivalent in the language you write every day** when you ask for it.
- **Checked against the code before answering**, including the setup the answer depends on — a
  serializer's converters, a framework default.
- **No headers, no footnotes, no "want me to go deeper?"** — the next question is yours.

## Install

```
/plugin marketplace add JashIbra/request-trace
/plugin install request-trace@request-trace-marketplace
```

Or, without the plugin machinery, copy each skill's folder from `plugins/request-trace/skills/` into
`~/.claude/skills/` — `request-trace/SKILL.md` and `explain-line/SKILL.md`. Personal skills are picked up at session start, with no
install step — you just lose versioning and updates.

## Use

```
/request-trace sending a chat message
```

With no argument it reads the current branch — `git log origin/main..HEAD` plus the diff — and traces
the feature the branch was cut for, naming its choice on the first line. On `main`, or when the diff
holds several unrelated edits, it asks instead of guessing.

## License

MIT
