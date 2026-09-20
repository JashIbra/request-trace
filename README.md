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
- Nothing calls it: `AddValidatorsFromAssemblyContaining` finds it by scanning the assembly, and
  `AddFluentValidationAutoValidation` runs it before the controller's first line.
- `RuleFor(x => x.PublishAt)` names the property. The lambda is passed as an expression tree, so
  the library reads the property *name* out of it; that name becomes the key in the 400 body.
- `.Must(...)` is the predicate itself, `true` meaning valid. `is not { Kind: DateTimeKind.Unspecified }`
  is a property pattern; it does not match on `null`, so an absent field passes.
- On failure ModelState is invalid and `[ApiController]` returns the 400 on its own.
````

Every entry carries a file and a line as a link, so the reader jumps straight to the code instead of
scrolling for it.

## What it insists on

- **The unit is a step, not a file.** One method can produce five consecutive entries; an untouched
  file still appears if the request goes through it.
- **Every name is marked with one of three states** — introduced in this branch, already there and
  unchanged, or *already there but now meaning something else*. The third is what review misses most
  often: the diff is tiny or absent while the meaning has moved.
- **Library calls get explained.** `RuleFor`, `encodeDefaults`, `[Flags]`, `expectSuccess` — the
  reader is not obliged to remember what each one does.
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
