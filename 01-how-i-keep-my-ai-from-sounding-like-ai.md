How I keep my AI from sounding like AI
======================================

Nothing I sign goes out until a Python script of about 500 lines says PASS. That covers job applications and messages to clients, video scripts and this note too. Below is why I moved the job from the prompt to code, what the script looks for, and the day it rejected 4 of my 6 drafts.

Why code and not a longer prompt
--------------------------------

My first attempt was the obvious one: a long voice guide in the prompt. It helped. Then on 5 November I did a blind test, mixing texts I wrote with texts the model wrote in my name. The generated English drafts were easy to pick out. They read too technical.

A prompt is a request. The model follows most of it, most of the time, and the misses are random. A script runs the same checks on every text, needs no model call and uses only the Python standard library. So I use the same split as in my other systems: code checks everything I can write down as a rule, and the model writes.

The script does not judge whether a text is good. It looks for marks I have seen in AI text and never in mine.

What it looks for
-----------------

Some findings fail the text. Others only warn, and a person decides. The main groups:

1. Phrases models love and I never write: 50 of them, 38 in English and 12 in Spanish. I can't quote most of them here, because this note goes through the same check and would fail. Think of the opener about hoping the email finds you well, or the fancy verbs for "use" and "look into". All of them fail.
2. The long dash used as a pause. It fails in every channel. Inside a date range a short dash is fine.
3. Emojis fail everywhere. A plain ":)" is allowed, because it is just text.
4. Markdown where the text should be plain. That is why the headings in this repo are underlined: a hash mark at the start of a line would fail this note.
5. Shapes of sentence. The not-this-but-that sentence warns once and fails from the second. More than one list of three items per 300 words warns.
6. Flat rhythm. The script measures every sentence. If the standard deviation of their lengths is under 6 words it warns, and under 4 it fails.
7. No concrete facts. It counts numbers with a unit, dates, links and named tools. An email needs 2; a public note like this one needs 1.
8. Jargon: 27 technical words I rejected in English, such as "schema". From 2 in one text, it warns.

Same rules, different channels
------------------------------

A WhatsApp reply and a job application should not pass the same bar. Each run says which channel the text is for.

| Channel | Used for | Concrete facts required | Extra checks |
|---|---|---|---|
| email | Applications and emails | 2 | A warning above 220 words |
| form | Answers in application forms | 2 | A bullet list fails |
| doc, post | Public notes like this one, posts | 1 | None |
| whatsapp, chat, meeting | Short replies, instructions to tools, spoken turns | 0 | None |
| script | Video and audio scripts | 0 | Spoken checks, see below |

A script fails if it opens by summarising the video instead of saying what changed in my work. It also fails with hype words like "incredible", or with two spoken fillers in the same sentence.

The pipeline that drafts my job applications adds a stricter layer. Over 220 words fails, not just warns. The first sentence has to name the company or the role. It also checks shapes that one text alone cannot show:

1. More than one line that opens with a label and a colon.
2. A sentence that judges my own fit, such as "strong fit".
3. The same 8-word phrase in more than 2 of the last 30 drafts. Facts from my profile are exempt.
4. Any link that is not in my list of facts or in the job post.

What happens when a draft fails
-------------------------------

The pipeline never sends anything. The only thing it can do in Gmail is create a draft, and a test fails if anyone adds a send function.

1. The model writes a draft.
2. Code runs every check. Warnings go into a note for me next to the draft. A single fail rejects it.
3. On a rejection, the model gets one more try. The new prompt opens with "YOUR PREVIOUS DRAFT WAS REJECTED BY CODE FOR THESE REASONS" and lists each error as the code wrote it, with the offending line quoted.
4. If the second draft fails too, there is no draft. The job is marked as an error and shows up in the run counters.
5. If the only problem is a link outside my list, the draft is saved to a file for me. It never reaches Gmail.

The model gets a second chance to fix its text. It never gets a failed text in front of a reader.

The day it rejected 4 of 6 drafts
---------------------------------

On 7 November I ran the pipeline on a fixed set of 6 real job posts, with a stronger model writing the drafts. The checker rejected 4 of the 6. The model was not the main cause:

1. The repeated-phrase check was comparing each new draft with every old one. The old drafts came from an earlier prompt and all copied the same example, so new drafts failed for phrases they never chose. Now each draft records its prompt version, and only the current version counts.
2. My own prompt asked for one sentence that opens with a gap label, plus a separate story about a failure. That gave two label openers, and my checker allows one. The prompt was breaking the checker's rule. Now it asks for both inside ordinary sentences.
3. Drafts were too long. The target is now 120 to 180 words, and the fail message says how many words to cut.

Each rejection became an offline test that reproduces it. None of the fixes made the checker easier on bad text. One fixed the checker itself, and two fixed my prompt.

The same day, an agent working on the pipeline added a field to the shared checker and broke a test. I restored the file byte for byte. Now installed skills are read-only for agents.

What the checker cannot do
--------------------------

It cannot tell whether a fact is true. That is a separate rule: the model may only use facts from my profile files.

It cannot tell whether a text sounds like me either. It only knows the patterns I gave it. So I still read everything before it goes out, and every correction I make becomes a dated line in my rules file. When a mistake keeps coming back, it goes into the script.

What you can reuse
------------------

1. If you can write a rule as a pattern, put it in code. Keep the prompt for what code cannot see.
2. Keep fail and warn apart. A fail stops the text; a warning goes to the person who reads it.
3. Use the same rules with a different bar per channel. A WhatsApp reply does not need two numbers.
4. When a draft fails, give the model the checker's own words and the offending line, then cap the retries. Mine gets two tries, then the job comes back to me.
5. When the checker rejects a lot at once, look at your prompt and your test data before you loosen the checker. In my case, 2 of the 3 causes were in my prompt.

Written by Mario Chasco, November 2026. https://www.mariochasco.com/en
