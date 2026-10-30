# Run of show - "The second Job"

Audience: VP of Engineering (Software Platforms), Chief Architect (Connected Vehicle & Cloud).
Slot: 10-15 minutes plus questions. Format: hybrid (live session kicked off at minute 0, pre-run session narrated while it works).
Vocabulary: see `CONTEXT.md`. Use those words in the room, especially Job / Processor / Downstream System / Evidence Pack / Gate / Drift.

## One-sentence thesis

The expensive part of moving the remaining Jobs is not writing Rust, it is proving to engineering and privacy reviewers that each Processor is safe. Devin does the Migration inside a Gate that produces that proof, and the same skill does it for every Job after the first.

## Screens you need open before walking in

1. Devin cloud, new-session page, repo `telemetry-migration`, branch `drift/dedup-and-idle` selected.
2. GitHub: the open PR from the pre-run migration session (`demo/charging-session-migrated` into `main`), scrolled to the Evidence Pack in the description.
3. Devin cloud: the completed pre-run migration session, scrolled to the top.
4. Devin cloud: the completed backup run of the live prompt (in case the live one is slow).
5. GitHub: `.agents/skills/migrate-job/SKILL.md` rendered.
6. GitHub: `AGENTS.md` rendered, scrolled to "Gate inputs are read-only".
7. Terminal in a local clone, on `violation/location-in-summary`, with `make gate PROCESSOR=charging-session` typed but not run (fallback for screen 11).
8. `docs/cutover.md` rendered.
9. DeepWiki page for the repo (optional, objection 5).
10. GitHub: the demo PR's checks - the `3. Gate verdict` summary page (per-Processor result table, green) and, one click in, the `2. Gate (charging-session)` summary page showing the same Evidence Pack.
11. GitHub: the draft PR from `violation/location-in-summary` into `main` - its `3. Gate verdict` summary page open, red, with privacy named in the result table.

Do not have a slide deck open. The repo and the sessions are the deck.

## Minute by minute

### 0:00 - 1:00 Frame and kick off the live session

Say: "You have one Job on Rust and the rest still on Spark. The blocker is per-Job proof, not per-Job code. I am going to start Devin on a real task now and come back to it."

Paste the live prompt (below) into screen 1. Start the session. Do not watch it.

### 1:00 - 3:00 The repo in 90 seconds

Screen 2 (GitHub tree). Point at four things only:

- `jobs/` - the Scala/Spark Jobs. `ota-status` (already migrated on `main`), `charging-session` (the one we are doing), and `battery-health` (prepared in PR #22 and on `demo/live-migration`, not yet on `main`).
- `processors/` - the Rust workspace. `ota-status` is the finished exemplar: pure core, file adapter, Kafka adapter behind a feature flag.
- `manifests/` - one YAML per Job and per Processor: what fields are read, derived, emitted, logged, and their classification.
- `evidence/ota-status/` - the Evidence Pack the first Migration produced. "This is what done looks like. The question is how to get the next one for the price of a prompt."

If the Architect asks how Devin learns an undocumented Job: screen 9, DeepWiki, ten seconds.

### 3:00 - 7:00 The pre-run Migration

Screen 5 for 30 seconds: the `migrate-job` skill. Read the step headings, the stop conditions and the acceptance criteria out loud. Step 7 is worth the extra seconds: Devin waits for the `3. Gate verdict` check, reads each red `2. Gate (<name>)` job log (a Gate report counts toward the three attempts like a Drift; no report is an infrastructure failure, re-run once then report), and refreshes the Evidence Pack in the PR description after every push. Acceptance is `3. Gate verdict` green on the PR, not a local run. "Every Job from here on starts with this. It is a file in the repo, so your architects review changes to the process the same way they review code."

Screen 3: the completed session. Scroll, do not read. Narrate the shape:

1. Devin read the Scala and wrote down the business rules it found (point at the characterisation notes it produced).
2. It implemented the Rust Processor against the existing skeleton.
3. It ran the Gate. Show the first red Gate if there was one in the session, then the green one.
4. It generated the Evidence Pack, opened the PR, and waited for `3. Gate verdict` to go green (step 7).

Screen 2: the PR. Three things: diff is confined to `processors/charging-session/` and `manifests/charging-session.processor.yaml`. Evidence Pack in the description. Devin Review findings below it. Then screen 10 for five seconds: `3. Gate verdict` green - the one check `main` requires - its summary page tables each Processor's result, and the `2. Gate (charging-session)` summary shows the same Evidence Pack. "Devin Review is a second agent reviewing the first. A human still merges."

Say the honest sentence once: "The Gate proves equivalence on these fixtures, under these checks. It does not prove production safety. What it does is turn a two-week discovery exercise into a 20-minute read."

### 7:00 - 9:00 The Gate going red for privacy

Screen 11. Say: "Here is a draft PR where the port looks fine and the Harness is green, but the Processor copies the vehicle's last location onto the session summary." `3. Gate verdict` is red, the per-Processor table naming privacy. Open the `2. Gate (charging-session)` summary - the Evidence Pack's finding: field `end_location`, classification `sensitive-location`, not permitted on topic `vehicle.charging.sessions`, Baseline unchanged, disposition `policy violation - requires remediation`. If the checks will not load, fall back to the closed rehearsal PR of the same branch; if GitHub is down entirely, screen 7: run `make gate PROCESSOR=charging-session`, same finding in the terminal.

Say: "Nobody rewrote the policy to make this pass, and the Evidence Pack tells a privacy reviewer exactly which field and which rule. That is what 'no lengthy review per Job' actually means: the reviewer reads this, not the code."

### 9:00 - 11:00 Back to the live session

Screen 1. Three outcomes:

- **Finished**: show the PR diff (only Rust files touched), read Devin's root-cause explanation of both Drifts, show `3. Gate verdict` green on its PR (about 60 s push-to-verdict with a warm cache). Best ending.
- **Mid-flight**: show where it is (usually it has found the dedup key by now; the Vehicle Clock takes longer because the port's wall-clock version looks reasonable until you read the Scala). Say "I ran this exact prompt yesterday" and switch to screen 4 for the ending. Come back to screen 1 at the end of Q&A if it has finished.
- **Stuck or wrong**: say so plainly, switch to screen 4. Do not rescue it live.

Whichever happens, screen 6 for ten seconds: the read-only section of `AGENTS.md`. "This is injected into every session on this repo. It is why the diff only touches Rust, and the Gate checks the same list mechanically."

### 11:00 - 13:00 Repeat for Job #3 and what humans keep

Screen 5 again, one sentence: "Battery-health is next. `@skills:migrate-job battery-health [target-branch]`. Same procedure, different argument. Omit the target for `main`, or use `demo/live-migration` for the three-Job format below."

Then what humans keep, in this order: architectural approval, privacy sign-off on the Evidence Pack, the cutover decision. Screen 8, `docs/cutover.md`, one sentence: "Shadow consumer group, shadow topic, idempotent producer, offset handoff. Documented, not demonstrated, because it depends on your deployment pipeline, which already exists."

### 13:00 - 15:00 Questions

Objection script below.

## Prompts, verbatim

### Live prompt (screen 1, branch `drift/dedup-and-idle`)

```
The migration gate for the charging-session processor is red on this branch.

Run `make gate PROCESSOR=charging-session`, find the root cause of every
failure, and fix it in the Rust processor under processors/charging-session/.

Constraints:
- Do not modify anything under harness/, fixtures/, goldens/, manifests/,
  contracts/, jobs/, downstream/, .github/ or schemas/. If you believe one
  of those is wrong, stop and say so.
- The Scala job in jobs/charging-session/ is the source of truth for
  behaviour. Read it before changing the Rust.

When the gate is green, open a PR that explains, for each drift you found:
what the legacy job actually does, what the Rust did instead, and how you
verified the fix.
```

### Pre-run prompt (screen 3, branch `main`)

```
@skills:migrate-job charging-session

Push your work to a new branch named demo/charging-session-migrated
and open the PR from it.
```

The skill body lives at `.agents/skills/migrate-job/SKILL.md` in the demo repo.

## Three-Job live migration format

This is an alternative to the charging-session storyline above. Use the frozen `demo/live-migration` branch at `22bfe3aa090ee716354ff1ae0ee0de1a0a7a27d4`. All three Processors are unfinished there. Only this branch strips the OTA core. `main` keeps its working OTA implementation. Battery-health and the prepared format are in PR #22, not yet on `main`. The platform-preparation PR's protected-path Gate is expected red. Do not bypass it or merge that PR to start the demo.

### Screens and steps

1. Open the repo on `demo/live-migration`, with the Scala source and Rust scaffold visible.
2. Open session 1 on that branch and paste prompt 1. Show the OTA characterisation, Rust change, PR and Evidence Pack.
3. In the same session, paste prompt 2. Show the existing repo skill and any small refinements. This is capture/review of a prepared skill, not a claim to create it from nothing.
4. Open a new supervising session on the same demo branch and paste prompt 3. Show exactly two child-session links, charging-session and battery-health.
5. Show the independent PRs targeting `demo/live-migration`, their Evidence Packs and actual `3. Gate verdict` checks. Never merge any of the three PRs or push to the demo base.

### Prompt 1: simple live migration

Paste into the first session:

```text
Use xlm/telemetry-migration. Prepare a live migration of ota-status only.
Fetch origin/demo/live-migration and make your own feature branch from it.
Do not use main, restore an answer from history, or push to the demo base.
Before editing, confirm the base has the intentional OTA core todo!()
and that charging-session and battery-health are also unfinished.
Stop and report if the starting state differs.

Read CONTEXT.md and AGENTS.md. Read the Scala Job under jobs/ota-status/
as the specification, including validation, DLQ reasons, status aliases,
case handling, clamping, output version, ordering and privacy. Do not infer
semantics from field names. Characterise the rules with Scala line
citations in processors/ota-status/CHARACTERISATION.md. Mark state,
deduplication and timeouts as not applicable if the Job has none.

Implement the Rust pure core and unit tests, reusing telemetry-core and
the existing JSONL wrapper. Compare the characterisation to the Baseline.
Only change processors/ota-status/ and, if necessary, its Processor
Manifest. Never edit the Scala, fixtures, Goldens, schemas, contracts,
Harness, Baseline, downstream checks or Gate workflow to get a pass.

Run the OTA Rust tests, make fmt and make lint. Commit the implementation.
From a clean tree, run:
make gate PROCESSOR=ota-status BASE=origin/demo/live-migration

Read any Drift against the Scala, fix only the Processor, commit and rerun.
Stop after three attempts on the same Drift or if a fix needs protected
inputs. Open a PR targeting demo/live-migration, never main. Include the
latest clean-commit Evidence Pack verbatim, resolved Drifts and residual
uncertainty in the PR description. Keep generated evidence out of commits.
Wait for 3. Gate verdict, fix legitimate Processor failures and report the
PR URL and check status. Do not merge. Do not migrate the other two Jobs.
```

### Prompt 2: capture the procedure as a repo skill

Paste into the completed live-migration session:

```text
Capture the procedure we just followed as the reusable migrate-job skill
at .agents/skills/migrate-job/SKILL.md. The repo already has a prepared
version: compare and refine it, do not claim you created it from nothing.
Show me the resulting file and the smallest changes needed, if any.

Parameterise the Job name and PR target branch. Use main by default and
demo/live-migration explicitly for this demo. Preserve Scala-as-spec,
characterisation, the Baseline/privacy boundary, per-Processor diff scope,
tests, a clean-commit Gate run using the target branch as BASE, Evidence
Pack in the PR description, stop conditions and no automatic merging.

Do not hard-code OTA business rules or weaken any Gate. Keep a matching
existing skill unchanged. Do not append skill edits to the migration PR
or publish another PR without asking. This step is a capture/review, not
another migration. The prepared skill on the demo base is what the two
child sessions will reuse.
```

### Prompt 3: supervising Devin fans out to two Jobs

Paste into a new supervising session:

```text
Supervise two parallel migrations in xlm/telemetry-migration. Do not
implement either migration yourself. Use the child-session tools and
their managing-child-sessions guidance to create exactly two children.
Do not use a workflow or launch an OTA migration.

Fetch origin/demo/live-migration, inspect it, and confirm charging-session
and battery-health both have intentional Rust core todo!() scaffolds.
Read CONTEXT.md, AGENTS.md and .agents/skills/migrate-job/SKILL.md there.
Stop and report if the prepared base or skill is absent or already
migrated. Record the demo base SHA and give the same SHA to both children.

Child 1: migrate-job charging-session demo/live-migration
Child 2: migrate-job battery-health demo/live-migration

Give each child its own session and feature branch starting from that
exact demo base SHA, with the shared target ref origin/demo/live-migration.
Tell each to read and follow the repo skill, substituting its Job name
and target branch. Include the repository, CONTEXT.md and AGENTS.md in each
prompt, and explicitly prohibit pushes to main or the demo base. Each
child must only modify its own Processor and Processor Manifest. No
shared branch, protected inputs, manual Goldens, or policy relaxation.

Each child must characterise its Scala source, implement and test its
Processor, commit it, then run its own Gate with
BASE=origin/demo/live-migration. It must open one PR targeting
demo/live-migration, include the latest clean-commit Evidence Pack and
wait for 3. Gate verdict. Generated evidence belongs in the PR body, not
another commit. Do not merge any PR.

Show both child-session links as soon as they exist. Monitor their
notifications, route legitimate fixes back to the responsible child,
and escalate blockers instead of guessing or weakening the Gate. Finish
with one concise table: Job, child session, PR, actual CI Gate verdict,
Drifts resolved and residual uncertainty. Do not report success until
both PRs have a green 3. Gate verdict, or explicitly report what blocked
them. Do not create more children.
```

### Expectations

- OTA and battery are small stateless examples. Charging is the more involved stateful example.
- Battery clamps SOH to 0..100, uses health thresholds 60 and 80, emits schema version 1 and retains available IDs on DLQ.
- Red Gates before migration are intentional. Each actual migration must pass its own CI Gate. A green verdict is evidence for human review, not approval or a production safety claim.
- Child provisioning, implementation and CI happen live. Completion time is not guaranteed. Preparation alone is not a three-migration rehearsal.
- Evidence Packs stay in PR bodies. The skill takes `<job-name> [target-branch]`, defaults to `main`, and uses `BASE=origin/<target>`. Always supply `demo/live-migration` here.

### Three-Job live format reset

After a practice run, archive the OTA, supervising and child sessions. Close open PRs targeting the demo branch and delete their feature branches:

```sh
gh pr list -R xlm/telemetry-migration --base demo/live-migration --json number --jq '.[].number' | xargs -I{} gh pr close {} -R xlm/telemetry-migration --delete-branch
git ls-remote https://github.com/xlm/telemetry-migration demo/live-migration
```

The ref must print `22bfe3aa090ee716354ff1ae0ee0de1a0a7a27d4`. Only if it moved, restore the frozen base from a telemetry-migration checkout:

```sh
git push --force-with-lease origin 22bfe3aa090ee716354ff1ae0ee0de1a0a7a27d4:refs/heads/demo/live-migration
git ls-remote https://github.com/xlm/telemetry-migration demo/live-migration
```

Only `main` has a ruleset. The demo branch is unprotected, so do not rely on GitHub to prevent an accidental merge or push. Routine retries start new sessions from the frozen base, without resetting it. The conditional restore above is only for a moved base.


## Objection script

Lead with the artefact, then the sentence. Never argue the general case.

| # | Objection | Artefact | Sentence |
|---|---|---|---|
| 1 | "How do we trust a Rust rewrite is behaviourally identical?" | Harness output, `goldens/` generated from the Scala Job, the Drift branch | "The Goldens come from running your Job, not from a spec. The Harness compares per session key, with DLQ and a Downstream System check. The Drift branch shows two schema-valid, wrong ports being caught." |
| 2 | "Devin will just edit the tests until they pass." | `AGENTS.md` (screen 6), live PR diff, Gate path check | "Three layers. `AGENTS.md` is injected into every session and names the read-only paths. The skill repeats it. The Gate diffs the branch against those paths mechanically and goes red if any were touched. If it had edited Goldens, the PR and the Gate would both show it." |
| 3 | "You didn't show it on Kafka. What about exactly-once?" | `processors/*/src/core.rs`, `docs/cutover.md` | "Ordering, dedup and timeout are properties of the core, and the core is tested deterministically, which a live broker cannot do. Transport, offsets and rebalances are the adapter and your pipeline, exercised by Job #1 already. Exactly-once here means Kafka-to-Kafka under transactional produce, and the doc says what that does and does not cover." |
| 4 | "Our privacy team won't accept machine-generated evidence." | Evidence Pack on the PR, violation branch output | "It is not a decision, it is a dossier. Field-level inventory, lineage, Baseline diff, runtime scan, with the residual uncertainties listed. Your reviewer still signs. They just read 20 minutes instead of discovering for two weeks." |
| 5 | "Our real Jobs are far messier than this." | DeepWiki (screen 9), skill stop conditions | "Agreed. That is why the skill's first step is characterisation, not code, and why it has explicit stop conditions. Some Jobs will stop at step one with a findings doc, and that is still a week saved." |
| 6 | "Where does our code run? What does Devin see?" | none | Answer from Cognition's security material: deployment options, data retention, what is and is not used for training. Do not improvise. |
| 7 | "What is the review burden on our senior engineers?" | PR size, Devin Review, Evidence Pack structure | "One PR per Job, confined to one Processor directory, with a second agent's review attached and a structured Evidence Pack. Review becomes reading, not archaeology." |
| 8 | "How many Jobs per week? What does it cost?" | none | Be candid: it depends on Job complexity and how many stop at characterisation. Offer a pilot on two real Jobs as the way to find out. |
| 9 | (unspoken) The stack is not exactly Rust-on-Kubernetes | none | "Nothing here is Rust-specific. The Gate, Manifests and the skill are the method. The target language is a parameter." |
| 10 | "Can the agent rewrite the checker or its own rules?" | Gate path check, `.github/CODEOWNERS` | "The Gate runs the PR's own harness, so it cannot guard itself - two things hold it on `main`. The Path check goes red if a diff touches any Gate input, and CODEOWNERS covers every protected path plus `Makefile`, `rust-toolchain.toml` and the agent's own rules, so required code-owner review puts any change in front of your reviewers. Follow-ups #12 (run the base branch's Gate) and #13 (Path check scope rule) land after the demo." |

## What this demo proves and does not prove

Say these if asked, and never claim more.

Proves:
- A Processor matches its Job on the committed fixtures, per session key, including DLQ and a Downstream System observable.
- Devin can port a stateful Job inside a Gate without touching the Gate's inputs.
- Semantic Drift (schema-valid, wrong values) is caught, not just schema breaks.
- A privacy-relevant change is detected from Manifest diff plus runtime scan, and reported in a form a reviewer can act on.
- The same Gate runs on every PR in GitHub Actions, with `3. Gate verdict` as the one check `main` requires.
- The same skill runs for the next Job.

Does not prove:
- Production safety under load, rebalance, restart or broker failure.
- Exactly-once end to end into non-Kafka sinks.
- That the fixtures cover every behaviour of the real Job.
- That source-level lineage equals runtime data flow (the runtime scan narrows this, it does not close it).
- That any privacy or legal review obligation is removed.

## Failure modes in the room and what to do

| Failure | Action |
|---|---|
| Wi-Fi dies | Terminal only. `make gate` on `main` for ota-status (green), `drift/dedup-and-idle` (red), `violation/location-in-summary` (red), `demo/charging-session-migrated` (green). Narrate the sessions from memory. |
| Live session slow | Screen 4. Say it is yesterday's run of the same prompt. |
| Live session edits a read-only path | It should not, because of `AGENTS.md`, but if it does: show it, say "this is exactly what Devin Review and the Gate's path check are for", switch to screen 4. |
| Draft PR or its checks will not load | The closed rehearsal PR of `violation/location-in-summary`; if GitHub is down entirely, screen 7 and `make gate`. |
| Someone asks to see the Scala run | `make goldens JOB=charging-session` in the terminal. Takes about a minute in Spark local mode. Only if you smoke-tested it that morning. |
| Someone asks to see Kafka | Only if the Redpanda compose was smoke-tested that morning. Otherwise objection 3. |

## Pre-meeting checklist (morning of)

- [ ] `make doctor` prints `doctor: ok` on this laptop.
- [ ] `git fetch`, every demo branch at its recorded SHA (see `docs/build-plan.md`, "Demo states").
- [ ] `make gate PROCESSOR=ota-status` green on `main`.
- [ ] Gate build cache warm on `main`: Actions, Gate, Run workflow on `main`, `Warm the build cache (main only)` green (push-to-verdict is about 60 s warm, 3 min cold).
- [ ] `main` ruleset on: `3. Gate verdict` required, review from Code Owners required.
- [ ] `make gate PROCESSOR=charging-session` red on `drift/dedup-and-idle` on the Harness for fixtures 04 and 09 and on the Downstream System for fixture 09 only, every other step green.
- [ ] `make gate PROCESSOR=charging-session` red on `violation/location-in-summary` on contract and privacy, both naming `end_location`.
- [ ] Draft PR on `violation/location-in-summary` open, `3. Gate verdict` red with privacy named; closed rehearsal PR reachable.
- [ ] Pre-run PR still open, `3. Gate verdict` green, Devin Review findings visible.
- [ ] Backup live-prompt session (screen 4) opens and scrolls.
- [ ] No stray `devin/*` branches or PRs from rehearsals.
- [ ] Three-Job format reset complete: sessions archived, demo-target PRs closed with branch deletion, remote base confirmed at `22bfe3aa090ee716354ff1ae0ee0de1a0a7a27d4`. Restore only if it moved (reset above).
- [ ] `.agents/skills/migrate-job/SKILL.md` and `AGENTS.md` read end to end.
- [ ] Laptop on power, notifications off, terminal font large.
