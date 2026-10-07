---
name: request-trace-ste-short
description: A short request trace in ASD-STE100 wording — the condensed walk of `request-trace-short` (about ten steps, one-sentence bullets) written in the simple style of `request-trace-ste`. Use only when the user types /request-trace-ste-short or asks for a short request trace "in STE" or "in simple language". For the full STE trace use `request-trace-ste`.
---

# Request trace — short, STE wording

This is `request-trace-short` written in the style of `request-trace-ste`.

## What to do

1. **Load the `request-trace-short` skill and follow it.** It in turn loads `request-trace`; its limits
   on length win wherever the two disagree.
2. **Load the `request-trace-ste` skill and apply its style rules** to every sentence: the bullets, the
   notes, the captions, the one line in the chat. Take only the style from it, not its instruction to
   follow the full trace.

Do not copy or paraphrase the other skills' rules into your answer. Do not mention that there are
variants. Do not ask which one the user wants.

## Where the two meet

- A bullet is one sentence in both: the short skill sets how many, the STE rules set how each reads.
- When a sentence has to be split for STE and the step would exceed five bullets, drop the least
  useful fact rather than the split.
