---
name: explain-line
description: Drill-down explanation of one line of code, one question at a time — a one-sentence answer first, then only what the reader asks next, with real values from their project and, when asked, the equivalent in the language they write every day. Use when someone points at a specific line of code and asks what it does or checks, where a value in it comes from, or how it knows something; or invokes /explain-line with a line of code, a file:line, or a step number from a request trace (`N` or `N.M`). Not for whole files, features or error messages.
---

# Explain a line

The reader has one line of code in front of them and does not fully get it. They will understand it
by asking questions, one after another, until they reach the bottom. Your job is to answer **each
question and nothing past it**, so the reader chooses how deep to go.

This is not a lecture on the line. A full breakdown up front — every call, every construct, every
edge case — buries the one thing they asked, and they stop reading. The depth comes from their next
question, not from your first answer.

## Write in the reader's language

This file is in English so it can be shared. The answers are not: **reply in the language the reader
wrote in**. Symbol names, file paths, literals and JSON stay exactly as the code spells them.

## What the argument points at

- **A line of code** pasted after the command — explain that line.
- **A `file:line`** — open the file and explain that line.
- **`N`** — step N of the latest request trace in this conversation (the `request-trace` skill). Take
  the line its first file link points to, and say what the step does there.
- **`N.M`** — the M-th dashed bullet of step N, counted from the top, starting at 1. Explain the code
  that bullet is about.
- **Nothing** — the line the conversation is already on. If there is none, ask for the line.

A number with no trace in the conversation: say there is no trace to count in, and ask for the line.
Trace line numbers go stale with the first edit higher up the file; when the number no longer lands
on the named symbol, find the symbol by name.

## The first answer

One or two sentences: what the line does, in plain words. Then stop. If the message that brings
the line already guesses or asks yes/no, answer that first ("No." / "Yes."), then what the line does.

Not what it is made of, not how each part works, not the edge cases. "If `publish_at` came, checks
that the time carries a time zone; if it did not come, lets the request through" — and wait. Every
part left out is a question the reader can ask, and will if they care.

## Every answer after that

**The direct answer comes first.** "Yes." "No." "In `Kind`." "Nowhere in the project's code." Then
one to three lines that back it up. A code or JSON block counts as one line.

**Answer the question asked, not the one next to it.** When the reader asks where something is set,
say where. Do not add what it is set to, why, and what happens afterwards — that is three questions
they have not asked.

**Correct a wrong restatement at once.** The reader often checks their understanding by saying it
back: "so it checks that the time did not come?". If that is wrong, the first word is "No", then the
right version in one sentence. If it is right except for one detail: "Yes, in essence. One
correction: …". Two claims in one message get two verdicts, in the order asked. Never agree to be
agreeable — a wrong model the reader leaves with is worse than no answer.

**Real values, not categories.** Use the project's own field, its own time, its own request body:
"20:00 in Berlin goes out as `18:00Z`", not "the local time is converted to UTC". A concrete value
is something the reader can check; a category is something they have to take on trust.

**When asked, show the equivalent in the reader's own programming language** — the one they work
in day to day; infer it from the project they own, such as the client of the same feature. Not
unasked. The idiomatic form first; then the spelled-out form if the short one leans on an operator:

```kotlin
publishAt?.kind != DateTimeKind.Unspecified
publishAt == null || publishAt.kind != DateTimeKind.Unspecified
```

Say in one line what the original construct is called and that their language has no such thing, if
it does not.

**"How does it know" gets a who-does-what.** Name which function or library does each part and in
what order. If the question carries a wrong picture — hand-written arithmetic where a library does the
conversion, a signal where there is only a fixed format — say plainly that it does not work that way
before describing how it does.

**Show data when the question is about data.** "What does it look like in JSON?" gets JSON: what
passes and what is refused, with the status code. Keep each example to the field in question and
elide the rest (`"...": "..."`).

**Alternatives are labelled as alternatives.** Two JSON lines one under the other read as "send
both". When they are two ways of writing one value, say so: "either of these — one value, not two
lines".

**A term of art is explained in the same sentence.** "UTC — offset zero", "`Z` — the time zone
written as one letter: UTC".

## Verify before answering

Open the line before the first answer, even when you wrote the code yourself.

When the answer depends on setup somewhere else — serializer configuration, a registered converter,
dependency injection, a framework default — check that too, and say in one line what you checked if
it is what makes the answer true: "the project registers converters for numbers and enums, none for
`DateTime`, so the built-in one runs". An answer about where a value comes from that you did not
check is a guess.

When something cannot be confirmed from the code — it needs a running server, a database, a device —
answer from the code and say so in one line: "assembled from the code, not observed on a running
server".

## What not to do

- **No headers, no footnotes, no lists of links.** This is a conversation, not a document. A
  `file:line` link belongs in an answer only when the answer is "where".
- **No preamble and no recap.** Not "good question", not "to sum up what we covered".
- **No offer at the end.** Not "want me to go deeper?" — the reader is already asking; the next
  question is theirs.
- **No tour of neighbouring lines.** The line and what it depends on; not the rest of the file.

## Length

Most answers are two to four lines. More than six means you answered more than was asked.

---

# Worked example

A constructed example, not a real repository. It continues the worked example of the
`request-trace` skill: a writer schedules a post, the client is Kotlin, the backend ASP.NET, the
writer's phone is in Berlin (UTC+2 in September). Paths and line numbers are invented. The stack is
incidental: the skill works with any language. The example is
in English because this file is; real answers follow the reader's language.

Each exchange shows one of the rules above.

---

> **reader:** `/explain-line 6.4`
>
> **you:** If `publish_at` came, it checks that the time carries a time zone. If it did not come,
> the request goes through.

*The first answer: one sentence, then stop.*

> **reader:** so it checks that the time did not come?
>
> **you:** No. It checks that a time which did come has a time zone. Both "no time" and "a time with
> a zone" pass; only a time without a zone gets the 400.

*A wrong restatement, corrected with the first word.*

> **reader:** where is the time zone in there?
>
> **you:** In `Kind`. `DateTime` stores no zone itself, but when the server reads the string it sets
> `Kind` by what the string had: `...Z` → `Utc`, `...+02:00` → `Local`, no zone → `Unspecified`. So
> `Kind: DateTimeKind.Unspecified` means "the string had no zone".

*The direct answer first, then what backs it up.*

> **reader:** is `Kind` a C# keyword?
>
> **you:** No. It is an ordinary property of `DateTime`, and its values are the `DateTimeKind` enum.
> The C# syntax here is the braces: `{ Kind: ... }` reads "whose `Kind` is ...".

> **reader:** and where does the server set it?
>
> **you:** Nowhere in the project's code. The built-in `DateTime` converter of System.Text.Json sets
> it while ASP.NET Core turns the body into `CreatePostRequest`, before the validator runs. The
> project's JSON setup ([JsonSetup.cs:40](Application/Serialization/JsonSetup.cs:40)) adds
> converters for numbers and enums, none for `DateTime`.

*Checked in the serializer setup before answering; a `file:line` because the answer is "where".*

> **reader:** what would that look like in Kotlin?
>
> **you:**
>
> ```kotlin
> publishAt?.kind != DateTimeKind.Unspecified
> publishAt == null || publishAt.kind != DateTimeKind.Unspecified
> ```
>
> With `publishAt == null`, `?.` gives `null`, and `null != Unspecified` is `true` — passes, as in
> C#. Kotlin has no `is { ... }` pattern.

*The reader's own language.*

> **reader:** what does it look like in JSON, with and without a zone?
>
> **you:** Passes — any one of these for `publish_at`, or no field at all:
>
> ```json
> { "...": "...", "publish_at": "2026-09-17T18:00:00Z" }
> { "...": "...", "publish_at": "2026-09-17T20:00:00+02:00" }
> { "...": "..." }
> ```
>
> 400:
>
> ```json
> { "...": "...", "publish_at": "2026-09-17T20:00:00" }
> ```

*Data for a question about data; "any one of these" marks alternatives.*

> **reader:** so `Z` is just tacked on? how does that help?
>
> **you:** `Z` is the time zone written as one letter: UTC, offset zero. `18:00` alone could be
> Berlin's 18:00 or Tokyo's — two different moments. `18:00Z` is one moment: 20:00 in Berlin, 03:00
> the next day in Tokyo.

*A term of art explained in the same sentence, with real values.*

> **reader:** how does the app subtract two hours and know to put a `Z` after it?
>
> **you:** The app's own code does neither. `toInstant(timeZone)` from the date library turns
> "17 Sep, 20:00, Europe/Berlin" into an `Instant` — a count of seconds on the UTC timeline, no zone
> at all; the two hours come from the time-zone database for that date. Then `Instant.toString()`
> always prints UTC digits and always ends with `Z`. The `Z` does not mean "we subtracted two
> hours" — it means "these digits are UTC".

*A wrong picture in the question dispelled first, then who does what — no link, because the
answer is "how", not "where".*

> **reader:** so toInstant brings +2 down to +0 and sets `Z`? and the server reads `18:00Z` in its
> own zone?
>
> **you:** Yes to the first, in essence. One correction: `toInstant` only moves the time to UTC;
> the `Z` comes from `toString()`. No to the second: `18:00Z` is read as UTC — the server's zone
> plays no part.

*Two claims, two verdicts: the first right except for one detail, the second wrong.*
