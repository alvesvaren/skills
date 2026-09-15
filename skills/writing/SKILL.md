---
name: writing
description: "Prose style for anything that is not code: chat replies, docs, READMEs, ADRs, commit messages, PR descriptions, skills, CLAUDE.md. Plain words, one thought per sentence, no AI tells, no ambiguity. Invoke with /alve-skills:writing when writing or reviewing prose."
disable-model-invocation: true
---

# Writing

Write for a tired engineer reading once. Three rules sit above everything else:

- **Cut every word that does no work.** If the sentence survives without the word, the word goes. "In order to" is "to". "It is important to note that" is nothing.
- **Use the short, everyday word.** "Use", not "utilize". "Help", not "facilitate". Use a longer word only when it is more precise.
- **When a rule makes a sentence worse, fix the sentence another way.** The rules serve the reader. A sentence that follows every rule and sounds machine-written has failed.

## Say the real thing

- The codebase is the word list. Write the real symbol, file, flag, or command name, not a synonym or a description of it.
- Use words a developer would say out loud: "move", "delete", "retry", not "evacuate", "ratchet", "orchestrate". Define a named pattern the first time it appears.
- Be specific over sterile. Not "schema changes can cause issues" but "a column rename fails the build". Not "experts believe" but the named source, or nothing.
- Cut significance inflation and promotion: "pivotal", "testament to", "landscape", "robust", "seamless", "groundbreaking". State what happened.
- Say "is" and "has". Not "serves as", "stands as", "boasts", "features".
- Cut "-ing" trailers that add nothing: "..., ensuring reliability", "..., highlighting the need for". Delete or make them a sentence with a real claim.

## One thought per sentence

- One instruction or one thought per sentence. Split instructions longer than about 20 words and other sentences longer than about 25.
- Talk to the reader as "you", in the present tense. "Will" only for things that happen later.
- Say who does what: "the compiler checks", not "is checked". Passive only when the actor is unknown or beside the point.
- Write instructions as commands with the condition first: "To delete the document, click Delete." Never "should be done". Never "simply", "just", "easy", or "quickly" in a procedure.
- Common case first, exceptions after.
- Mix sentence lengths on purpose. Short sentences make a point. Longer ones carry a fact with its condition or consequence. One thought per sentence does not mean one length per sentence.

## Leave no sentence open to two readings

- Keep "only" and "not" next to the word they change. "Only fails on growth" and "fails only on growth" say different things.
- Make every "it", "they", and "this" point at one obvious noun. Repeat the noun when in doubt. Never use "this" or "which" to point at a whole clause.
- Keep the small words that show structure: "the", "a", "that". "Remove backup file" reads two ways. "Remove the backup file" reads one.
- Break up noun strings: "the proto import budget check script" becomes "the script that checks the proto-import budget".
- Say which parts "and" or "or" joins when a sentence can group two ways. "Both... and" and "either... or" settle it.
- Call each thing by one name, everywhere. Pick "start" or "launch", not both. Synonym cycling teaches the reader three things where there is one.
- No slashes: write "a, b, or both", not "a/b" or "and/or". No "(s)" plurals. No idioms, Latin abbreviations, or metaphors.

## Remove the AI tells

Punctuation and formatting:

- No em dashes and no semicolons. Start a new sentence or use a comma.
- Colons only before a list or an example, never as a mid-sentence connector.
- Sentence case headings. Headings carry the point ("Pick the mode first"), not just the topic ("Modes").
- No decorative emojis. Bold only for UI elements. Code, paths, flags, and symbols in code font.
- Straight quotes.

Shapes:

- No "It's not just X, it's Y." State the point.
- No forced groups of three. Use the natural number.
- No false ranges: "from X to Y" only when X and Y sit on a real scale.
- Bullets for parallel items, numbers for sequences, prose for an argument. Introduce a list with a complete sentence and keep items parallel.

Chatbot artifacts:

- No "Great question", "You're absolutely right", "Hope this helps", "Let me know if". Answer directly.
- No hedging stacks. "Could potentially possibly" is "may".
- No generic conclusions. End when the content ends. No summary of what was just said, no offer of further help.

## Have a view where it belongs

Dry by default. Chat replies and explanations may carry an opinion. Say what you make of a tradeoff instead of listing pros and cons, and label it as in [pushback](../pushback/SKILL.md). Reference docs, how-tos, commit messages, and PR bodies stay dry.

## Per format

- **Chat replies.** Lead with the answer or outcome. If something could not be verified, say so first. No headers under about 500 words. Numbers go in a table or on their own line, not in prose.
- **Commits and PRs.** A briefing a reviewer reads in under a minute. What changed and why, not how you got there. No logs, SHA lists, or metric dumps. Link them.
- **Docs and READMEs.** One document, one purpose. Link to the neighboring doc instead of mixing a how-to into a reference.
- **Skills and CLAUDE.md.** Every line must be a rule an agent can check. Cut motivation to one clause. Apply the writing-for-agents skill too when it is installed.

## Swedish

The same rules apply. In addition: write compound words as one word ("kodgranskning", not "kod granskning"), use the Swedish term where one exists instead of the English loan, and keep "du" throughout.

## Before finishing

Re-read the text once as the reader, then check:

1. Can any word be cut without losing meaning? Cut it.
2. Does any sentence carry two thoughts? Split it.
3. Is every instruction a command with its condition in front?
4. Is "only" next to its word? Does every "it" and "this" point at one noun?
5. Does each thing have exactly one name?
6. Any em dash, semicolon, connector colon, title case, or emoji left?
7. Would a developer say these words out loud?
8. Are all symbols, paths, and counts real at this commit?
