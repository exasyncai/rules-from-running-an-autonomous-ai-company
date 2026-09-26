# Rules from running an autonomous AI company

Twenty-five rules for AI agents that work unsupervised, each one paid for by a real incident. Each entry names the incident, the date, and the rule that came out of it.

Exasync is a one-person company in Tallinn that builds automations for freight forwarders and runs its own operations with AI agents: session start, deployments, monitoring, mail drafts, daily planning. The agents work unsupervised for hours. That only works because of a rulebook that grew one failure at a time.

This repository is that rulebook, stripped of everything customer-specific. What is left is the pattern: what went wrong, why the obvious fix was not enough, and what the agents are required to do now.

## Install

Nothing to install. Copy the rules you need into the instruction file your agent reads at session start (`CLAUDE.md`, `.cursorrules`, `AGENTS.md` or similar), or clone the repository and link it:

```
git clone https://github.com/exasyncai/rules-from-running-an-autonomous-ai-company.git
```

## What you have after 2 minutes

- `RULES.md`: all 25 rules, R1 to R25, in the order they were numbered. Each rule has three parts: **Incident** (what happened, with date), **Why the obvious fix failed**, **Rule** (what is required now, what is forbidden). Read the incidents first; pick the rules whose incident you recognise.
- `posts/linkedin-25-posts.md`: one short post per rule, written for LinkedIn. Use them as a summary or as a checklist for a team review.
- Five themes that group the rules, listed below, so you can start with the one that hurts most.

The rules are written for AI agents that read a project instruction file at session start (Claude Code, Cursor, Codex and similar). They work just as well as a review checklist for humans.

## The five themes

1. **Truth lives in one place.** Code in the Gist, pipeline state in the database, machine data in one table, credentials in one vault. Summaries and memories are context, never proof. (R5, R8, R11, R14, R18, R25)
2. **Change only what is broken.** Minimal edits, no full-backup restores, hash checks on every transfer. (R1, R2, R3, R12)
3. **A run that produced nothing is a failure.** Logging, run records, validation after every deploy, and an output metric instead of a heartbeat. (R7, R9, R10, R15, R16, R17, R23)
4. **Learn once, apply forever.** Every solved error becomes a stored pair. Every decision becomes a precedent. (R4, R6, R25)
5. **Some things an agent never does alone.** Sending mail, spending money, running unbounded jobs, writing a number without its status. (R19, R20, R21, R22, R24)

## Limits, stated honestly

- These rules come from one company with one stack: Python pipelines, a Postgres database, Claude Code agents, customers in road logistics. Some rules name that stack. Translate before you adopt.
- They are rules, not a framework. Nothing here enforces itself. An agent follows them because its instruction file says so and because a human checks. Where we built tooling to enforce a rule, the rule says so.
- Dates are the dates an incident was documented, not the day it started. Customer names, internal system names and business data are removed, which sometimes makes an incident sound more abstract than it was.
- Twenty-five is where the count stood when this was published. The list grows with the next incident.

## What this is not

It is not a framework and not a product. Take the rules that match a failure you already had, and ignore the rest until you have the incident that makes them necessary. That is how they were written.

## Contributing

Corrections, translations and your own incident-born rules are welcome as issues or pull requests. A rule proposal needs the incident behind it. Security reports go to the address in [SECURITY.md](SECURITY.md).

## License

MIT, see `LICENSE`.

## Author

Bodo Buschick, Exasync OÜ, Tallinn. Building office automation for freight forwarders since 2021, before that on the operational side at two large logistics companies.

More at [exasync.ai](https://exasync.ai).
