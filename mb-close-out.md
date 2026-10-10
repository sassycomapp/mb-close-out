---
title: "mb-close-out"
doc-id: "mb-c0e15a53"
updated: "2026-10-10"
---
# mb-close-out

Closes out the current session and its task in one run. Invoked as `/mb-close-out`. Explicit invocation only; never auto-trigger. It executes the existing scripts and skills; it replaces none of them.

## Rules

1. The agent never commits, pushes, runs a Git command that changes anything, or uses credentials. Each such action is a line in the developer's terminal block.
2. The agent never approves a grant. When a read or write is blocked, it hands the developer exactly `mg-grant --path <folder> --count N`, with the count the work needs.
3. A denial is final. It is never retried, reshaped or routed around; it becomes a developer-run line.
4. One approval gate (Step 5). After it, execution only.
5. A terminal block contains only known values. A value that exists only in an earlier block's output goes into a second, fully literal block.
6. Nothing is left open. A step the agent cannot complete becomes a numbered line in the developer's block.
7. Credentials are referenced by variable name only; never printed, logged or copied.

## Step 1 — Session page

Write the session summary as a file, `sessions/<YYYY-MM-DD>-<short-slug>.md`: in the current repository when it is on the commit-all list, otherwise in `C:\mybizz\mybizz-config-docs\sessions\`. Body only; the commit hook adds the frontmatter, and commit all imports the page into GBrain. Never write it with `put_page` or `gbrain capture`: those write directly into a source's folder, which needs a source owner, and no source has one. One screen: what was done and the evidence, the decisions and standards the developer set, quirks, the documents written or changed, what is open. The documents hold the detail; the page says what happened and where to look.

## Step 2 — Candidate facts

- Durable rules, decisions and gotchas only. One claim each. Skip any fact a committed document already states. A session with no facts says so.
- Route each fact: **advisory** (the default: context, pointers, findings that only need to be found again) or **governed** (rules, decisions and procedures that must be trusted later without re-checking). When unclear, ask one focused question at the gate.
- For each governed fact, write one fact file named `<YYYY-MM-DD>-<slug>.md`: in the `facts/` folder of the current repository when it is on the commit-all list, otherwise in `C:\mybizz\mybizz-config-docs\facts\`. Body only; the commit hook adds the frontmatter.
- For each fact file, run `mb-submit-memory` Steps 1–4. Its reports go into the gate message.

## Step 3 — Project documents

When the session worked in one of the four projects (mb-3-cs, mb4ecom, mb5pdlf, pdlf), run `/mb-project-docs` up to its Step 6. Its presentation goes into the gate message. Otherwise skip this step.

## Step 4 — Changed files

Run `bash /mnt/c/mybizz/scripts/commit-all.sh --status`. Flag any change that is not this session's work; the developer decides at the gate whether it is kept.

## Step 5 — The gate

Present once, in one message: the session page title; the candidate facts (subject, fact, routing); the `mb-submit-memory` report for each governed fact; the project-document presentation; the flagged files. Then STOP. The developer may cancel, amend, reword or re-route; the amended set is final. This approval is the approval that `mb-submit-memory` Step 5 and `mb-project-docs` Step 6 require.

## Step 6 — After the approval

1. Advisory facts: save each with `remember`, visibility `world`, provenance naming the session, entity set when the fact has a subject. Keep each returned `fact-NN`.
2. Governed facts: run `mb-submit-memory` Steps 6–9 for each approved fact file, exactly as that skill defines them.
3. Project documents: run `/mb-project-docs` Step 7.

## Step 7 — Terminal block 1 (developer)

Fully literal, in this order:

1. The `AGENTS.md` copy line from Step 6.3, when there is one.
2. Any line produced by Rules 1, 2 or 6.
3. `bash /mnt/c/mybizz/scripts/commit-all.sh "close-out: <topic>"`

The developer pastes the output back.

## Step 8 — Terminal block 2 (developer; only when a governed fact is under review)

One literal promote per memory id reported `under_review` in Step 6.2:

```bash
cd /home/dev-p/memory-governor && set -a && source .env && set +a
curl -s -X POST http://127.0.0.1:8321/memory/<memory-id>/review \
  -H "X-Governor-Credential: $MEMORY_GOVERNOR_WRITER_MANUAL_SUBMISSION" \
  -H "Content-Type: application/json" \
  -d '{"decision": "promote", "reason": "approved at the <date> close-out gate", "reviewer": "dev-p"}'
```

## Step 9 — Verify and report

- Commit all: every repository `committed` or `nothing to commit`, `pushed` or `already up to date`, ending `All done.` Any "the mb-doc-check hook is not installed" warning or BLOCKED repository: STOP; the developer fixes it and runs the same command again.
- Governed facts: `grep "remembered as fact" /tmp/memory-governor-worker.log | tail -10` is the record of what landed and its fact numbers. If recall shows nothing but the worker log confirms the write, the fact exists; never re-submit.
- The governor review queue must be empty before the close-out ends.
- Report: the session page, facts with their numbers, project documents, the commit summary, anything flagged.

## Completion status

DONE; DONE_WITH_REVIEWS_PENDING (list each open memory id); BLOCKED (what stopped it and what was tried). Never DONE with an unverified step.

## Traps

- Recall may not show a saved fact, because agent-side recall does not show other projects' facts. The worker log is the record.
- `under_review` is designed behaviour. Resolve it through Step 8, never by re-submitting with a different confidence.
- An open review item blocks PDLF steps everywhere. The queue must be empty at the end.
