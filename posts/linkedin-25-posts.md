# 25 LinkedIn posts, one per rule

Drafts. Tone: dry, concrete, one incident per post, no inspiration, no lessons-for-life. Hook in line one. Short paragraphs. Three hashtags at the end, never more. Each post ends with a small open thread, not a moral. English; German versions on request.

Series title for the first post and for the repo link: "Rules from running an autonomous AI company".

---

## Post 1 (R1)

My AI agents ran old code for four days and nothing complained.

The pipeline code sits in a versioned repository. Every machine has a run script with the hash of the version it should pull. I updated the code. The agent updated the hash on the machine it was working on. Not on the other one.

Pulling an old version is not an error. So there was no error.

Rule since then: the new hash comes from the API response of the update, never from memory, and it goes to every machine in the same session. A forgotten hash is a broken deploy, not a small oversight.

This is rule 1 of 25. All of them were written after something broke. I am putting them in a public repo over the next weeks.

#automation #aiagents #logistics

---

## Post 2 (R2)

An AI agent fixed one function and took the whole pipeline down with a docstring.

While it was in the file, it "tidied up" a comment two hundred lines further down. Forgot the closing quote. The module did not parse anymore. The scheduler reported exit code 1 for three nights before anyone looked.

Every cleanup looks harmless in the diff. The damage sits in the parts of the file the agent did not read before editing.

Rule 2: read the whole file first. Change only the failing line. Then check the diff. If anything else moved, start over.

Sounds obvious. Ask your agent to show you its last ten diffs.

#aiagents #automation #softwareengineering

---

## Post 3 (R3)

Yesterday's backup contains yesterday's bug.

The script broke after an edit. The agent did what a careful person would do: copied the backup over it. The backup had the bug we fixed the day before and lacked two functions added since. Two working days gone, old bug back.

A backup feels safe because it once worked. For yesterday's world.

Rule 3: a backup is something you read, not something you restore. Find the one block that worked, copy that block back. Never the file.

#automation #aiagents #devops

---

## Post 4 (R4)

I used to start every AI session by spawning ten specialist agents. Nine of them did nothing.

The idea was quality control: a reviewer, a security check, an architect, all present from the first minute. In practice they cost tokens and contributed on maybe one day in five. And on days the start ritual was skipped, quality control was skipped with it.

Two failure modes from one design.

Now the session start is deterministic and cannot be skipped: pick the tenant, load the last three projects from the database, propose a role. The main agent plays that role itself. A helper is spawned only when the task actually needs isolated context.

Same protection, a fraction of the cost. Rule 4.

#aiagents #claudecode #automation

---

## Post 5 (R5 and R8)

For four months my pipeline definitions lived in JSON files on a static website. They drifted from what actually ran. Of course they did.

The rule at the time was "deploy the JSON after every change". It was followed most of the time. Most of the time is not what a source of truth means.

On one day in June I moved all fifteen workflows into a database and froze the static site. The monitor reads live from the table. There is nothing left to forget to deploy.

Rule 5 and rule 8 are the same sentence: pipeline state has one home, and everything else loses in a conflict.

#automation #dataengineering #aiagents

---

## Post 6 (R6)

The same encoding error, solved three times in three weeks, each time from scratch.

My agent solved it fine every time. It just never wrote the solution down where the next session would look. Chat summaries are not storage. The next session does not read summaries. It searches.

Rule 6: before every fix, search the learnings store for the error text. After every fix, store the pair: exact error, cause, solution, context. Since August there is one more step: mark which stored entries actually helped, so the ranking learns.

An error that shows up twice without a stored learning is now counted as a process failure. Not a code failure.

#aiagents #automation #knowledgemanagement

---

## Post 7 (R7)

A scheduler showed "Running" for three weeks. It had produced nothing for three weeks.

It used print statements. No log file. No run record anywhere. The hang was found by a customer who noticed missing output.

"Add logging when needed" means logging gets added after the incident that needed it.

Rule 7: every automation script writes a log with timestamp, step and status, and one run record into a central table. Scripts without both do not ship. A deploy gate checks it, not a person.

#automation #logistics #devops

---

## Post 8 (R9 and R10)

A module got renamed in one file and not in the other two. The monitor showed "no code stored". No error. A missing key is a valid state.

The rename was correct in every single file. It was only wrong across files.

Rule 9: a script checks cross-file consistency before every deploy. Rule 10: a new workflow is created completely in one session, or not at all. Ending a session with a tab that says "Loading..." is forbidden.

Both rules were later replaced by a database validator. The principle stayed: a checklist a human runs is a checklist that gets half-run.

#automation #aiagents #softwareengineering

---

## Post 9 (R11)

My agent's memory said the pipeline had 9 modules. It had 19.

The memory note was accurate when written, three weeks earlier. Memories do not expire on their own. The session started from the wrong picture and planned changes against modules that had long been split.

Rule 11: questions about module count, file list or code content are answered from the live repository, never from memory. Memory is context and history. After every deploy, the memory entry is refreshed with count, list, hash and date.

Presenting a number from memory as a fact, without checking the source, is forbidden. That sentence turned out to apply to more than code.

#aiagents #automation #knowledgemanagement

---

## Post 10 (R12)

The fix ran on the customer's machine. Just not the fix I wrote.

The script went from my laptop to a second machine through a cloud-synced folder, and from there onto the customer VM. The sync had not finished. An outdated version ran, with the old bug, and the log looked exactly like a good run.

"Wait for the sync" cannot be verified by waiting.

Rule 12: every script that travels through a synced folder prints the first 16 characters of its own SHA-256 as its first line of output. When I hand a script to someone, I name the expected hash. If their run shows a different one, the transfer is not done.

#automation #devops #logistics

---

## Post 11 (R13)

My AI agents stopped writing German umlauts. I had to make it a rule to get them back.

One script type on one old Windows version really does break on non-ASCII characters. The agent learned that, and then applied the workaround to everything: chat, documents, commit messages. The company name has an Ü in it.

Rule 13: write the language correctly. Transliteration is a bug. The exceptions are technically forced and listed by name. When in doubt, write the correct character.

Small rule. It taught me that agents generalize workarounds faster than they generalize principles.

#aiagents #automation #writing

---

## Post 12 (R14)

The agent asked me for a Cloudflare token. It had been in the vault for weeks.

Asking a human is the safe default for a model. Here it was the wrong default, because the answer was already stored and the question cost me a context switch.

My reply, in capitals, became the name of rule 14: look in the credential vault before asking. New services go into the vault, not into an env file. Tokens are never read from memory files.

The rule is not about tokens. It is about where an agent looks before it interrupts you.

#aiagents #automation #security

---

## Post 13 (R15)

A pipeline crashed on an unexpected input and left no trace. No run record, no error entry, no mail.

The fix everyone reaches for: wrap the failing line in try/except. That protects that line. The next unexpected input hits a different one.

Rule 15: every new pipeline runs inside an outer wrapper that guarantees the run record even on a crash, and every module inside a module wrapper that logs and triggers a fallback. Several errors in one run are collected into one consolidated mail. Not one mail per error, ever again.

Existing pipelines are not migrated unless I say so. That part of the rule matters as much as the rest.

#automation #aiagents #softwareengineering

---

## Post 14 (R15b and R16)

The wrapper caught the error. Then nothing happened, because "nothing" was the registered fallback.

A wrapper that catches errors still has to be told what to do with them. Rule 15b: every code section in a pipeline has at least one registered fallback with a priority. A section without one is a defined gap and blocks the deploy.

Rule 16 came three days earlier for a related reason: log files existed, but answering "which fallback actually fired last night" meant opening files on three machines. Now every log line also lands in a central table, linked to the run, the module and the section.

Operations questions span machines. File logs do not.

#automation #devops #aiagents

---

## Post 15 (R17)

A checklist nobody executes is executed by hope.

After moving pipeline state into a database, the old manual checks did not apply anymore. A new pipeline shipped with a step pointing at a module version that did not exist. A user found it.

Rule 17: after every pipeline change, a validator checks nine invariants. Exit 0 or 2 closes the deploy. Exit 1 or 3 blocks it. Ignoring the output is forbidden.

The rule is one sentence. The validator is 400 lines. That ratio is normal.

#automation #devops #aiagents

---

## Post 16 (R18)

Twenty minutes diagnosing a firewall problem. The IP in the memory file was from before the VM was rebuilt.

Memory files about infrastructure are snapshots. They never say they are stale.

Rule 18: every machine's data, IP, specs, SSH access, mail account, lives in one table with a view for the UI. Changes go into the table. In a conflict the table wins. A VM without a row does not exist.

Rules 5, 8, 11, 14 and 18 all say one thing: one home per kind of truth. I needed five incidents to hear it.

#automation #infrastructure #aiagents

---

## Post 17 (R19)

A nightly import slowed down every customer pipeline for hours. The import had no idea. It only measured its own progress.

It had saturated the database's disk IO budget. Adding a limit afterwards protects against that crash. The next unbounded process is a different one.

Rule 19: every unattended process that consumes a limited resource ships with three parts from day one. Adaptive pacing. A hard kill switch with automatic pause and hysteresis. A run record plus alert when the guard fires.

The three parts are in the creation checklist. Going live without them is forbidden. Nothing is "add later".

#automation #dataengineering #aiagents

---

## Post 18 (R20)

We asked a customer how we could get access to a planning file. We had had access for weeks. Through a different project for the same customer.

The knowledge existed. It was stored under the other project, and nobody searched across projects. The customer noticed.

Rule 20: before every customer-facing action, search the customer's own knowledge namespace, and for anything about data or systems, specifically the access entries. A question to the customer is allowed only when those searches come back empty, and the empty result is written down.

The rule has a second half most people skip: after every call and every content mail, the facts go into that namespace line by line, with source and date. Otherwise there is nothing to search.

#logistics #automation #customersuccess

---

## Post 19 (R21)

A number carried through five sessions was off by thirteen orders of magnitude.

Side project, research context. The number came from a chat output and was never recomputed. "Double-check the numbers" does not say what a check is.

Rule 21: every quantitative result carries a status, hypothesis, indication, confirmed or calibrated, and the status is never more generous than the completed steps justify. Prediction before computation. Controls on a known system. Limitations section, always.

A number without status is not a result. That applies to business numbers just as much as to physics.

#research #aiagents #datascience

---

## Post 20 (R22)

I banned the em dash in one sentence. It was the most effective rule of the year.

Generated mails and documents were full of em dashes, bold lists and headings. Recipients read them as machine-written, because they were. A style note in the prompt loses against the model's preference. A hard check does not.

Rule 22: no em dash, no en dash, anywhere. Mail bodies are plain text in normal paragraphs. No bold, no bullets, no tables. Structure goes into an attachment. A script checks before every send.

Look at the last mail you received from a company. Count the dashes.

#writing #aiagents #automation

---

## Post 21 (R23)

The watchdog said healthy for four days while the pipeline sent nothing.

"The job ran" was its definition of healthy. The job did run. A DNS blip four days earlier had killed a different job, the one that refills the queue, and that one had no run record and no alert.

Four earlier incidents in five weeks had the same shape. PDFs not generated. Notices not sent. An export without attachment. A monitor on a dead source. Every time the process was alive and the output was missing.

Rule 23: every unattended automation declares its expected output. One number per run, a minimum, an idle window. An idle streak beyond the window alerts like a crash. A negative result is never recorded as success.

Heartbeat is a sign of life. It is not a sign of work.

#automation #monitoring #aiagents

---

## Post 22 (R23, second half)

A manual copy step between two systems is a bug with a person in it.

Same incident as the last post. Seven access codes were in the database and not in the deployment secret, because someone was supposed to copy them over. The watchdog for that step would have been the wrong fix.

Rule 23, second half: where an automation mirrors state between two systems, the metric is the number of discrepancies, measured in the target, expected zero. And more important than any alarm: the copy step gets automated, not monitored.

The guard is the net. The net is not the plan.

#automation #devops #aiagents

---

## Post 23 (R24)

An AI agent sent a customer mail on its own. Its task file told it to.

The file had been generated the night before by another agent. It said "check and send, CC the service address". The morning session read that as approval. The rule "never send customer mail without approval" already existed. The file looked like approval.

Rule 24: the only valid approval is me seeing the finished draft and approving it live. "Send" in a prompt file, a plan, a calendar entry or an agent briefing is not approval. And agents that generate task files are forbidden from writing a send instruction into them at all.

A file cannot grant permission its author did not have.

#aiagents #automation #governance

---

## Post 24 (R25)

My agent asked me the same question two days in a row, as the top priority both times. One of the drafts it was asking about was already in my trash.

The session summary said "waiting for approval". My decision had never been written down. Summaries are for understanding. They cannot know what happened after they were written.

Rule 25: status is computed from the primary source, not remembered from a summary. Every open item gets a checkable done condition: subject in Sent, reply from sender, workflow run, metric, stored decision. An observer runs the checks every 15 minutes and closes items with evidence. A deleted draft counts as a decision without words.

Two weeks later, 70 of 91 open items turned out to be auto-generated questions nobody closed. Dropped all 70 in one command. Rule got an addendum.

#aiagents #automation #productivity

---

## Post 25 (the whole list)

Twenty-five rules. Not one was written in advance.

Rules 1 to 3 are about editing: change only the broken line, never restore a whole backup, update the hash everywhere. Rules 5, 8, 11, 14, 18 say the same thing five times: one home per kind of truth. Rules 7, 9, 10, 15, 16, 17 are the pipeline version: logged, wrapped, validated, or not shipped. Rules 19 and 23: unattended processes are bounded and measured by output, not by heartbeat. Rules 20, 22, 24: the customer's side of the screen. Rules 4, 6, 25: the agent's own memory.

Every rule has a date and an incident. The repo has all of them, stripped of customer names.

If you run agents unsupervised and have not had at least ten of these incidents yet, you will. Take the rules for the ones you had. Ignore the rest until it is their turn.

Link in the first comment.

#aiagents #automation #opensource
