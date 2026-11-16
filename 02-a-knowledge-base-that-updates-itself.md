A knowledge base that updates itself every morning
==================================================

Every day at 08:00 my server pulls 31 sources about the AI tools I work with. Most are official docs and changelogs; a few are video transcripts. When I type a prompt in Claude Code, a small hook adds a pointer to the note that fits. It adds at most 1,200 characters and takes about 0.25 seconds. Below is how it works, why official sources always win and what broke on the way.

The problem
-----------

A coding agent does not know what changed yesterday in its own tools. Claude Code ships changes often, and the docs move with it.

The easy answer is to paste the whole documentation into the context. That spends the context on pages the agent does not need, on every prompt. The second problem is mixing sources. Advice from a video can be three versions old, and if it sits next to the official docs, the agent has no way to tell which one wins.

What the system does
--------------------

1. A daily job syncs every source: official docs, changelogs, GitHub releases and the transcripts of a few creator channels.
2. Claude reads each new video and writes a short note about it. The notes are plain Markdown files, so I can read them too.
3. Everything goes into a local SQLite file with keyword search (BM25).
4. An MCP server lets my agents search it. A search returns short fragments, and a second call returns only the section or the minute of video they ask for. Never a whole document.
5. A hook runs on every prompt and adds the pointer, so the agent knows where to look before it starts.
6. Once a week, new items are proposed as additions to my notes, on a separate git branch. I read the branch and merge it, or I delete it.

It is written in Python, and the tests run offline.

The design rule: official sources rank above opinion
----------------------------------------------------

Every document gets a label when it comes in. Official docs and changelogs are "canonical". Creator videos and their notes are "practice".

When the two disagree, canonical wins. The search shows both. Each result carries its label and its date, so the agent can see the disagreement and still follow the official one. I also ask it to say when a practice note is old, because a trick from a video can stop working after one release.

Practice is still useful. Videos show how people use a tool in real work, which the docs rarely do. But when the question is how a feature works, the official page decides.

The router: a pointer, not a dump
---------------------------------

The router is a plain file with one line per topic: the keywords, the notes to open and the ids of the official pages. The hook scores my prompt against those lines, with light stemming in Spanish and English and a list of synonyms. It then adds the best one or two lines as a pointer.

A few rules keep it cheap:

1. At most 1,200 characters per prompt, whatever happens.
2. Short prompts and slash commands get nothing.
3. If no line scores high enough, it searches the knowledge base instead and adds up to two hits.
4. If anything fails, it adds nothing and the prompt goes through. A hook that sits in front of every prompt must never block one.

Code decides what goes into the prompt. The model only reads the pointer and decides whether to open more.

How I know it picks the right line
----------------------------------

For a while I judged the router by feel. That was a mistake, because a new keyword can quietly steal prompts from another line and nothing tells you.

Now it has a golden set: a fixed list of real prompts, each with the line it should get. It also has a second list written before I tuned anything, and a list of prompts that should get nothing at all, such as "thanks". The test fails if the first line it picks is right less than 80 % of the time. When it fails, I fix the keywords, not the expected answers.

What broke and how I fixed it
-----------------------------

1. The server went down under my own job. On 13 September a big backfill, started from my editor, pushed the server into the out-of-memory killer. The server has 16 GB and runs several coding sessions at once. The killer closed the editor and the work with it. Since then every heavy job runs in its own capped unit: lowest priority, half a CPU, 1,400 MB of memory at most, and first in line to die if memory runs out. It also processes at most 3 videos per run, with 60 seconds between them.
2. The video platform blocked my server. The same day, every download from the server failed with a "confirm you're not a bot" message. The block was on the server's address, so retrying could not fix it. I now send only those requests through a free proxy that needs no account. It worked on 5 of 5 test videos. Before each run, one cheap check tells whether the proxy is up. If it is down, the run skips the videos without counting a failed try.
3. Failed items kept retrying forever. Now each one waits 1 hour, then 6, then 24, then 48. After 20 tries or 30 days it is marked as failed, with the reason saved. The state is saved per item, so an interrupted run loses nothing.
4. The router picked the wrong line too often. I added synonyms and stemming. The hook can now add two lines when the top scores are close. I measure each change against the golden set.
5. My Mac had an older Python and could not run the hook. I made the hook work on the older version, so the same hooks run on both machines.

One command, kb doctor, now shows on one screen whether the proxy is up and which items are waiting or failed.

What I have not measured yet
----------------------------

1. Whether my agents answer better with it than without it. I have no side-by-side test, only the router test above.
2. How many tokens it saves. I cap what goes in, but I have not compared it with pasting the docs.
3. Cost per day: I don't have a figure.

What you can reuse
------------------

1. Label every source by authority and let the official one win. Show both, so the conflict is visible.
2. Return the fragment, not the document, and put a hard cap on what any hook adds to a prompt.
3. A hook on every prompt must fail open: on any error, add nothing.
4. Test the router with a fixed set of prompts, including ones that should get nothing. Without those, adding something to every prompt looks like a good score.
5. On a shared machine, run long jobs capped and resumable. My first backfill cost me an editor session.

The code is open source (MIT): https://github.com/mariochasco78557/ai-knowledge-feeds

Written by Mario Chasco, November 2026. https://www.mariochasco.com/en#sys-knowledge-feeds
