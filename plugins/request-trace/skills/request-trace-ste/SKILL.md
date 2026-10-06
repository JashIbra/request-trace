---
name: request-trace-ste
description: Request trace written in the style of ASD-STE100 (Simplified Technical English), about 80% strict — the same numbered walk through the code as `request-trace`, with every sentence short, one idea each, active voice, present tense. Use only when the user types /request-trace-ste or asks for the request trace "in STE", "in simple language" or "by ASD-STE100". For the ordinary trace use `request-trace`.
---

# Request trace — STE variant

This is the **same trace** as `request-trace`. Only the writing changes.

## What to do

1. **Load the `request-trace` skill and follow it in full.** Use the Skill tool with `request-trace`.
   Everything there stays in force: what to trace, the unit of a step, the file and line links, the
   state labels, the footnotes, the HTML page, the fragments next to the steps, the line-number and
   verification rules, the length limits.
2. **Then apply the style rules below to every sentence you write** — the bullets under each step,
   the footnotes, the notes next to the pictures, the one line in the chat.

Do not copy or paraphrase the other skill's rules into your answer. Do not mention that there are two
variants. Do not ask which one the user wants.

## The style rules

- **One idea per sentence.** About 20 words at most. If a sentence joins two actions with "and", make
  two sentences.
- **Active voice, present tense.** "The validator checks the field." Not "The field is checked by the
  validator."
- **One word, one meaning.** Name a thing the same way every time. Do not use a synonym for variety.
  Do not replace the name with "it", "this" or "that" — write the name again. (`request-trace`
  already requires this. In STE it is stricter: the bullet must read alone.)
- **No figures of speech.** No metaphors, idioms or analogies, unless the user asks.
- **Instruction first, reason second.** A warning is its own sentence, and it comes before the action
  it concerns.
- **Explain a new term once**, in one short sentence, where it first appears.
- **Numbers, not "many" or "a lot".** Every count, limit and timeout goes in, as the base skill
  already requires.
- **No filler.** No "in fact", "basically", "it is worth noting".

## About 80%, not 100%

Follow the rules, but not to the letter. In Russian a word-for-word result reads like a machine, and
the user rejects it. Keep a natural word order. Join two very short sentences when the split sounds
broken. The goal is clarity, not conformity to a dictionary.

## What does not change

- The language: write in the language the user writes in. Do not switch to English.
- Symbol names, file paths, HTTP verbs, status codes and library calls stay as the code spells them.
- The structure: numbered steps, dashed bullets, state chips, one line in the chat with the file path.
- The facts. A simpler sentence must say the same thing as the original. Do not drop a number or a
  condition to make a sentence shorter.
