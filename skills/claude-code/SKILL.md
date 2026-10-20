---
name: work-conventions
description: Standing rules for how to work in this project. Apply this skill on every task in this directory, not only when asked — any question about the data or code here, any markdown or documentation you write, any factual claim that could be out of date, any time you are about to create a file, and any time you are about to run a git command. It covers five things - do not create files nobody asked for, verify answers against the data before you give them, search the web instead of trusting memory, write documentation in ASD-STE100 Simplified Technical English that states reasons positively without referring to earlier drafts, and never commit or push (the user does that).
---

# Work conventions

These rules apply to all work in this project. Read them once at the start of a session and keep them in mind throughout.

## 1. Do not create files nobody asked for

Create a file only when the user asks for one, or when the task cannot be completed without it. A finished task does not need a `SUMMARY.md`, a `NOTES.md`, a `PLAN.md`, or a `CHANGES.md` next to it. Report what you did in the chat reply instead.

Why this matters: each unasked-for file is clutter the user must find, read, and delete. Over many tasks the clutter hides the files that matter. A chat reply gives the same information and leaves nothing behind.

Before you call `Write`, ask: did the user ask for this file, or does a script or tool need it? If the answer to both is no, put the content in your reply.

When the user does want notes or documentation, put them in one file they named or in the project's existing docs location. Do not spread small files across directories.

## 2. Verify answers against the data before you give them

When a question depends on the data or code in this directory, open the files and check. Do not answer from a guess about what the files probably contain.

Use a scratchpad for the check: a short script, a query, a `Read` of the relevant file, a `Grep` for the pattern. Run it. Then answer from the result.

Why this matters: a confident answer built on a hunch looks the same as a verified one, but only one of them is correct. The user cannot tell the difference from the reply alone. The cost of a quick check is seconds. The cost of a wrong answer is work built on a false premise.

Put scratch work in a temporary location (`/tmp`, or a `scratch/` directory the user already uses), not in the project tree, and do not leave it behind. The scratchpad is for finding the answer, not for delivering it. Rule 1 applies.

Examples of questions that need a check:
- "How many rows have a null in column X?" — run the count.
- "Does the loader handle gzipped input?" — read the loader.
- "Which years are in the panel?" — list the distinct values.

If a check is impossible (the data is not here, or the question is about intent rather than content), say so and give your best reasoning with that caveat attached.

## 3. Search the web instead of trusting memory

When a factual claim could have changed since training — a current office holder, a price, a tool's current version, a law in force, what a named product does today — run a web search before you state it. This applies even when the memory feels detailed and certain. Detail is not evidence of currency.

Why this matters: a dated answer delivered with confidence is worse than an unfamiliar one, because nothing about it looks wrong. The user has no signal to check it.

Stable knowledge does not need a search: how an algorithm works, a historical date, a definition. If you are unsure whether something is stable, search.

Include the name the user wrote in at least one query. A search for the broad category can miss the specific thing they asked about.

## 4. Write in ASD-STE100 Simplified Technical English, applied at about 75%

Use Simplified Technical English for prose, documentation, comments, commit messages, and chat replies. Apply the core rules consistently. Apply the strict vocabulary rules loosely: the goal is clear, plain text, not a certified STE document.

Core rules to keep:
- One topic per sentence. One instruction per sentence in procedures.
- Short sentences. Aim for 20 words or fewer in procedures, 25 or fewer in descriptions.
- Active voice. "The script reads the file," not "The file is read by the script."
- Present tense unless the time really is past or future.
- Imperative mood for instructions. "Run the test," not "The test should be run."
- One meaning per word. Use the same word for the same thing throughout.
- No noun strings longer than three words. "The panel of school board members by district," not "the district school board member panel."
- Articles ("the", "a") stay in. Do not drop them to sound terse.
- Lists for sequences and sets. Prose for reasons.

Rules to relax (the other 25%):
- Domain terms stay as they are. "Fixed effects," "endogeneity," "DataFrame" are fine.
- The approved-word dictionary is a guide, not a gate. If a common word is clearer than the STE-approved word, use the common word.
- Chat replies can be a little warmer than a maintenance manual. The sentence rules still apply.

Why this matters: STE removes ambiguity. Every reader, including a future Claude session, gets the same meaning from the same sentence. It also makes documentation shorter, which makes it more likely to be read.

## 5. State reasons positively. Do not refer to earlier drafts

When you document a design choice, give the reason the choice is right. Do not frame it as a rejection of what came before.

Why this matters: documentation describes the system as it is. A reader who was not present for the earlier draft gains nothing from a comparison to it, and the comparison ages badly as the earlier draft fades from memory. The positive reason stays true on its own.

Write this:
> The panel keys on (district_id, school_year). A district can change its name, so the stable identifier is the key.

Not this:
> In an earlier draft we keyed on district name. That broke when districts renamed. So we switched to district_id.

Phrases that signal the wrong framing: "previously," "in an earlier version," "we used to," "this replaces," "unlike before," "the old approach." When one appears, rewrite the sentence to state what is true now and why.

The exception is a changelog or a migration note, where the reader's question is "what changed?" There, a before-and-after comparison is the content. Everywhere else, give the positive reason.

## 6. Do not commit or push. The user does that

Edit files in the working tree and stop there. Do not run `git commit`, `git push`, `git merge`, `git rebase`, `git reset`, `git stash`, or any command that rewrites history or sends changes to a remote. Do not open a pull request.

Why this matters: a commit is the user's record of what they reviewed and accepted. A push makes that record visible to other people. Both steps need a human who has read the diff. When you leave the changes uncommitted, the user keeps that review step and keeps control of the history.

Read-only git commands are fine and often useful: `git status`, `git diff`, `git log`, `git blame`, `git show`. Use them to understand the state of the tree.

When your work is ready, say so in the reply and list the files you changed. The user will commit. If the user tells you in the chat to commit or push in a specific case, that instruction overrides this rule for that case only.
