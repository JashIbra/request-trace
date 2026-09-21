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
- **File and line are required** — as a link, `[File.cs:124](path/from/root/File.cs:124)`. In a
  terminal that link opens the file at that line, which is the whole point: the reader compares
  against the code instead of scrolling for it.
- Two to six bullets. One means the step is not a step; seven means it is two.
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
`PublishAt` field; the library reads that field's name out of the lambda and puts it in the 400
body as `publish_at`, so the client sees which field is wrong". If a term of art is unavoidable —
an expression tree — explain it in the same line, in plain words.

## New in this branch comes first

Every name this branch introduced gets its own line: function, variable, field, constant,
parameter. Those are exactly the questions the trace exists to answer.

And every name carries one of three states. Without the mark the reader goes hunting in the diff
for something that is not there — or worse, walks past a line whose meaning moved.

**1. Introduced in this branch.** Break it down fully: what it does, why this way, what would
happen without it.

**2. Was there, unchanged.** Explain only as far as the new code needs. A full tour of an
untouched file is not part of a trace. But do say **how the new code uses it** — that is the
answer to "why are we even in this file".

**3. Was there, but now means something else.** The dangerous one, and the strongest candidate for
a line. The diff is tiny or absent while the meaning has moved, and that is what review misses
most often. Say it plainly: what it meant before, what it means now, and what changed for everyone
reading that name.

Examples of the third kind: a column that kept its name but now records the day work is *placed*
on rather than the day it was handed out; a guard call that is byte-identical but now asks about a
different day; a wire field that survives but always answers zero, kept only for app versions
already in people's hands.

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

Two repositories in one trace: paths relative to the root of whichever one you are working in,
and a relative path (`../sibling-repo/...`) for the other. Otherwise half the links will not open.

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

**No preamble, no summary.** Not "let us walk through this step by step", not "so the request
crosses N layers". First entry, list, done.

**Do not pull in files that are not on the path.** Neighbouring edits, documentation, tests are
not steps.

## Length

As many entries as there are places on the path. Usually ten to twenty. More than thirty means
things that are not on the path have crept in.

---

# Worked example

A constructed example, not a real repository: a writer schedules a post and picks when it goes
live. The client is Kotlin Multiplatform, the backend ASP.NET. The paths and line numbers are
invented — the shape is what to copy, not the coordinates.

Match its **density and register**: one fact per line, no preamble, no hedging. The library calls
explained in it are real, and those explanations transfer as they are.

Two caveats.

**The set of layers is not a template.** This one has sixteen entries with a validator, a quota
check and a migration because that is how this request is built. Another may have no validator at
all, and no second part. Copying the shape means inventing steps.

**The example is in English because this file is.** A real trace follows the reader's language.

---

Path of the request when a post is published with a scheduled time.

## 1. Client, `ComposeScreenComponent.onPublishClicked()` → `performPublish()`

- Was there before the branch, unchanged: [ComposeScreenComponent.kt:214](app/src/commonMain/kotlin/com/example/compose/ComposeScreenComponent.kt:214) and [:266](app/src/commonMain/kotlin/com/example/compose/ComposeScreenComponent.kt:266).
- The tap arrives as a `Publish` event through `onEvent`; `onPublishClicked()` first checks the draft for empty required blocks and raises a sheet when it finds any.
- `performPublish()` re-validates: a network wait sits between the tap and the send, and the editor stays live throughout.
- The "when" field is deliberately outside that check — the publish button never depends on it.

## 2. Same component, the clock and the timer

- `clock: Clock = Clock.System` introduced in this branch, [:88](app/src/commonMain/kotlin/com/example/compose/ComposeScreenComponent.kt:88). A constructor parameter with a default, so existing call sites still compile and a test can hand in its own clock.
- `scheduledMoments` introduced in this branch, [:95](app/src/commonMain/kotlin/com/example/compose/ComposeScreenComponent.kt:95) — a mirror of the chosen instants, because the state holder is not a `Flow`.
- `expireScheduleWhenItArrives()` introduced in this branch, [:130](app/src/commonMain/kotlin/com/example/compose/ComposeScreenComponent.kt:130): one sleep armed at the chosen instant, re-armed through `flatMapLatest` whenever the writer picks a different one. A fixed ticker would be late by up to its own interval.
- `dropMomentsAlreadyGoneBy()` introduced in this branch, [:151](app/src/commonMain/kotlin/com/example/compose/ComposeScreenComponent.kt:151). Called from the timer and from the lifecycle's resume callback — a timer does not run while the app is backgrounded.

## 3. Same component, handling `PublishAtChanged`

- The event was introduced in this branch, [ComposeScreenEvent.kt:34](app/src/commonMain/kotlin/com/example/compose/ComposeScreenEvent.kt:34).
- Handler at [ComposeScreenComponent.kt:198](app/src/commonMain/kotlin/com/example/compose/ComposeScreenComponent.kt:198): `takeIf { it != null && it > clock.now() }` — a past instant is never stored at all, or the field would show a time in the past for a frame.
- `PostDraft.publishAt` introduced in this branch, [PostDraft.kt:22](app/src/commonMain/kotlin/com/example/posts/PostDraft.kt:22); `null` means "as soon as it is sent".
- `SavedDraft.publishAtEpochMillis` introduced in this branch, [:402](app/src/commonMain/kotlin/com/example/compose/ComposeScreenComponent.kt:402) — the choice survives process death, and a stale one is corrected on the next resume.

## 4. Client, `PostsRepositoryImpl.submitDraft()`

- Was there before the branch; one argument added: [PostsRepositoryImpl.kt:141](app/src/commonMain/kotlin/com/example/posts/data/PostsRepositoryImpl.kt:141).
- `draft.publishAt?.toString()` — `kotlin.time.Instant.toString()` is ISO-8601 in UTC with a trailing `Z`, exactly the shape the server demands.
- The field `publishAt: String? = null` introduced in this branch, [CreatePostRequest.kt:18](app/src/commonMain/kotlin/com/example/posts/data/dto/CreatePostRequest.kt:18). The default is load-bearing: with `encodeDefaults = false`, a property equal to its default is never handed to the serializer, so the key is absent rather than null.
- Without `= null` the body would carry `"publish_at": null`, which is a different message even where a server happens to tolerate it.

## 5. `POST /api/posts`

- Was there before the branch: [PostsApiService.kt:44](app/src/commonMain/kotlin/com/example/posts/data/PostsApiService.kt:44), Ktor with the app-wide `Json` instance.
- `expectSuccess = true` — any non-2xx response becomes an exception and is mapped to an app error, including the new 400.

## 6. Validator `CreatePostRequestValidator` → 400

- File introduced in this branch; the rule sits at [CreatePostRequestValidator.cs:14](Application/Validators/CreatePostRequestValidator.cs:14).
- No code in the project calls it — ASP.NET does: `AddValidatorsFromAssemblyContaining` registers it by scanning the assembly at startup, and `AddFluentValidationAutoValidation` makes the framework run it on each request, before the controller's first line.
- `RuleFor(x => x.PublishAt)` — the checks chained after it apply to the `PublishAt` field. The compiler hands the library not the lambda's code but a description of it ("take `PublishAt` from `x`") — an expression tree — so the library reads the field's name without running anything, and writes it into the 400 body as `publish_at`, telling the client which field is wrong.
- `.Must(...)` is the predicate itself, `true` meaning valid. `is not { Kind: DateTimeKind.Unspecified }` is a property pattern; it does not match on `null`, `is not` yields `true`, so an absent field passes.
- `.WithMessage(...)` states both the mistake and the accepted form instead of a generic "invalid request".
- On failure ModelState is invalid and `[ApiController]` returns the 400 with `ValidationProblemDetails` on its own.

## 7. Controller `PostsController.Create`

- Was there before the branch, rewritten to a single expression: [PostsController.cs:52](Presentation/Controllers/PostsController.cs:52).
- `PublishOutcome` and its result wrapper were deleted outright — with the daily posting cap gone they carried one member.
- The 409 "already posted today" and the 500 "not configured" arms went with them.
- `GetPostingAllowance` at [:31](Presentation/Controllers/PostsController.cs:31) now means something else: synchronous, calls no service, always answers "allowed" — kept only for app versions already shipped.

## 8. Service `PostService.CreateAsync`, opening lines

- Return type changed to the response DTO: [PostService.cs:58](Application/Services/PostService.cs:58).
- The author lookup and the timezone resolution were there before: zone from the profile, else the account's country, else the platform default.
- Reading the `PublishDelayHours` setting and the settings dependency were removed — the "goes live in N hours" delay no longer exists.

## 9. Same method, `PublishMomentUtc`

- Method introduced in this branch, [:121](Application/Services/PostService.cs:121); the constant `PastInstantWorthReporting` at [:33](Application/Services/PostService.cs:33).
- `request.PublishAt is not { } requestedAt` — a null check and a capture in one expression; no field, return "now".
- `ToUniversalTime()` rather than `SpecifyKind` — an offset such as `+05:00` arrives as `Kind = Local` and names a different instant than its digits read; the column is `timestamptz` and rejects a non-UTC kind outright.
- A past instant is replaced by "now": the writer meant "as early as possible", and refusing would only cost them the post. `Log.Warning` fires only past a five-minute drift, because a clock a little behind is ordinary and a clock an hour behind is a client bug.

## 10. Same method, the quota check → 409

- The calls themselves were there before the branch, [:71–74](Application/Services/PostService.cs:71).
- The second check now means something else: it used to ask about the day a few hours of delay landed on, and now asks about the day the writer named, which may be weeks out.
- The quota window is loaded once with no upper bound, so a post aimed far into the future is still measured against the right day.
- Both throw; an exception filter turns them into a 409 carrying a code and the day that is already full.

## 11. Same method, building the row

- `PublishOnDay` was there before the branch but now means something else: [:88](Application/Services/PostService.cs:88) — it used to be the day the post was written, and is now the day it goes live, read in the author's zone.
- The test-mode branch that nulled it is gone — the null existed only to dodge a unique index that this branch drops.
- `CreatedAt` and `PublishAt` can now be weeks apart, which was impossible before.

## 12. Repository `PostRepository.CreateAsync`

- Return type changed from nullable to non-nullable: [PostRepository.cs:19](Infrastructure/Repositories/PostRepository.cs:19).
- The unique-violation catch was removed: it guarded the daily cap, and the only uniqueness left is the primary key on a freshly generated id.
- The Npgsql import and the null branch in the service went with it.

## 13. Same method, the change notification, then the 201

- The notification was there before the branch, unchanged: [PostService.cs:103](Application/Services/PostService.cs:103), but it matters more now — the instant can be far off and subscribers' devices must learn about it in advance.
- The target enum is `[Flags]`, so one ping can name several parts of the client's copy at once instead of two pings that cancel each other's fetch.
- In the response the instant is stamped `Kind = Utc`, or JSON omits the trailing `Z` and the client's parse fails.

After the request:

## 14. `PostRepository.GetVisibleForFeedAsync`

- Was there before the branch, unchanged: [PostRepository.cs:28](Infrastructure/Repositories/PostRepository.cs:28), but it carries more now — it alone hides a post scheduled for the day after tomorrow.
- Filters on "not deleted" plus "publish at or before now"; a flag lifts the gate so the author's own drafts view can show what is still pending.

## 15. `QuietHoursRescheduler` and `SetPublishAtAsync`

- `SetPublishAtAsync` gained a third parameter and now writes three columns: [PostRepository.cs:96](Infrastructure/Repositories/PostRepository.cs:96) — the instant, the day it lands on, and a reset of the "already announced" flag.
- The parameter is required rather than defaulted, so the compiler points at the one call site; a default would let the instant and the day drift apart in silence.
- The author's zone is threaded down to the write, because only the caller knows whose calendar the day is read in.
- The undo record is written before the row moves: a record with the row still in place is harmless, while a moved row nobody recorded could never be put back.

## 16. Migration `DropDailyPostUniqueIndex`

- Introduced in this branch: [DropDailyPostUniqueIndex.cs:24](Infrastructure/Migrations/DropDailyPostUniqueIndex.cs:24).
- `DROP INDEX IF EXISTS` as raw SQL rather than the builder's `DropIndex` — this database was adopted at a squashed baseline, so the migration chain does not prove the index is there.
- `Down()` recreates a unique index and will fail once an author has two posts on one day; the comment says so outright rather than letting a rollback discover it.
