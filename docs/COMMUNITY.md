# Community

How to get involved, where to talk, and what help is needed most.

## Where things happen

| Channel | Use it for |
|---|---|
| [Issues](https://github.com/NurMohammadDuari/TypingMaster-RE/issues) | Bugs, concrete tasks, analysis corrections |
| [Discussions](https://github.com/NurMohammadDuari/TypingMaster-RE/discussions) | Questions, ideas, "how do I…", showing your results |
| [Pull requests](https://github.com/NurMohammadDuari/TypingMaster-RE/pulls) | Proposing concrete changes |
| `SECURITY.md` | Private vulnerability reports — **never** a public issue |

## Most-needed help right now

Ordered roughly by usefulness to the project.

1. **Close the big decompiler gap.**
   `FUN_0049196c` → `FUN_004b5584` is ~146 KB with no decompiled body
   (see `ROADMAP.md` and `analysis/ANALYSIS-DEEP.md`). Anything that
   recovers or explains it is high value.
2. **Verify the analysis.** Reproduce a claim, check cited offsets, names and
   figures, and report anything that does not hold up.
3. **Decode the lesson formats.**
   Field-level structure of `.kbd`, `.exi`/`.exm`, `.cnt`, `.hdr`, `.wzd`.
4. **Document the 35 screens.** Controls and handler wiring for each form.
5. **Explain the WPM/accuracy maths.** It funnels into `FUN_003ba4ac`
   (float constants 25 / 50 / 100 / 125).
6. **Translations.** The documentation is English-only.
7. **Small fixes.** Typos, broken links, unclear wording — genuinely useful
   and a great first contribution.

## Skills that map to tasks

| You are good at… | Try… |
|---|---|
| Reading C / disassembly | The decompiler gaps, name recovery |
| Ghidra / rizin / IDA | Re-analysing the decompilation |
| Wine / Windows internals | Explaining how the program runs |
| Writing | Documentation, format notes, examples |
| A language other than English | Translating the docs |
| Typing / keyboards | Explaining layouts, lessons, courses |
| Organisation | Triage, labels, turning discussions into issues |

Not on the list? Open a Discussion and say what you can do.

## Communication style

Be kind, be concrete, assume good faith. Point at evidence rather than
opinions. Full expectations are in `CODE_OF_CONDUCT.md`.

## Recognition

Contributors appear in GitHub's contributor graph. Recurring or significant
contributors can be listed in `AUTHORS.md` on request, and are credited in
release notes.

## A note on scope

This project studies a **commercial, proprietary** program. Community work
here is about understanding and documenting, never about cracking, pirating,
or redistributing the software. See `NOTICE.md`.
