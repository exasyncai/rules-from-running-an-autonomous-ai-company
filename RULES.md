# The 25 rules

Format per rule: **Incident** (what happened, when), **Why the obvious fix was not enough**, **Rule** (required, forbidden).

Customer names, internal system names and business data are removed. Dates are the dates the incident was documented, not the day it started. Where the exact date was not recorded, the quarter is given.

---

## R1. Update the version hash on every machine, every time

**Incident (Q1 2026).** Pipeline code lives in a versioned Gist. Each machine that runs it has a run script with the hash of the version it should pull. After a code update the hash was left unchanged on one of two machines. The pipeline kept running old code for days without any error, because pulling an old version is not an error.

**Why the obvious fix was not enough.** "Remember to update the hash" is not a rule, it is a wish. The agent remembered on the machine it was working on and forgot the other one.

**Rule.** After every code update, take the new hash from the API response of that update (never from memory, never guessed) and write it to every machine that runs the code, in the same session. A forgotten hash is treated as a broken deploy, not as a small oversight.

---

## R2. Change only the broken line

**Incident (2026-03-20, found 2026-03-23).** While fixing one function, the agent also "tidied up" a docstring in another file. It forgot the closing triple quote. The module no longer parsed, the whole nightly pipeline stopped, and it took three days for anyone to notice because the scheduler reported exit code 1 and nothing else.

**Why the obvious fix was not enough.** Every cleanup looks harmless in the diff. The damage comes from the parts of the file the agent did not read before editing.

**Rule.** Read the whole file before the edit. Change only the failing location. After the edit, check the diff: only the failing location may differ. No reformatting, no restructuring, no "while I am here". If in doubt, make the smallest possible change.

---

## R3. Never restore a whole backup

**Incident (Q1 2026).** A script broke after an edit. The agent copied yesterday's backup over it. The backup contained the bug that had been fixed the day before, plus lacked two functions added since. Two working days of changes were gone, and the original bug was back.

**Why the obvious fix was not enough.** A backup feels safe because it once worked. It worked for yesterday's world.

**Rule.** A backup is a reference to read, not a file to restore. Identify the specific function or block that worked, copy only that block back. Copying a whole backup file over the current file is forbidden.

---

## R4. Start every session the same way, and spawn helpers lazily

**Incident (2026-06-27).** The session-start protocol required the agent to spin up a full team of specialist agents before doing anything. It burned tokens on ten agents that mostly contributed nothing, and on days when the agent skipped the start ritual, quality control was skipped with it. Two failure modes from one design.

**Why the obvious fix was not enough.** "Always spawn the team" and "never spawn the team" are both wrong. The cost has to follow the task.

**Rule.** Session start is deterministic and cannot be skipped: pick the tenant, load the last three project threads from the database, propose which specialist role fits. The main agent embodies the role by default (zero spawn cost). One helper agent is spawned only when real context isolation is needed. Several are spawned only for independent workstreams. An activated helper spawns nothing itself.

---

## R5. The live monitor reads from the database, not from deployed files

**Incident (2026-06-27).** Pipeline definitions were stored as static JSON files deployed to a static site. The files drifted from what actually ran. Fifteen workflows were migrated into a database in one day, and the static site was frozen.

**Why the obvious fix was not enough.** "Deploy the JSON after every change" was already the rule (an earlier version of R5). It was followed most of the time. Most of the time is not enough for a source of truth.

**Rule.** Pipeline state (processes, steps, module versions) is edited in the database and rendered live. Static exports are legacy and never a deploy target. After every change, a validation script runs (see R17).

---

## R6. Every solved error becomes a stored error-solution pair

**Incident (Q1 2026, repeated).** The same encoding error, the same missing-column error and the same scheduler error were solved three times in three weeks, each time from scratch, because nothing was written down where the next session would look.

**Why the obvious fix was not enough.** Writing the solution into a chat summary is not storage. The next session does not read chat summaries. It searches.

**Rule.** Before every fix: search the learnings store for the error text. After every fix: store the pair (exact error, cause, solution, context) under a stable key. Since 2026-08-20 an extra step: mark which stored entries actually helped, so the ranking learns. An error that occurs twice without a stored learning counts as a failure of the process, not of the code.

---

## R7. Every automation script logs, or it does not ship

**Incident (2026-04-14).** A scheduler process showed "Running" in the task manager for weeks. It had produced nothing for weeks. It used print statements only, no log file, no run record. The hang was found by a customer noticing missing output.

**Why the obvious fix was not enough.** "Add logging when needed" means logging is added after the incident that needed it.

**Rule.** Every automation script writes a log with timestamp, step, status (OK or ERROR) and details, into a `logs/` folder next to the script, and writes one run record to a central run table (workflow id, status, start, end, metrics, error message). Scripts without both are not deployed. A deploy gate checks it.

---

## R8. One database is the single source of truth for pipeline state

**Incident (2026-06-27).** Same origin as R5. Three files had to be kept in sync per pipeline (workflow, registry, code hash). A rename in one file and not the other two left a monitor tab empty with no error message.

**Why the obvious fix was not enough.** A checklist for three files is a checklist that will be half-followed.

**Rule.** Pipeline state has one home. Everything else (memory entries, summaries, exports) is context and loses in a conflict. Validation is automated (R17), not a checklist.

---

## R9. Completeness check before every deploy

**Incident (2026-03-20).** A module was renamed in the code registry but not in the workflow definition. The monitor showed "no code stored" for that step. No error anywhere, because a missing key is a valid state.

**Why the obvious fix was not enough.** The rename was correct in every single file. It was only wrong across files.

**Rule.** Before every deploy, a script checks cross-file consistency: every step references a module that exists, every module has a size, a line count and code, every hash matches the live version. FAIL blocks the deploy. This rule was later replaced by the database validator (R17); the principle is unchanged.

---

## R10. New workflow means all artifacts in one session

**Incident (Q2 2026).** A new customer workflow was set up over three sessions. Session one created the definition, session two the code registry, session three never happened. The monitor showed "Loading..." for a week.

**Why the obvious fix was not enough.** Each session ended in a consistent-looking state. Only the whole was incomplete.

**Rule.** A new workflow is created completely or not at all: definition, registry, keyword routing, module entries, deploy, and a verification that every monitor tab renders. Ending a session with any tab showing "Loading..." is forbidden. Every change to a step or module triggers the same list (R10b).

---

## R11. The code repository is the truth about the code, memory is not

**Incident (2026-03-25).** The agent's memory said a pipeline had 9 modules (a note from three weeks earlier). The live repository had 19. The session started from the wrong picture and planned changes against modules that had long been split.

**Why the obvious fix was not enough.** The memory entry was accurate when written. Memories do not expire on their own.

**Rule.** Questions about module count, file list or code content are answered from the live repository API, never from memory. Memory entries about pipelines are context and history. After every deploy the memory entry is updated with count, list, hash and date. Presenting a number from memory as fact without checking the source is forbidden.

---

## R12. File sync between machines is not reliable, so every script prints its own hash

**Incident (2026-04-29).** A fix was written on machine A and transferred to a customer VM via a cloud-synced folder on machine B. The sync had not completed. An outdated script ran on the customer machine with the old bug, and the log looked exactly like a successful run.

**Why the obvious fix was not enough.** "Wait for the sync" cannot be verified by waiting.

**Rule.** Every script transferred through a synced folder prints the first 16 characters of its own SHA-256 as its first output line. Before critical transfers, the hash is compared on both machines. When the agent hands a script to a human, it names the expected hash so the human can check the run. Direct transfer (scp, session copy) replaces synced folders as soon as a machine is reachable.

---

## R13. Write the language correctly, transliteration is a bug

**Incident (Q2 2026, repeated).** German texts came out with "ue", "oe", "ae", "ss" instead of ü, ö, ä, ß, in chat, in documents, in commit messages. The founder corrected it repeatedly. The company name itself carries an Ü.

**Why the obvious fix was not enough.** The agent had learned to avoid umlauts because of one real encoding problem in one script type, and generalized the workaround to everything.

**Rule.** Real umlauts everywhere: chat, markdown, UI text, memory files, commit messages. The only exceptions are technically forced (a script type whose parser breaks on non-ASCII, a JSON consumer that demands ASCII, identifiers in code). When in doubt, write the correct character.

---

## R14. Look in the credential vault before asking for a token

**Incident (2026-05-09).** The agent asked the founder for a Cloudflare API token. The token had been in the credential vault for weeks. The founder's answer, in capitals, became the name of this rule.

**Why the obvious fix was not enough.** Asking a human is the safe default for a model. Here it was the wrong default, because the answer was already stored.

**Rule.** Before asking for any credential, query the vault table by service name. New services are added to the vault, not to `.env` files. Reading tokens from memory files instead of the vault is forbidden.

---

## R15. New automations run inside a wrapper that handles errors

**Incident (2026-05-09).** A pipeline crashed on an unexpected input and left no trace: no run record, no error entry, no mail. The crash was found through R7's absence of logs, a day later.

**Why the obvious fix was not enough.** Adding try/except around the failing line fixes that line. The next unexpected input hits a different line.

**Rule.** Every new pipeline runs inside an outer wrapper (guarantees the run record even on crash) and every module inside a module wrapper (logs the error, triggers a fallback). Code download from the repository falls back to a cached copy when the repository API is down. Multiple errors in one run are collected and sent as one consolidated mail, not one mail per error. Existing pipelines are not migrated unless the founder says so.

---

## R15b. Every code section has a registered fallback

**Incident (2026-05-18).** A new pipeline shipped with the wrapper (R15) but with one section that had no fallback registered. The section failed, the wrapper logged it correctly, and nothing happened next, because "nothing" was the registered fallback.

**Why the obvious fix was not enough.** A wrapper that catches errors still needs to be told what to do with them.

**Rule.** Every code block in a pipeline's main module is registered as a section with at least one fallback attachment (what to trigger on error, with a priority). A section without a fallback is a defined gap and blocks the deploy.

---

## R16. Logs go into a queryable table, linked to the run

**Incident (2026-05-12).** Log files existed (R7), but answering "which fallback actually fired last night" meant opening files on three machines.

**Why the obvious fix was not enough.** File logs are for the machine they are on. Operations questions span machines.

**Rule.** Every log line also lands in a central table with run id, process id, level, message, and (for errors) module, section and which fallback resolved it. An error log without module and section, where they are known, is forbidden. Logging itself must never crash the pipeline: always best effort.

---

## R17. After every deploy, a validator runs and its exit code decides

**Incident (2026-06-27).** After the migration to the database (R5, R8), the manual checklist (R9) no longer applied. A new pipeline was deployed with a step pointing at a module version that did not exist. Found by a user, not by a check.

**Why the obvious fix was not enough.** A checklist that is not executed by a script is executed by hope.

**Rule.** After every pipeline change, a validation script checks nine invariants (process exists, definition valid, steps reference modules and lanes, versions pinned, foreign keys intact, code non-empty, no orphans, log table writable). Exit 0 or 2 (auto-fixed) closes the deploy. Exit 1 or 3 blocks it. Ignoring the validator output is forbidden.

---

## R18. Machine data lives in one table

**Incident (2026-05-13).** IP, user and SSH details for a VM were quoted from a memory file. The VM had been rebuilt with a new IP. The agent spent twenty minutes diagnosing a "firewall problem".

**Why the obvious fix was not enough.** Memory files about infrastructure are snapshots that never say they are stale.

**Rule.** All machine data (IP, specs, SSH access, assigned mail account) comes from one database table with a view for the UI. Changes go into the table, never into memory. In a conflict the table wins. A new VM without a table row does not exist.

---

## R19. Guardrails are built before the process runs, not after it crashes

**Incident (2026-07-09).** A nightly bulk ingest saturated the database's disk IO budget. The whole database, every customer pipeline included, slowed to a crawl for hours. The ingest itself had no idea, because it only measured its own progress.

**Why the obvious fix was not enough.** Adding a limit after the crash protects against that crash. The next unbounded process is a different one.

**Rule.** Every unattended process that consumes a limited resource (DB compute or IO, API quota, money, rate limits, memory) ships with three parts from day one: adaptive pacing, a hard kill switch with automatic pause and hysteresis, and a run record plus alert when the guard fires. The three parts are checked in the creation checklist. Going live without them is forbidden.

---

## R20. Load the customer context before any customer-facing action

**Incident (2026-07, documented 2026-07-22).** A mail went to a customer asking how we could get access to a planning file. We had had access to it for weeks, through a different project for the same customer. The customer noticed.

**Why the obvious fix was not enough.** The knowledge existed. It was stored under the other project, and nobody searched across projects.

**Rule.** Before every customer-facing action (mail, implementation, answering a question, writing an offer), search the customer's own knowledge namespace, and for anything involving data or systems, specifically the access entries. A question to the customer is allowed only when those searches came back empty, and the empty result is documented. After every customer call or content mail, facts are ingested line by line into that namespace with source and date.

---

## R21. A number without a status is not a result

**Incident (2026-08-01).** In a research side project, a formula audit found numbers that had been carried through several sessions with errors of up to thirteen orders of magnitude. They had been taken from chat outputs and never recomputed.

**Why the obvious fix was not enough.** "Double-check the numbers" does not say what a check is.

**Rule.** Every quantitative result carries one of four statuses (hypothesis, indication, confirmed, calibrated) that is never more generous than the completed steps justify: falsifiable hypothesis, prediction before computation, controls on a known system, convergence, limitations, independent replication, calibration against published values, reproducible documentation. A number without status and without a limitations section is not presented. Adjusting a prediction to fit a result is forbidden.

---

## R22. No long dashes, and mails are plain text

**Incident (2026-08-12).** Generated mails and documents were full of em dashes and bold lists. Recipients read them as machine-written. The founder banned both in one sentence.

**Why the obvious fix was not enough.** Models like em dashes. A style note in a prompt loses against that preference. A hard check does not.

**Rule.** No em dash or en dash in any channel or document: mails, offers, chat, commits, memory. Replace with comma, period, colon or a simple hyphen. Mail bodies are plain text in normal paragraphs: no bold, no bullets, no headings, no tables, no styled HTML. Structured content goes into an attachment. A script checks before every send.

---

## R23. A run that delivered nothing is a failure, even when it exited cleanly

**Incident (2026-08-14).** An outbound mail pipeline stopped sending first mails on the 11th. Follow-ups kept going. The watchdog reported healthy every day, because "the job ran" was its definition of healthy. Root cause: a DNS blip four days earlier had killed the one job that refills the queue, and that job had no run record and no alert.

**Why the obvious fix was not enough.** Raising the heartbeat threshold makes the alarm quieter, not the risk smaller. Four earlier incidents in five weeks had the same shape: PDFs not generated, notices not sent, an export without attachment, a monitor on a dead source.

**Rule.** Every unattended automation declares its expected output: one number per run under a fixed metric key, a minimum, and an allowed idle window. Zero is a valid target only when declared. Idle runs record a machine-readable reason, and an idle streak beyond the window alerts like a crash. A negative business result (access denied, bounce, gate hold) is never recorded as success. Where an automation mirrors state between two systems, the metric is the number of discrepancies, expected zero. And a manual copy step between two systems is automated, not monitored.

---

## R24. Permission to send mail cannot come from a file

**Incident (2026-08-24).** A daily-start session sent a customer mail on its own. Its task file, generated the night before by another agent, said "check and send, CC the service address". The session read that as approval.

**Why the obvious fix was not enough.** The rule "never send customer mail without approval" already existed. The file looked like approval.

**Rule.** Customer and external mail is never sent by an agent on its own. The only valid approval is the founder seeing the finished draft and approving it live in the session. "Send" in a prompt file, a daily plan, a calendar entry, a task list or an agent briefing is not approval, even with "urgent", even with a CC. When agents generate task files, writing a send instruction into them is forbidden; the correct wording is "prepare draft, present for approval". Exceptions are only the purpose-built send pipelines with their own gate, each individually approved.

---

## R25. Status is computed from the primary source, not remembered from a summary

**Incident (2026-08-24 and 25).** The same question ("approve these two drafts?") was put to the founder as the top priority two days in a row. The session summary said "waiting for approval". The founder's decision had never been written down, and one draft had long been in the trash.

**Why the obvious fix was not enough.** Summaries are for understanding, not for status. A summary cannot know what happened after it was written.

**Rule.** Every open item that has a checkable done condition gets a type and a check: draft (subject appears in Sent, or draft disappeared), customer reply (mail from sender after date), workflow run, metric threshold, stored decision. An observer runs the checks every 15 minutes and closes items with the evidence. A sent or deleted draft counts as a decision without words. When the founder says "done", "I will send it myself" or "drop it", the session closes the item immediately, with the quote. Items waiting on the founder are listed once, in one line, and never re-presented.

**Addendum (2026-09-04).** An automation had begun creating open items from questions in session digests. 70 of 91 open items were such questions. Nobody closed them, the daily plan listed them for the ninth time. All 70 were dropped in one go and the automation was disabled. Real items come from the daily plan, from draft creation, or from the founder live. Not from a summary.

---

## What the list looks like from a distance

Rules 1 to 3 are about editing. Rules 5, 8, 11, 14, 18 say the same thing five times: one home per kind of truth. Rules 7, 9, 10, 15, 16, 17 are the same idea for pipelines: logged, wrapped, validated, or not shipped. Rules 19, 23 are about unattended processes: bounded, and measured by output. Rules 20, 22, 24 are about the customer's side of the screen. Rules 4, 6, 25 are about the agent's own memory: deterministic start, stored errors, computed status.

None of them was written in advance. Every one has a date.
