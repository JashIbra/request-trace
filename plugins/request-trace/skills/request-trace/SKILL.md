---
name: request-trace
description: Request trace — a numbered walk of one user action through the code, from the tap in the client to the response and what happens afterwards, with the mechanics of each call spelled out underneath. Use when someone asks for the path of a request, the call chain, "what calls what", "walk me through it", or is about to review an unfamiliar feature.
---

# Request trace

The reader wants to **walk the code in the same order the request does**, and to understand what
happens at each stop. Their editor is already open. Give them a route with a breakdown, not a
retelling.

They read it while reviewing a change. So the job of the sub-bullets is to **answer, in advance,
the questions a new name will raise**: what is this function, where did this variable come from,
why is this field here. Every such question left open comes back as a separate message.

That splits two things it is easy to conflate: **order** comes from the request's path, **selection**
of what to explain comes from the branch's diff. A step is not the same as a changed file — one
method can produce five consecutive steps, and a file with no edits at all still belongs in the
list if the request passes through it.

## Write in the reader's language

This file is in English so it can be shared. The trace is not. **Produce it in whatever language
the person wrote to you in** — asked in Russian, answered in Russian.

What is never translated: symbol names, file paths, HTTP verbs, status codes, library calls.
`RuleFor`, `encodeDefaults`, `POST /api/...`, `409` stay exactly as the code spells them, whatever
the surrounding prose is.

## Write it to a Markdown file

The trace is long and is read next to the editor, a step at a time — so it goes into a `.md` file,
not into the chat. The chat gets one line: the file's path.

- **Where:** inside the repository the links start from, in a folder git ignores, so the file never
  shows up in the diff. Check with `git check-ignore`; `.claude/traces/` when `.claude/` is ignored.
  If nothing suitable is ignored, ask where to put it.
- **Name:** after the branch or the action — `global-repetition-scheduled-time.md`. Tracing the same
  thing again overwrites the file; that is the point, the old numbers are stale.
- **Links are relative to the file itself**, so from `.claude/traces/` they start with `../../`.
  Each footnote goes into a small file of its own in a folder beside the trace, and the trace has
  no notes list at the end (see the marker rule below).

## What to trace when you are not told

The subject is one human action, followed through to the end. Take it in this order:

1. **Named in the request** — "scheduling a post", "sending a chat message". Use that.
2. **Not named — read the current branch.** `git log origin/main..HEAD` plus the diff: a branch is
   cut for one feature, and that feature is almost always what they want traced. Name what you
   picked on the first line, before the list: "Path of the request when a task is sent." One line,
   not a paragraph.
3. **The branch says nothing** — you are on `main`, or the diff holds several unrelated edits.
   Ask which action to walk. Do not guess: a trace of the wrong request is useless entirely, not
   partly.

If the action spans two requests in a row (create, then confirm), trace the first and say in one
line that the second is its own path.

## The unit is a step, not a file

One entry is one place where something happens to the request. Not one file, not one class.

A list of files, even ordered by importance, is not a trace. It does not answer the question the
reader actually has, which is what runs after what.

When several distinct things happen inside one method, that is several entries, each opening with
**"Same method:"**. That is how the reader sees you have not moved to another file.

## Format

The entry heading carries the number, the layer and the symbol. Underneath, dashed bullets, one
fact per line.

```text
## N. Layer, `Class.Method()`

- first fact about what happens here
- second fact
```

- Layer: Client, Validator, Controller, Service, Repository, Background job. Entries opening with
  "Same method:" do not repeat it — it has not changed.
- The symbol goes in backticks, **with the method**. A class name alone does not say where to look.
- Write the network call as it is: `POST /api/posts`.
- **File and line are required** — as a link, `[File.cs:124](../../path/File.cs#L124)`: the line in
  the text, the `#L124` anchor in the target. A viewer that knows the anchor opens the file at that
  line, which is the whole point — the reader compares against the code instead of scrolling for
  it; one that does not still opens the file. Never `File.cs:124` as the target: outside a terminal
  that names a file that does not exist.
- Two to six bullets for a step with new or changed code; seven means it is two. A step that is
  on the route but unchanged gets one.
- No paragraphs inside an entry. A wall of prose under a number reads as mush; that is what the
  bullets are for.

## What belongs in the sub-bullets

A bullet explains **mechanics**. It does not restate the method's name in other words.
"`PublishMomentUtc` computes the publish moment" is an empty line — it repeats the identifier.
Write what the name does not already say.

Always break down:

1. **Library calls.** `RuleFor`, `Must`, `WithMessage`, `encodeDefaults`, `[Flags]`,
   `expectSuccess` — the reader is not obliged to remember what each one does. One line per call:
   what it does and what follows from it.
2. **Language constructs the logic rests on.** Pattern matching, a lambda passed as an expression
   tree rather than a function, a property's default value. If the behaviour depends on it being
   *that* construct, say so.
3. **A choice between two obvious options.** Where the code could have gone the simpler way and
   did not — name both and the reason. `ToUniversalTime` rather than `SpecifyKind`. The undo
   record written before the move, not after.
4. **What breaks without this line.** That is why the line is there, and it is what the reviewer
   is checking.

Do not break down: a plain assignment, a getter call, the order of arguments.

**No vague verbs.** "Specifies the property", "handles the request", "is responsible for", "takes
care of" — each of these leaves the reader asking *meaning what?*. Say what concretely happens and
where it shows up: not "`RuleFor` specifies the property" but "the checks after it apply to the
`PublishAt` field; the library reads that field's name out of the lambda and names it in the 400
body, so the client sees which field is wrong". If a term of art is unavoidable —
an expression tree — explain it in the same line, in plain words.

**Name what you mean, in every bullet.** "The field", "this value", "it" work only when the same
bullet has already said which one. Each bullet is read on its own — the reader arrived at it from a
link — so name the field, variable or case again: not "so a request without the field passes" but
"so a request without `publish_at` — the writer left the time on 'now' — passes".

## Footnotes for everything the project did not write

The reader knows their own code. What they may not know is the platform under it: a validation
library's `RuleFor`, a coroutine operator, a serializer setting, a database column type. Every such
name gets a footnote.

**What gets one:** any name or construct whose definition is not in the project — the language
itself, its standard library, a framework, a third-party library, a database, a format standard.
The test is simple: if searching the project will not find where it is defined, it gets a note.
The project's own names never do; the states below cover them.

**The marker** is a Unicode superscript number directly after the name, at its first mention, and
it is a link to that note's own file: `` `RuleFor`[¹⁶](post-scheduling/16-RuleFor.md) ``. Every note
is written to a small file of its own, in a folder named after the trace — `.claude/traces/
post-scheduling/16-RuleFor.md` — holding a heading with the number and the name, and the note.
A click on the marker opens the note in its own tab, beside the trace. No link back to the trace:
the Claude app's preview reopens a Markdown file from the top, and with a `#L` anchor as source text
rather than the rendered page. Nor are the notes put under each step instead — a block of
definitions after every entry buries the route it explains.
The trace itself carries no list of notes at the end — the files are the notes. Not an in-page
anchor: the Claude app's preview follows links to files but
does not jump to `#id` anchors or `[^12]` footnotes, and shows no link titles on hover. Rewriting a
trace rewrites its note folder — clear it first, the numbering has moved. Number in order of first
appearance. A later mention far from the first may carry the same number again, so the reader does
not have to scroll back to find it.

**Each note** — the text of its file, under a heading with its number and name in the reader's
language — answers two questions, in this order:

1. **What it is.** Its kind — method, class, attribute, setting, type, operator, language feature,
   column type, standard — and where it comes from, with that source's purpose in a few words: "a
   method of FluentValidation, a library for validating objects against rules written as code"; "an attribute from ASP.NET
   Core, Microsoft's web framework"; "a function from the Kotlin standard library". The kind tells
   the reader what sort of thing they are looking at; the source and its purpose tell them where it
   lives and why it exists.
2. **What it does** — one plain sentence, in general terms.

**The purpose is the source's own, not this project's.** "FluentValidation, a library for checking
incoming request data" describes how one backend happens to wire it, not what it is — the library
validates any object, wherever it came from. Write the general purpose in the note; what this
project does with it belongs in the bullet. A reader who meets the library again elsewhere must not
come away with a wrong idea of it.

**Every note stands on its own.** The reader arrives at one note from one marker and will not go
hunting for a second, so the source's purpose is repeated in every note that cites it — a few words,
not a paragraph. Explaining a library once, inside the note for some other name, hides it from
everyone who lands elsewhere.

**Name the library in the body, too.** At the step built on it, say so — "a FluentValidation
validator class", "the `DropIndex` of Entity Framework Core migrations" — and give the library its
own note there. A reader searching the text for a library they keep seeing in the notes should find
it.

A note says what the thing **is**; the bullet says **how this code uses it**. Do not repeat the
bullet in the note, and do not make the bullet define the thing — with the note in place, the bullet
can stay on the mechanics of this request.

## New in this branch comes first

Every name this branch introduced gets its own line: function, variable, field, constant,
parameter. Those are exactly the questions the trace exists to answer.

**Every bullet opens with its state, in bold — no bullet without one.** Without the mark the reader
goes hunting in the diff for something that is not there, or walks past a line whose meaning moved.
A bullet that only explains a consequence of the one above takes the state of the code it talks
about. The labels are translated with the rest of the trace.

- **New in this branch.** Break it down fully: what it does, why this way, what would happen without
  it.
- **Was there, unchanged.** One short bullet that **opens with why it is in the trace** — what in
  the new behaviour depends on it, which refusal happens there, or that the route starts here — and
  then only as much mechanism as backs that up: "Mentioned because the reminder at the chosen time
  depends on it: its list has no upper bound, so a post a month out is in it too, and the server
  needs nothing new." That is the answer to "why are we even in this file", and the reader should
  not have to work it out.
- **Was there, changed.** Say what the diff did to it — a parameter added, a return type narrowed,
  a body rewritten — **and why**, in the same bullet: "non-nullable now, because the only path that
  returned null was the unique-violation catch, and the index it caught is gone".
- **Was there, now means something else.** The dangerous one, and the strongest candidate for a
  line. The diff is tiny or absent while the meaning has moved, and that is what review misses most
  often. Say it plainly: what it meant before, what it means now, and what changed for everyone
  reading that name.
- **Removed in this branch.** What went, and why nothing needs it any more — the mechanism, in the
  same bullet: "the import served only the catch", not "went with the catch".

Examples of the fourth kind: a column that kept its name but now records the day work is *placed*
on rather than the day it was handed out; a guard call that is byte-identical but now asks about a
different day; a wire field that survives but always answers zero, kept only for app versions
already in people's hands.

**A change without its reason is half a bullet.** The reviewer reads the diff to see *what*
changed; the trace is there for *why*. Every bullet marked changed, removed or now meaning something
else says why in that same bullet — never in the next one, never implied by "along with", "went with
it", "together with". Those name a relation and hide its mechanism, exactly like a vague verb: say
the link itself — "only the catch used this import", "the catch returned null, and it is gone, so
the method never returns null". A reason the reader has to assemble from two bullets is a question
they will send you.

**One decision, one bullet.** When several edits in the diff follow from one decision — an index
dropped, so the catch that guarded it goes, so the method can no longer return null, so its import
and the caller's null check go too — they are one bullet, under the label of the edit that started
it, with the others named inside it as its consequences. Do not cut one decision into a bullet per
artefact because each piece could carry a label of its own: the reason then repeats in every piece,
and what changed in behaviour is said in none of them. Inside that bullet, lead with what changed in
behaviour — "`CreateAsync` now always returns the saved row" — then the mechanism. Edits with
separate reasons stay separate bullets.

**Unchanged code earns its place, or it goes.** Keep an unchanged place only when the new behaviour
depends on it (the client turns the new 400 into an error, the push service arms a reminder for the
new instant), when a refusal happens there, or when the route would break without it — and then in
one bullet, not a breakdown. Code that merely sits on the path and does not bear on the feature is
cut, and so are the unchanged details of a step you keep: how a sheet of unrelated warnings works, how
a range of pages is filled in, which flags an enum can carry. A reviewer reads every line you leave in
looking for what changed.

**Decide every state from git, never from memory** — not even for code you wrote an hour ago:
`git diff <base>...HEAD -- <file>` says whether the branch touched it, `git show <base>:<file>` shows
what was there. A file created in the branch and edited again in a later commit of the same branch is
still new. Before handing the file over, go through every bullet: it starts with a bold label, and
if the label is changed, removed or now meaning something else, the same bullet says why; and no
single decision is spread over several bullets.

## Start with the human's action, not the endpoint

The first entry is what the person tapped and which component caught it. Then the client: where
the request body is assembled and what ends up in it. Only then the network call.

The client half is half the route. Without it there is no telling where a field in the body came
from, or why it is absent in the other case.

If the client is not yours, or there is none, start at the endpoint and say so in one line.

## Always flag

**Every refusal point and its status code.** Where the request can fail to complete and what the
caller gets: `→ 400`, `→ 409`. It is the first thing the reader looks for.

**Whatever runs before the controller.** Validators, filters and middleware execute before the
controller's first line, and they are easy to miss precisely because nothing in the project's code
calls them. Say it outright, and **name who does run it**: "no code in the project calls it — the
framework runs it when the request arrives, before the controller". Never a figure of speech such
as "not called by hand" — in translation that reads as slang and still does not say who calls it.

**Side effects.** Signals, pushes, log lines, cache invalidations. They do not shape the response,
but they are what people go looking for later when something "did not refresh".

## Second part — life after the response

Many features do half their work after the request has finished: deferred visibility, a reminder,
a recalculation triggered by an event, a display on another screen.

Those go in a separate block after the line "After the request:", and **the numbering continues**
unbroken. The order there is not chronological but by place — the reader needs to know which other
files touch this entity.

## Line numbers

Put the line on **the thing the entry is about**, not on the top of the file: the method
declaration, the assignment, the guard call. If one entry covers several places, the links belong
in the bullets they describe, not piled into the heading.

Take the numbers from `grep -n` against the live file, never from the diff — a diff has its own
numbering and it will not match what the reader sees in the editor.

**Always put the symbol's name next to the line.** Numbers drift with the first edit made higher
up the file; names do not. The pair "name plus line" still works once the number has gone stale:
the name finds it, the number lands on it directly.

Two repositories in one trace: the file lives in one of them, and links into the other climb out of
it — `../../../sibling-repo/...` from `.claude/traces/`. Before handing the file over, resolve every
link against the file's folder and confirm the target exists and the line lands on the named
symbol. Otherwise half the links will not open.

## Verify against the code, not from memory

Open every method and read it. The order of checks inside a method is part of the answer — it is
how the reader learns what is validated before what.

This applies to code you wrote yourself an hour ago. Reconstructing a method's order from the
memory of writing it is how traces end up wrong.

Before handing it over, walk your own list and confirm that every named method exists and is
called from where you said.

## What not to do

**Do not draw it.** A sequence diagram of the same trace holds less, takes longer to read, and
cannot be pasted into a ticket. Draw one only if asked separately.

**No preamble, no summary** — in the file or in the chat. Not "let us walk through this step by step", not "so the request
crosses N layers". First entry, list, done.

**Do not pull in files that are not on the path.** Neighbouring edits, documentation, tests are
not steps.

## Length

As many entries as there are places on the path. Usually ten to twenty. More than thirty means
things that are not on the path have crept in.

## Going deeper into one entry

The companion skill `explain-line` picks up where the trace stops: `/explain-line 7` drills into
step 7, `/explain-line 7.4` into the fourth bullet of step 7, one question at a time. That only works
if the numbers mean what the reader sees — keep entries numbered without gaps and bullets dashed, one
fact each, so a count of bullets lands on the one they mean.

---

# Worked example

A constructed example, not a real repository: a writer schedules a post and picks when it goes
live. The client is Kotlin Multiplatform, the backend ASP.NET. The paths and line numbers are
invented — the shape is what to copy, not the coordinates. The links are written as they would be
in `.claude/traces/post-scheduling.md`, hence the `../../`, and the footnote markers point at the
note files in `.claude/traces/post-scheduling/`. The stack is incidental too: the skill
works with any language, and on another stack the notes explain that stack's libraries instead.

Match its **density and register**: one fact per line, no preamble, no hedging. The library calls
explained in it are real, and those explanations transfer as they are.

Two caveats.

**The set of layers is not a template.** This one has sixteen entries with a validator, a quota
check and a migration because that is how this request is built. Another may have no validator at
all, and no second part. Copying the shape means inventing steps.

**The example is in English because this file is.** A real trace follows the reader's language, notes included.

---

Path of the request when a post is published with a scheduled time.

## 1. Client, `ComposeScreenComponent.onPublishClicked()` → `performPublish()`

- **Was there, unchanged.** Where the route starts: the tap reaches `onPublishClicked()` at [ComposeScreenComponent.kt:214](../../app/src/commonMain/kotlin/com/example/compose/ComposeScreenComponent.kt#L214), which hands the draft to `performPublish()` at [:266](../../app/src/commonMain/kotlin/com/example/compose/ComposeScreenComponent.kt#L266) for the send.
- **Was there, unchanged.** Mentioned because the new "when" field is outside `performPublish()`'s check: the publish button never depends on it.

## 2. Same component, the clock and the timer

- **New in this branch.** `clock: Clock = Clock.System`[¹](post-scheduling/01-Clock.md), [:88](../../app/src/commonMain/kotlin/com/example/compose/ComposeScreenComponent.kt#L88). A constructor parameter with a default, so existing call sites still compile and a test can hand in its own clock.
- **New in this branch.** `scheduledMoments`, [:95](../../app/src/commonMain/kotlin/com/example/compose/ComposeScreenComponent.kt#L95) — a mirror of the chosen instants, because the state holder is not a kotlinx.coroutines[²](post-scheduling/02-kotlinx.coroutines.md) `Flow`[³](post-scheduling/03-Flow.md).
- **New in this branch.** `expireScheduleWhenItArrives()`, [:130](../../app/src/commonMain/kotlin/com/example/compose/ComposeScreenComponent.kt#L130): one sleep armed at the chosen instant, re-armed through `flatMapLatest`[⁴](post-scheduling/04-flatMapLatest.md) whenever the writer picks a different one. A fixed ticker would be late by up to its own interval.
- **New in this branch.** `dropMomentsAlreadyGoneBy()`, [:151](../../app/src/commonMain/kotlin/com/example/compose/ComposeScreenComponent.kt#L151). Called from the timer and from the lifecycle's resume callback — a timer does not run while the app is backgrounded.

## 3. Same component, handling `PublishAtChanged`

- **New in this branch.** The `PublishAtChanged` event, [ComposeScreenEvent.kt:34](../../app/src/commonMain/kotlin/com/example/compose/ComposeScreenEvent.kt#L34).
- **New in this branch.** Handler at [ComposeScreenComponent.kt:198](../../app/src/commonMain/kotlin/com/example/compose/ComposeScreenComponent.kt#L198): `takeIf { it != null && it > clock.now() }`[⁵](post-scheduling/05-takeIf.md) — a past instant is never stored at all, or the field would show a time in the past for a frame.
- **New in this branch.** `PostDraft.publishAt`, [PostDraft.kt:22](../../app/src/commonMain/kotlin/com/example/posts/PostDraft.kt#L22); `null` means "as soon as it is sent".
- **New in this branch.** `SavedDraft.publishAtEpochMillis`, [:402](../../app/src/commonMain/kotlin/com/example/compose/ComposeScreenComponent.kt#L402) — the choice survives process death, and a stale one is corrected on the next resume.

## 4. Client, `PostsRepositoryImpl.submitDraft()`

- **Was there, changed.** One argument added, `publishAt`, so the writer's chosen time reaches the request body: [PostsRepositoryImpl.kt:141](../../app/src/commonMain/kotlin/com/example/posts/data/PostsRepositoryImpl.kt#L141).
- **New in this branch.** `draft.publishAt?.toString()` — `kotlin.time.Instant.toString()`[⁶](post-scheduling/06-kotlin.time.Instant.md) is ISO-8601[⁷](post-scheduling/07-ISO-8601.md) in UTC with a trailing `Z`, exactly the shape the server demands.
- **New in this branch.** The field `publishAt: String? = null`, [CreatePostRequest.kt:18](../../app/src/commonMain/kotlin/com/example/posts/data/dto/CreatePostRequest.kt#L18). The default is load-bearing: with `encodeDefaults = false`[⁸](post-scheduling/08-encodeDefaults.md) in kotlinx.serialization[⁹](post-scheduling/09-kotlinx.serialization.md), a property equal to its default is never handed to the serializer, so the key is absent rather than null.
- **New in this branch.** Without `= null` the body would carry `"publish_at": null`, which is a different message even where a server happens to tolerate it.

## 5. `POST /api/posts`

- **Was there, unchanged.** Mentioned because this is how the new 400 reaches the writer: Ktor[¹⁰](post-scheduling/10-Ktor.md) sends the request at [PostsApiService.kt:44](../../app/src/commonMain/kotlin/com/example/posts/data/PostsApiService.kt#L44), and `expectSuccess = true`[¹¹](post-scheduling/11-expectSuccess.md) turns any non-2xx answer, the new 400 included, into an exception the app shows as an error.

## 6. Validator `CreatePostRequestValidator` → 400

- **New in this branch.** The file, a FluentValidation[¹²](post-scheduling/12-FluentValidation.md) validator class; the rule sits at [CreatePostRequestValidator.cs:14](../../Application/Validators/CreatePostRequestValidator.cs#L14).
- **Was there, unchanged.** No code in the project calls it — ASP.NET Core[¹³](post-scheduling/13-ASP.NET-Core.md) does: `AddValidatorsFromAssemblyContaining`[¹⁴](post-scheduling/14-AddValidatorsFromAssemblyContaining.md) registers it by scanning the assembly at startup, and `AddFluentValidationAutoValidation`[¹⁵](post-scheduling/15-AddFluentValidationAutoValidation.md) makes the framework run it on each request, before the controller's first line.
- **New in this branch.** `RuleFor(x => x.PublishAt)`[¹⁶](post-scheduling/16-RuleFor.md) — the checks chained after it apply to the `PublishAt` field. The compiler hands the library not the lambda's code but a description of it ("take `PublishAt` from `x`") — an expression tree[¹⁷](post-scheduling/17-Expression-tree.md) — so the library reads the field's name without running anything and names it in the 400 body, telling the client which field is wrong.
- **New in this branch.** `.Must(...)`[¹⁸](post-scheduling/18-Must.md) is the predicate itself, `true` meaning valid. `is not { Kind: DateTimeKind.Unspecified }`[¹⁹](post-scheduling/19-Property-pattern.md) is a property pattern; it does not match on `null`, `is not` yields `true`, so a request without `publish_at` — the writer left the time on "now" — passes.
- **New in this branch.** `.WithMessage(...)`[²⁰](post-scheduling/20-WithMessage.md) states both the mistake and the accepted form instead of a generic "invalid request".
- **Was there, unchanged.** On failure ModelState[²¹](post-scheduling/21-ModelState.md) is invalid and `[ApiController]`[²²](post-scheduling/22-note.md) returns the 400 with `ValidationProblemDetails`[²³](post-scheduling/23-ValidationProblemDetails.md) on its own.

## 7. Controller `PostsController.Create`

- **Removed in this branch.** `PublishOutcome` and its result wrapper, so `Create` now simply returns the post: [PostsController.cs:52](../../Presentation/Controllers/PostsController.cs#L52). The outcome told the controller which answer to give — 409 "already posted today" for the daily cap, 500 "not configured" for a missing `PublishDelayHours` setting. The branch drops both the cap and the setting, which would leave a single outcome, "created"; so the wrapper and both error arms go, and the action shrinks to a single expression.
- **Was there, now means something else.** `GetPostingAllowance` at [:31](../../Presentation/Controllers/PostsController.cs#L31): synchronous, calls no service, always answers "allowed" — kept only for app versions already shipped.

## 8. Service `PostService.CreateAsync`, opening lines

- **Was there, changed.** Return type changed to the response DTO, because the result wrapper only existed to carry an outcome for the controller, and the outcomes are gone: [PostService.cs:58](../../Application/Services/PostService.cs#L58).
- **Was there, unchanged.** Mentioned because the new day is read in this zone: the author lookup and the timezone resolution take it from the profile, else the account's country, else the platform default.
- **Removed in this branch.** Reading the `PublishDelayHours` setting and the settings dependency — the writer now picks the moment, so there is no fixed "goes live in N hours" delay left to read.

## 9. Same method, `PublishMomentUtc`

- **New in this branch.** The method, [:121](../../Application/Services/PostService.cs#L121); the constant `PastInstantWorthReporting` at [:33](../../Application/Services/PostService.cs#L33).
- **New in this branch.** `request.PublishAt is not { } requestedAt`[¹⁹](post-scheduling/19-Property-pattern.md) — a null check and a capture in one expression; no field, return "now".
- **New in this branch.** `ToUniversalTime()`[²⁴](post-scheduling/24-ToUniversalTime.md) rather than `SpecifyKind`[²⁵](post-scheduling/25-SpecifyKind.md) — an offset such as `+05:00` arrives as `Kind = Local`[²⁶](post-scheduling/26-DateTimeKind.md) and names a different instant than its digits read; the column is a PostgreSQL[²⁷](post-scheduling/27-PostgreSQL.md) `timestamptz`[²⁸](post-scheduling/28-timestamptz.md) and rejects a non-UTC kind outright.
- **New in this branch.** A past instant is replaced by "now": the writer meant "as early as possible", and refusing would only cost them the post. `Log.Warning`[²⁹](post-scheduling/29-Log.Warning.md) from Serilog[³⁰](post-scheduling/30-Serilog.md) fires only past a five-minute drift, because a clock a little behind is ordinary and a clock an hour behind is a client bug.

## 10. Same method, the quota check → 409

- **Was there, unchanged.** The calls themselves, [:71–74](../../Application/Services/PostService.cs#L71).
- **Was there, now means something else.** The second check: it used to ask about the day a few hours of delay landed on, and now asks about the day the writer named, which may be weeks out.
- **Was there, unchanged.** The quota window is loaded once with no upper bound, so a post aimed far into the future is still measured against the right day.
- **Was there, unchanged.** Both throw; an exception filter turns them into a 409 carrying a code and the day that is already full.

## 11. Same method, building the row

- **Was there, now means something else.** `PublishOnDay`, [:88](../../Application/Services/PostService.cs#L88) — it used to be the day the post was written, and is now the day it goes live, read in the author's zone.
- **Removed in this branch.** The test-mode branch that nulled it — the null existed only to dodge a unique index that this branch drops.
- **Was there, now means something else.** `CreatedAt` and `PublishAt` can now be weeks apart, which was impossible before.

## 12. Repository `PostRepository.CreateAsync`

- **Removed in this branch.** The unique-violation catch from Npgsql[³¹](post-scheduling/31-Npgsql.md), so `CreateAsync` now always returns the saved row. Before, a second post for the same day hit the daily unique index, and the catch turned that into a null. The branch drops the index, and the only uniqueness left is the primary key on a freshly generated id, which cannot collide. Three more edits follow from this one: the return type narrows from nullable to non-nullable ([PostRepository.cs:19](../../Infrastructure/Repositories/PostRepository.cs#L19)), the service's null check goes, and so does the Npgsql import, which served only the exception types in the catch.

## 13. Service `PostService.CreateAsync`, the change notification, then the 201

- **Was there, unchanged.** The notification, [PostService.cs:103](../../Application/Services/PostService.cs#L103); it matters more now — the instant can be far off and subscribers' devices must learn about it in advance.

After the request:

## 14. `PostRepository.GetVisibleForFeedAsync`

- **Was there, unchanged.** Mentioned because it alone hides a post scheduled for the day after tomorrow: [PostRepository.cs:28](../../Infrastructure/Repositories/PostRepository.cs#L28).

## 15. `QuietHoursRescheduler` and `SetPublishAtAsync`

- **Was there, changed.** `SetPublishAtAsync` gained a third parameter and now writes three columns: [PostRepository.cs:96](../../Infrastructure/Repositories/PostRepository.cs#L96) — the instant, the day it lands on, and a reset of the "already announced" flag. The day is the new parameter: when quiet hours push the instant, `PublishOnDay` — now the day the post goes live — has to move with it, or the author's list would keep showing the old day.
- **New in this branch.** The parameter is required rather than defaulted, so the compiler points at the one call site; a default would let the instant and the day drift apart in silence.
- **Was there, changed.** The author's zone is threaded down to the write, because only the caller knows whose calendar the day is read in.

## 16. Migration `DropDailyPostUniqueIndex`

- **New in this branch.** [DropDailyPostUniqueIndex.cs:24](../../Infrastructure/Migrations/DropDailyPostUniqueIndex.cs#L24).
- **New in this branch.** `DROP INDEX IF EXISTS`[³²](post-scheduling/32-DROP-INDEX-IF-EXISTS.md) as raw SQL rather than the `DropIndex`[³³](post-scheduling/33-DropIndex.md) of Entity Framework Core[³⁴](post-scheduling/34-Entity-Framework-Core.md) migrations — this database was adopted at a squashed baseline, so the migration chain does not prove the index is there.
- **New in this branch.** `Down()`[³⁵](post-scheduling/35-Down.md) recreates a unique index and will fail once an author has two posts on one day; the comment says so outright rather than letting a rollback discover it.

---

The trace ends there. Below are the texts of its note files, one per file in
`.claude/traces/post-scheduling/` — gathered here only to show them; the trace file holds none of
them.

## Notes

1. `Clock` — an interface from the Kotlin standard library (`kotlin.time`). Its one method, `now()`, returns the current instant; `Clock.System` reads the real clock, and code that takes a `Clock` as a parameter can be handed a fake one in a test.
2. kotlinx.coroutines — a JetBrains library for asynchronous code in Kotlin: coroutines, and the `Flow` streams built on them.
3. `Flow` — a type from kotlinx.coroutines, the library for asynchronous code in Kotlin. A stream of values delivered over time to whoever collects it.
4. `flatMapLatest` — an operator from kotlinx.coroutines, the library for asynchronous code in Kotlin. For each new value of a flow, starts new work and cancels the work started for the previous value.
5. `takeIf` — a function from the Kotlin standard library. Returns the value itself when the condition holds, otherwise `null`.
6. `kotlin.time.Instant` — a type from the Kotlin standard library. A point on the global timeline with no time zone; `toString()` writes it as an ISO-8601 string in UTC.
7. ISO-8601 — an international standard for writing dates and times as text, e.g. `2026-09-17T20:00:00Z`; the trailing `Z` means UTC.
8. `encodeDefaults` — a setting of kotlinx.serialization, the library that turns Kotlin objects into JSON and back. When `false`, a property whose value equals its default is left out of the JSON.
9. kotlinx.serialization — a JetBrains library that turns Kotlin objects into JSON and back.
10. Ktor — a JetBrains library for HTTP in Kotlin; on the client it is what sends the request.
11. `expectSuccess` — a setting of the Ktor HTTP client. When `true`, a response with a non-2xx status throws an exception instead of being returned as an ordinary response.
12. FluentValidation — a third-party .NET library for validating objects: the rules are written as code in a validator class rather than as attributes on the properties, and the object can come from anywhere.
13. ASP.NET Core — Microsoft's web framework for .NET: it receives HTTP requests, turns their bodies into objects and calls controller methods.
14. `AddValidatorsFromAssemblyContaining` — a method of FluentValidation, the library for validating objects against rules written as code. Finds every validator class in the assembly that holds the given type and registers them in the dependency-injection container.
15. `AddFluentValidationAutoValidation` — a method of FluentValidation's integration with ASP.NET Core, the web framework. Makes the framework run the matching validator on each incoming request body, before the controller method.
16. `RuleFor` — a method of FluentValidation, the library for validating objects against rules written as code. Starts a rule for one property; every check chained after it applies to that property.
17. Expression tree — a C# language feature. When a lambda goes to a parameter of type `Expression<...>`, the compiler passes a description of its code instead of compiled code, so the receiver can read, for instance, which property it touches.
18. `Must` — a method of FluentValidation, the library for validating objects against rules written as code. Adds a custom check to a rule: a function that returns `true` when the value is valid.
19. Property pattern (`is { ... }`, `is not { ... }`) — a C# language feature, part of pattern matching. Tests that a value is non-null and that its listed properties match; the empty form `is { } x` only checks for non-null and puts the value in `x`.
20. `WithMessage` — a method of FluentValidation, the library for validating objects against rules written as code. Sets the error text returned when the preceding check fails.
21. ModelState — an object from ASP.NET Core, Microsoft's web framework. Records whether the current request's data was read and validated successfully, with the errors if not.
22. `[ApiController]` — an attribute from ASP.NET Core, Microsoft's web framework. Among other things, makes the framework answer 400 itself when ModelState has errors, without calling the controller method.
23. `ValidationProblemDetails` — a class from ASP.NET Core, Microsoft's web framework. The standard JSON shape of a validation error: a title, a status, and an `errors` object keyed by field name.
24. `ToUniversalTime` — a method from the .NET standard library. Converts a `DateTime` to UTC according to its `Kind`: a `Local` value is shifted by its offset, a `Utc` value is returned as is.
25. `SpecifyKind` — a method from the .NET standard library. Returns the same date and clock digits under a different `Kind` label, shifting nothing.
26. `DateTimeKind` — an enum from the .NET standard library. Says whether a `DateTime` is in UTC (`Utc`), in the machine's zone (`Local`), or has no zone at all (`Unspecified`).
27. PostgreSQL — the relational database the backend stores its data in.
28. `timestamptz` — a column type in PostgreSQL, the database. Holds an absolute instant ("timestamp with time zone"); the .NET driver accepts only UTC `DateTime` values for it.
29. `Log.Warning` — a method of Serilog, a logging library for .NET. Writes a warning-level entry through the application-wide logger.
30. Serilog — a logging library for .NET.
31. Npgsql — the .NET driver for PostgreSQL, the library through which .NET code talks to the database; `PostgresErrorCodes.UniqueViolation` is its code for an insert that breaks a unique index.
32. `DROP INDEX IF EXISTS` — an SQL command in PostgreSQL, the database. Removes an index, and does nothing instead of failing when the index is not there.
33. `DropIndex` — a method of Entity Framework Core migrations; Entity Framework Core is .NET's library for working with a database through C# objects. Emits a plain `DROP INDEX`, which fails when the index is missing.
34. Entity Framework Core — .NET's library for working with a database through C# objects; its migrations describe each schema change as code.
35. `Down()` — a method of Entity Framework Core migrations, .NET's library for working with a database. Undoes the migration when the database is rolled back to an earlier version.
