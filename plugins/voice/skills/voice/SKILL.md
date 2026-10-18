---
name: voice
description: Voice mode — from now on every answer is short and is also spoken aloud at once through the Mac's speakers with the macOS `say` command, no file and no tap needed. Use when the user types /voice, or says «голосом», «озвучивай ответы», «голосовой режим». Stays on until the user says «без голоса», «стоп голос» or «выключи голос».
---

# Voice mode

The user is looking at something else — code under review, a device — and asks quick questions. Every
answer from now on is spoken aloud the moment it is ready, so they never have to look at the chat or tap
anything.

## Turn on

Reply with one short line ("Голос включён.") and speak it, so the user hears the mode is on.

## Every answer while the mode is on

1. **Short.** One or two sentences: the answer itself, no preamble, no recap, no offer to go deeper.
   A question that genuinely cannot be answered that short (a plan, a review, a walk through code) gets
   its normal written answer, and only its first sentence — the verdict — is spoken.
2. **Write it** as the reply text, in the user's language.
3. **Speak it** in the same turn, with a backgrounded Bash call so the reply is not held up:

   ```bash
   killall say 2>/dev/null; say -v Milena -r 185 "<the spoken text>"
   ```

   - `killall say` first, so a new answer cuts off the previous one instead of talking over it.
   - Pick the voice for the reply's language: `Milena` for Russian; for another language take a voice
     from `say -v '?'`.
   - Inside the quotes escape `"` and `$`, or pass the text through a here-doc into `say -f -`.

## What goes into the spoken text

- Plain words only. No code, file paths, links, symbol names, `snake_case`, numbers with units
  glued on — the synthesizer reads them as noise. Say what the thing is: not "`syncInterval` in
  `RepetitionTaskDao`" but "синхронизация этапов в базе телефона".
- The written reply may keep the symbols; only the spoken copy is cleaned.
- Numbers as the language says them aloud ("девять часов", not "9h").

## Stop

- «без голоса», «стоп голос», «выключи голос» — the mode ends; answer that one in text, and from then
  on answer as usual.
- «стоп» or «замолчи» while it is talking — run `killall say` and nothing else.

## Limits to keep in mind

- The sound plays on the Mac this session runs on, not on the user's phone.
- No sound can be started from the chat app itself, so this is the only way to answer aloud without a tap.
