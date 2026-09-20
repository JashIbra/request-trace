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

1. **Named in the request** — "creating a global repetition", "sending a chat message". Use that.
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
- Write the network call as it is: `POST /api/GlobalRepetition`.
- **File and line are required** — as a link, `[File.cs:124](path/from/root/File.cs:124)`. In a
  terminal that link opens the file at that line, which is the whole point: the reader compares
  against the code instead of scrolling for it.
- Two to six bullets. One means the step is not a step; seven means it is two.
- No paragraphs inside an entry. A wall of prose under a number reads as mush; that is what the
  bullets are for.

## What belongs in the sub-bullets

A bullet explains **mechanics**. It does not restate the method's name in other words.
"`OpeningMomentUtc` computes the opening moment" is an empty line — it repeats the identifier.
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
controller's first line, and they are easy to miss precisely because nothing calls them
explicitly. Say it outright: "nothing calls it; the pipeline runs it".

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

A real trace across a Kotlin Multiplatform client and a .NET backend: a teacher assigns a "global
repetition" and picks when it opens for the student. Match its **density and register** — one fact
per line, no preamble, no hedging.

Two caveats.

**The set of layers is not a template.** This one has twenty-two entries with a validator, a leave
check and a migration because that is how this request is built. Another may have no validator at
all, and no second part. Copying the shape means inventing steps.

**It is frozen in time** and quotes code that has since moved on. It is a model of form, not a
description of a live system.

The example below is in English because this file is. A real trace follows the reader's language.

---

Path of the request when a global repetition is sent with a chosen time. Links open the file at
the line; the root is the backend, the client is reached through `../`.

## 1. Client, `AssignTaskScreenComponent.onSubmitClicked()` → `performAssign()`

- Was there before the branch, unchanged: [AssignTaskScreenComponent.kt:514](../TahfeezAIRoot/TahfeezAIKMP/tahfeezApp/src/commonMain/kotlin/com/app/tahfeez/tasks/presentation/assign/AssignTaskScreenComponent.kt:514) and [:592](../TahfeezAIRoot/TahfeezAIKMP/tahfeezApp/src/commonMain/kotlin/com/app/tahfeez/tasks/presentation/assign/AssignTaskScreenComponent.kt:592).
- The tap arrives as a `Submit` event through `onEvent`; `onSubmitClicked()` first asks how many unfinished tasks the student has and raises a sheet when there are any.
- `performAssign()` re-validates the cards: a network wait sits between the tap and the send, and the fields stay editable throughout.
- The "when" field is deliberately outside that check — the send button never depends on it.

## 2. Same component, the clock and the timer

- `clock: Clock = Clock.System` introduced in this branch, [:140](../TahfeezAIRoot/TahfeezAIKMP/tahfeezApp/src/commonMain/kotlin/com/app/tahfeez/tasks/presentation/assign/AssignTaskScreenComponent.kt:140). A third constructor parameter with a default, so existing call sites still compile and a test can hand in its own clock.
- `chosenOpeningMoments` introduced in this branch, [:159](../TahfeezAIRoot/TahfeezAIKMP/tahfeezApp/src/commonMain/kotlin/com/app/tahfeez/tasks/presentation/assign/AssignTaskScreenComponent.kt:159) — a mirror of the chosen moments, because Decompose's `Value` is not a `Flow`.
- `expireChosenMomentsWhenTheyArrive()` introduced in this branch, [:201](../TahfeezAIRoot/TahfeezAIKMP/tahfeezApp/src/commonMain/kotlin/com/app/tahfeez/tasks/presentation/assign/AssignTaskScreenComponent.kt:201): a `Value.subscribe(lifecycle, CREATE_DESTROY)` plus `flatMapLatest` onto a sleep that ends at the chosen moment.
- `openingTicks` was there before the branch, [OnlyOpened.kt:53](../TahfeezAIRoot/TahfeezAIKMP/tahfeezApp/src/commonMain/kotlin/com/app/tahfeez/core/util/OnlyOpened.kt:53), used here in a new way: it sleeps to the exact instant instead of ticking every minute.
- `dropMomentsAlreadyGoneBy()` introduced in this branch, [:231](../TahfeezAIRoot/TahfeezAIKMP/tahfeezApp/src/commonMain/kotlin/com/app/tahfeez/tasks/presentation/assign/AssignTaskScreenComponent.kt:231). Called from the timer and from `lifecycle.doOnResume` — a timer does not run in the background.

## 3. Same component, handling `AvailableAtChanged`

- The event was introduced in this branch, [AssignTaskScreenEvent.kt:48](../TahfeezAIRoot/TahfeezAIKMP/tahfeezApp/src/commonMain/kotlin/com/app/tahfeez/tasks/presentation/assign/AssignTaskScreenEvent.kt:48).
- Handler at [AssignTaskScreenComponent.kt:396](../TahfeezAIRoot/TahfeezAIKMP/tahfeezApp/src/commonMain/kotlin/com/app/tahfeez/tasks/presentation/assign/AssignTaskScreenComponent.kt:396): `takeIf { it != null && it > clock.now() }` — a past moment is never stored at all, or the card would show a time in the past for a frame.
- `AssignTaskDraft.availableAt` introduced in this branch, [AssignTaskDraft.kt:39](../TahfeezAIRoot/TahfeezAIKMP/tahfeezApp/src/commonMain/kotlin/com/app/tahfeez/tasks/domain/model/AssignTaskDraft.kt:39); `null` means "as soon as it is sent".
- `SavedAssignCard.availableAtEpochMillis` introduced in this branch, [:830](../TahfeezAIRoot/TahfeezAIKMP/tahfeezApp/src/commonMain/kotlin/com/app/tahfeez/tasks/presentation/assign/AssignTaskScreenComponent.kt:830) — the choice survives process death, and a stale one is corrected on the next resume.

## 4. Client, the "when" field on the card

- `AssignTaskWhenField` introduced in this branch, [AssignTaskWhenField.kt:49](../TahfeezAIRoot/TahfeezAIKMP/tahfeezApp/src/commonMain/kotlin/com/app/tahfeez/tasks/presentation/assign/ui/AssignTaskWhenField.kt:49); placed on the card at [AssignTaskCard.kt:352](../TahfeezAIRoot/TahfeezAIKMP/tahfeezApp/src/commonMain/kotlin/com/app/tahfeez/tasks/presentation/assign/ui/AssignTaskCard.kt:352), for the repetition kind only.
- Two dialogs in sequence: the calendar reuses the existing `AppDatePickerDialog`, while `AppTimePickerDialog` was introduced in this branch, [AppTimePickerDialog.kt:49](../TahfeezAIRoot/TahfeezAIKMP/tahfeezApp/src/commonMain/kotlin/com/app/tahfeez/auth/presentation/components/AppTimePickerDialog.kt:49).
- Past days are greyed out through `SelectableDates`; Material 3's time picker has no such hook, so a past hour resolves to "now" rather than a refusal.
- `assignTaskWhenText` introduced in this branch, [AssignTaskWhenFormatter.kt:31](../TahfeezAIRoot/TahfeezAIKMP/tahfeezApp/src/commonMain/kotlin/com/app/tahfeez/tasks/presentation/assign/ui/AssignTaskWhenFormatter.kt:31) — either "now" or "17 Sep, 20:00" in the device's zone.

## 5. Client, `TeacherTasksRepositoryImpl.submitDraft()`

- Was there before the branch; one argument added: [TeacherTasksRepositoryImpl.kt:539](../TahfeezAIRoot/TahfeezAIKMP/tahfeezApp/src/commonMain/kotlin/com/app/tahfeez/tasks/data/repository/TeacherTasksRepositoryImpl.kt:539).
- `draft.availableAt?.toString()` — `kotlin.time.Instant.toString()` is ISO-8601 in UTC with a trailing `Z`, exactly the shape the server demands.
- The field `availableAt: String? = null` introduced in this branch, [CreateGlobalRepeatRequest.kt:29](../TahfeezAIRoot/TahfeezAIKMP/tahfeezApp/src/commonMain/kotlin/com/app/tahfeez/repetition/data/network/dto/CreateGlobalRepeatRequest.kt:29). The default is load-bearing: with `encodeDefaults = false`, a property equal to its default is never handed to the serializer.
- Without `= null` the body would carry `"available_at": null`, which is a different message, even though this server happens to tolerate it.
- The memorization branch is untouched: a memorization lands on a study day and the teacher never names a moment.

## 6. `POST /api/GlobalRepetition`

- Was there before the branch: [RepetitionApiService.kt:96](../TahfeezAIRoot/TahfeezAIKMP/tahfeezApp/src/commonMain/kotlin/com/app/tahfeez/repetition/data/network/RepetitionApiService.kt:96), Ktor with the app-wide `JsonDefault`.
- `expectSuccess = true` — any non-2xx response becomes an exception and is mapped to an app error, including the new 400.

## 7. Validator `CreateGlobalRepeatTaskRequestValidator` → 400

- File introduced in this branch; the rule sits at [CreateGlobalRepeatTaskRequestValidator.cs:14](Application/Validators/CreateGlobalRepeatTaskRequestValidator.cs:14).
- Nothing calls it: `AddValidatorsFromAssemblyContaining` finds it by scanning the assembly, and `AddFluentValidationAutoValidation` runs it before the controller's first line.
- `RuleFor(x => x.AvailableAt)` names the property. The lambda is passed as an expression tree, so the library reads the property *name* out of it; that name becomes the key in the 400 body, snake-cased.
- `.Must(...)` is the predicate itself, `true` meaning valid. `is not { Kind: DateTimeKind.Unspecified }` is a property pattern; it does not match on `null`, `is not` yields `true`, so an absent field passes.
- `.WithMessage(...)` states both the mistake and the accepted form instead of a generic "invalid request".
- On failure ModelState is invalid and `[ApiController]` returns the 400 with `ValidationProblemDetails` on its own.

## 8. Controller `GlobalRepetitionController.Create`

- Was there before the branch, rewritten to a single expression: [GlobalRepetitionController.cs:65](Presentation/Controllers/GlobalRepetitionController.cs:65).
- `GlobalRepeatOutcome` and `GlobalRepeatCreateResult` were deleted outright — with the daily cap gone they carried one member.
- The 409 "already assigned today" and the 500 "not configured" arms went with them.
- `GetStudentProgress` at [:47](Presentation/Controllers/GlobalRepetitionController.cs:47) now means something else: synchronous, calls no service, always answers "available" — kept only for app versions already shipped.

## 9. Service `GlobalRepetitionService.CreateAsync`, opening lines

- Return type changed to the response DTO: [GlobalRepetitionService.cs:61](Application/Services/GlobalRepetitionService.cs:61).
- `GetByIdAsync` and `HelperTimeZone.ResolveTimezone` were there before: zone from the profile, else the country, else a default.
- Reading the `AccessGlobalRepeat` setting and the settings dependency were removed — the "opens in N hours" delay no longer exists.

## 10. Same method, `OpeningMomentUtc`

- Method introduced in this branch, [:124](Application/Services/GlobalRepetitionService.cs:124); the constant `PastInstantWorthReporting` at [:35](Application/Services/GlobalRepetitionService.cs:35).
- `request.AvailableAt is not { } requestedAt` — a null check and a capture in one expression; no field, return "now".
- `ToUniversalTime()` rather than `SpecifyKind` — an offset such as `+05:00` arrives as `Kind = Local` and names a different instant than its digits read; the column is `timestamptz` and rejects a non-UTC kind outright.
- A past moment is replaced by "now"; `Log.Warning` fires only past a five-minute drift.
- A future moment: `serverNowUtc + _testMode.Scale(span)`. Outside test mode `Scale` returns the span untouched, so one expression covers both.

## 11. Same method, the leave checks → 409

- The calls themselves were there before the branch, [:75–77](Application/Services/GlobalRepetitionService.cs:75).
- The second `EnsureDayIsFree` now means something else: it used to ask about the day a few hours of delay landed on, and now asks about the day the teacher named, which may be weeks out.
- The calendar loads every approved leave with no upper bound, so a repetition aimed into a distant approved leave is caught without further work.
- All three throw; an exception filter turns them into a 409 with a code, the student id and the leave's end date.

## 12. Same method, building the row

- `AssignmentDay` was there before the branch but now means something else: [:92](Application/Services/GlobalRepetitionService.cs:92) — it used to be the day the work was handed out, and is now the day it is placed on, read in the student's zone.
- The `_testMode.IsEnabled ? null : ...` branch is gone — the null existed only to dodge a unique index that this branch drops.
- `CreatedAt` and `AvailableAt` can now be weeks apart, which was impossible before.

## 13. Same method, `FillRangeFields`

- Was there before the branch, unchanged: [:95](Application/Services/GlobalRepetitionService.cs:95).
- Switches on the unit string and fills the unit number plus a page span, so the row carries pages whichever unit was chosen.
- Immediately afterwards the ayah bounds are overwritten from the request as absolute ids.

## 14. Repository `GlobalRepetitionRepository.CreateAsync`

- Return type changed from nullable to non-nullable: [GlobalRepetitionRepository.cs:18](Infrastructure/Repositories/GlobalRepetitionRepository.cs:18).
- The unique-violation catch was removed: it guarded the daily cap, and the only uniqueness left is the primary key on a freshly generated id.
- The Npgsql import and the `created == null` branch in the service went with it.

## 15. Same method, the change notification

- Was there before the branch, unchanged: [GlobalRepetitionService.cs:106](Application/Services/GlobalRepetitionService.cs:106), but it matters more now — the moment can be far off and the device must learn about it in advance.
- The target enum is `[Flags]`, so one ping can name several parts of the client's copy at once.

## 16. Response 200, `MapToResponse`

- Was there before the branch, unchanged: [:303](Application/Services/GlobalRepetitionService.cs:303).
- The availability instant is stamped `Kind = Utc`, or JSON omits the trailing `Z` and the client's parse fails.
- `IsAvailable` compares now against the opening moment; for work scheduled ahead it is usually false.

After the request:

## 17. `GlobalRepetitionRepository.GetActiveForStudentAsync`

- Was there before the branch, unchanged: [GlobalRepetitionRepository.cs:25](Infrastructure/Repositories/GlobalRepetitionRepository.cs:25), but it carries more now — it alone hides work scheduled for the day after tomorrow.
- Filters on "not done" plus, when enabled, "available at or before now"; a flag lifts the gate so the client can pre-fetch content.

## 18. `PendingPushService`

- Was there before the branch, unchanged: the catch-up window at [PendingPushService.cs:52](Application/Services/PendingPushService.cs:52).
- A projection over four tables that already store a fire instant; the window reaches two hours into the past and has no upper bound, so work scheduled at any distance appears in the snapshot.
- That is why the reminder needs no new infrastructure: the device arms its own alarm.

## 19. `LeaveTaskRescheduler` and `SetAvailableAtAsync`

- `SetAvailableAtAsync` gained a third parameter and now writes three columns: [GlobalRepetitionRepository.cs:114](Infrastructure/Repositories/GlobalRepetitionRepository.cs:114).
- The parameter is required rather than defaulted, so the compiler points at the one call site; a default would let the two columns drift apart in silence.
- The student's zone is threaded through `MoveTaskAsync` [LeaveTaskRescheduler.cs:231](Application/Services/LeaveTaskRescheduler.cs:231) and `SetOpeningMomentAsync` [:276](Application/Services/LeaveTaskRescheduler.cs:276); only the repetition arm computes a day, the other kinds have no such column.
- `PullBackAsync` [:89](Application/Services/LeaveTaskRescheduler.cs:89) gives back both the instant and the day on an early return.
- The shifting rule itself did not change: a leave moves every future repetition, not only the ones falling inside it.

## 20. `TaskStudentRepository.AssignedRepetitions`

- The query did not change but returns something else: [TaskStudentRepository.cs:345](Infrastructure/Repositories/TaskStudentRepository.cs:345) — the "starts on" day comes from `AssignmentDay`, and that column changed meaning.
- The matching filter still reads the instant inside the *teacher's* day window, so across time zones a row will not be found by the very date it displays.

## 21. `TeacherService`, the group card counter

- Not on the request's path, but broken by it: [TeacherService.cs:426](Application/Services/TeacherService.cs:426).
- The counter stays on the wire but now means something else: always zero.
- The shipped client adds it to the memorization counter to get "tasks still to hand out"; at any other value the card's progress bar would be stuck below half forever.
- The internal counterpart and the "busy" band were deleted — the first was a copy of the reachable-student count, the second unreachable.

## 22. Migration `DropGlobalRepetitionDailyUniqueIndex`

- Introduced in this branch: [20260920174322_DropGlobalRepetitionDailyUniqueIndex.cs:30](Infrastructure/Migrations/20260920174322_DropGlobalRepetitionDailyUniqueIndex.cs:30).
- `DROP INDEX IF EXISTS` as raw SQL rather than the builder's `DropIndex` — production was adopted at a squashed baseline, so the chain does not prove the index is there.
- `Down()` recreates a unique index and will fail once a student has two repetitions on one day; the comment says so outright.
- No separate migrate step is needed: this service applies migrations at startup, before it binds its port.
