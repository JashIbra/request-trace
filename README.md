# request-trace

A Claude Code skill that walks **one user action through the code** — from the tap in the client to
the response, and on into whatever happens after it — and spells out the mechanics of every call
along the way.

It exists for code review. A list of changed files tells you where to look; it does not tell you
what runs after what. This produces the route, and under each stop it answers, in advance, the
questions a reviewer is about to ask: what is this function, where did this variable come from, why
is the code here at all.

## What it produces

````markdown
## 6. Validator `CreatePostRequestValidator` → 400

- File introduced in this branch; the rule sits at [CreatePostRequestValidator.cs:14](Application/Validators/CreatePostRequestValidator.cs:14).
- No code in the project calls it — ASP.NET does: `AddValidatorsFromAssemblyContaining`¹⁰ registers it
  at startup, and `AddFluentValidationAutoValidation`¹¹ makes the framework run it on each request,
  before the controller's first line.
- `RuleFor(x => x.PublishAt)`¹² — the checks chained after it apply to the `PublishAt` field. The
  compiler hands the library a description of the lambda rather than its code — an expression tree¹³ —
  so the library reads the field's name without running anything and writes it into the 400 body as
  `publish_at`, telling the client which field is wrong.
- On failure ModelState¹⁷ is invalid and `[ApiController]`¹⁸ returns the 400 on its own.

…

## Notes

10. `AddValidatorsFromAssemblyContaining` — FluentValidation. Scans the assembly that contains the
    given type and registers every validator class it finds in the dependency-injection container.
12. `RuleFor` — FluentValidation. Starts a rule for one property of the object being validated; the
    checks chained after it apply to that property.
13. Expression tree — a C# feature. When a lambda is passed to a parameter of type `Expression<...>`,
    the compiler hands over a description of the lambda's code rather than compiled code, so the
    receiver can read, for instance, which property it touches.
````

Every entry carries a file and a line as a link, so the reader jumps straight to the code instead of
scrolling for it. Every name the project did not write — a library call, a language construct, a
database type — carries a small superscript number pointing to a note at the end that says what it is.

## What it insists on

- **The unit is a step, not a file.** One method can produce five consecutive entries; an untouched
  file still appears if the request goes through it.
- **Every name is marked with one of three states** — introduced in this branch, already there and
  unchanged, or *already there but now meaning something else*. The third is what review misses most
  often: the diff is tiny or absent while the meaning has moved.
- **Everything the project did not write gets a footnote.** `RuleFor`, `flatMapLatest`,
  `encodeDefaults`, `timestamptz` — a superscript number at the first mention, and a numbered note at
  the end saying what the thing is. The bullet keeps to how this code uses it.
- **Refusal points are flagged with their status codes**, and so is anything running before the
  controller, which is easy to miss because nothing calls it explicitly.
- **No diagrams, no preamble, no summary.** A sequence diagram of the same trace holds less and
  cannot be pasted into a ticket.

The skill is written in English so it can be shared; the trace it produces follows the language the
person asked in.

## Install

```
/plugin marketplace add JashIbra/request-trace
/plugin install request-trace@request-trace-marketplace
```

Or, without the plugin machinery, copy `plugins/request-trace/skills/request-trace/SKILL.md` into
`~/.claude/skills/request-trace/SKILL.md`. Personal skills are picked up at session start, with no
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
