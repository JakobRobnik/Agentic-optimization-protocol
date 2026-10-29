# Agentic Optimization Protocol (AOP)

## Purpose

AOP is a reusable protocol for developing algorithms with coding agents and evaluating them on public and hidden examples. A user works with an examiner agent to prepare the experiment, starts separate applicant sessions to develop solutions, and then returns to the examiner to evaluate frozen submissions. GitHub carries the public specification, evaluation code, development examples, and submitted solutions.

This document defines the workflow and information boundaries. A separate **problem specification** defines the scientific or engineering task, submission interface, allowable operations, benchmarks, score, failure rules, and reporting requirements. Use this protocol unchanged for new problems; put problem-specific decisions and run settings in that specification and the experiment manifest.

## 1. Roles and responsibilities

- **User:** works with the examiner, approves the public experiment, starts independent applicant chats, may discuss development with applicants, approves final submissions, and returns to the examiner for evaluation.
- **Examiner:** implements and validates the evaluator with the user; prepares public and private examples; writes the public applicant instructions; reviews submissions for compliance; runs the evaluator and reports results.
- **Applicants:** read the public instructions, develop solutions in their own restricted sessions, run public and self-created examples, discuss their work with the user, and submit their final code on separate GitHub branches after user approval.
- **Evaluator:** executable code that produces the score according to the frozen problem specification. The examiner interprets results but must not substitute subjective judgment for the specified score.

The user starts the chats. The examiner does not spawn applicant subagents, create applicant chats, or assume that it can inspect their conversations. Sessions may use different coding-agent products or models; record the choices. Any further delegation must be explicitly allowed by the experiment configuration and preserve the same access boundaries.

## 2. Directory and information boundaries

Use distinct local roots:

```text
experiment/
    examiner/                       # examiner session's project boundary
        public/                     # Git repository exchanged through GitHub
            evaluator/              # functions used to compute scores
            examples/               # public development examples
            instructions/           # applicant task and execution instructions
            tests/                  # public evaluator/interface checks
        private/                    # never part of the public Git repository
            examples/               # additional hidden evaluation examples
            references/             # hidden answers and calibration products
            manifests/              # private test selection and run settings
            results/                # evaluation products and examiner reports
    applicants/
        applicant_01/               # applicant 01 session's project boundary
            repo/                   # independent clone of examiner/public/
            work/
        applicant_02/               # applicant 02 session's project boundary
            repo/
            work/
```

The examiner has project-file access only to `examiner/`; each applicant has project-file access only to its own `applicants/applicant_ID/` folder, not the whole `applicants/` parent. Each session may access the public internet and necessary runtime dependencies and session-private temporary storage. Exchange artifacts through GitHub, not by granting the examiner access to applicant folders or vice versa.

These restrictions must be enforced by the runtime or operating system for **reads as well as writes**. Merely choosing a working directory, using a Git worktree, or instructing an agent not to inspect another folder is insufficient. Configure and verify the actual controls of the chosen product before development. If those controls cannot be established, disclose the limitation and do not represent the experiment as isolated.

Start applicant chats with fresh contexts containing only public instructions and approved public material. Do not copy, fork, or summarize the examiner's conversation into them. Disable tools, connectors, shared memories, transcript access, credentials, and authenticated browser access that could retrieve private examiner information. Public internet access must not create an alternate route to private/local services. Applicants must not be able to broaden their permissions themselves.

Verify access with harmless dummy files and conversation markers under the intended session configuration. Public reads/writes should succeed; private reads and conversation retrieval should fail or be unavailable. Record checks and limitations. Separate chats prevent history inheritance only when created fresh; they do not automatically remove other retrieval paths.

For the Claude desktop app (Code tab), a tested implementation is in [Appendix A](#appendix-a--isolation-in-the-claude-desktop-app-code-tab).

Only `examiner/public/` is pushed as the public repository. Keep private material outside that repository from the beginning, including outside its Git history. `.gitignore` alone is not an access boundary. Check that examples, tests, logs, notebook outputs, configuration, commit messages, and dependencies do not expose hidden examples or examiner conversations. Here, public means available to applicants; the GitHub repository may itself be publicly listed or access-controlled.

## 3. Phase A — prepare the experiment with the examiner

Start the examiner chat with this protocol and the problem specification. Work incrementally with the user to build:

1. A public evaluator implementing the submission interface, scoring functions, resource accounting, and failure handling.
2. Public examples that applicants can use for training, development, and their own validation splits.
3. Additional private examples for held-out evaluation, with any required private reference answers.
4. Public tests, dependency/environment instructions, and a minimal valid example submission.
5. Public applicant instructions describing the objective, allowed operations, submission format, development resources, stopping conditions, and how to run the evaluator.

Use the same scoring implementation for public and private examples; only the examples and their associated reference data differ. Keep the evaluator independent of private model definitions and private data. The public scoring rules must be complete even though the private examples are hidden.

Validate the implementation, references, baseline if applicable, and representative failure cases before applicant development. Use short pilots to estimate resources. Do not invent performance or runtime measurements. Record unresolved assumptions for discussion with the user.

Prepare an experiment manifest containing the problem-specification version, dependency versions, benchmark and reference identifiers, initialization/seed rules, evaluator configuration, applicant count, submission count, permitted collaboration, resource allowances, and required reporting. Numerical budgets belong in this manifest, not in AOP itself.

## 4. Phase B — user approval, freeze, and publication

When the user agrees that the prepared experiment is ready:

1. Freeze the evaluator, public instructions, and benchmark selection. Record exact versions and immutable identifiers for public and private products.
2. Push `examiner/public/` to the agreed GitHub repository. Use a concrete repository URL supplied or approved by the user; do not invent one.
3. Record the exact **protocol commit**: the public repository revision that all applicants must use.
4. Prepare a public starting prompt giving the repository URL, protocol commit, applicant ID, submission branch/path, resource allowance, and instructions to read the repository's applicant brief.

Do not publish the examiner's conversation or hidden examples. Generate the applicant brief from the public contract, not by forwarding the private problem-planning conversation. A problem specification may contain examiner-only benchmark plans; derive a public brief rather than assuming the whole specification is safe to publish.

If the evaluator or rules change later, document the change and apply it consistently to all affected applicants. Do not silently change the scoring system after seeing submissions or hidden results.

## 5. Phase C — user starts applicants and development proceeds

The user starts a separate restricted session for each applicant and provides the public starting prompt. The repository copy is placed in the applicant's `repo/` folder before it starts (a tarball of the protocol commit, see Appendix A; `git clone` does not work inside the applicant sandbox), so the applicant reads the instructions and starts work without cloning. Later pulls must not silently switch to a changed evaluator.

Applicants may:

- Discuss their ideas, implementation, and public results with the user.
- Run the public examples and create additional development examples.
- Research on the public internet.
- Iterate within the agreed development and submission budgets.

The user and examiner must not supply hidden examples, private results, or hints derived from private tests during this phase. General rule clarifications should be recorded publicly and distributed consistently. Record material user guidance so that it is clear whether results came from autonomous or user-assisted development. Different research starting directions are allowed when documented.

Public feedback comes from the public evaluator. Private tests are not an iterative debugging service. The examiner need not remain running while applicants work. The user coordinates progress and continuation; there is no automatic forwarding of messages between chats.

Independence and collaboration are experiment settings. For independent attempts, applicants must not access one another's implementations before freezing their own. **Branches in one GitHub repository are not private from its readers.** Preserve independence by delaying candidate pushes until all candidates are frozen, or by using separately access-controlled repositories until final collection. If early visible sharing is allowed, record the resulting work as collaborative rather than independent.

## 6. Phase D — approve and publish submissions

Each applicant produces the required final artifact, for example `submission.py`, at the path specified by the problem contract. Include a short method report describing the approach, public experiments, dependencies, limitations, and available usage records.

When the user approves an applicant's final work, freeze it, record its commit SHA and artifact hash, and push it to the applicant's own branch under the agreed publication schedule. For example, use branch `applicant-01` and path `submissions/applicant_01/submission.py`. Multiple development commits are not additional final submissions; the experiment manifest defines how many frozen submissions are allowed.

Submissions must be reproducible from the declared artifacts and pinned dependencies. Do not rely on uncommitted files, private local resources, or downloads of mutable executable code at evaluation time. Applicants may not replace the trusted evaluator, metric, or reference data. Proposed evaluator fixes go through the user and examiner as common protocol changes.

## 7. Phase E — return to the examiner and evaluate

The user returns to the examiner chat with the final branch and commit references. The examiner retrieves those branches from GitHub, pins the submitted commits, and imports only the allowed submission artifacts. Fetching and extracting files is preferable to merging a candidate branch into the trusted evaluation checkout: retain the evaluator at its approved protocol commit.

Before running candidate code, examine it for compliance with the problem contract: allowed inputs and operations, permitted dependencies, resource limits, undeclared data access, side effects, evaluator modification, and attempts to bypass the scoring rules. Review import/setup behavior as well as the main entry points. Run public interface and smoke checks first. Preserve rejected submissions and explain the violated rule; do not silently repair them using private-test feedback.

Code review is not a security sandbox. Use an execution boundary appropriate to the submission contract so candidate code cannot read examiner files or reference answers outside its permitted numerical inputs, communicate private results externally, or alter the trusted evaluator. Do not assume that a function signature or its author's restricted chat establishes this boundary. Define and validate the mechanism during preparation.

Run each accepted, frozen submission on **both public and private examples**, using the same approved evaluator and the problem-specific rules. Keep failures explicit. Distinguish candidate failures from evaluator/infrastructure failures; document corrections and rerun all affected submissions consistently.

Report to the user in the examiner session:

- Public and private scores, with the problem-specific breakdowns, uncertainty, and failure/exclusion counts.
- Compliance findings and disqualifications.
- Methods, strengths, limitations, and evidence supporting the conclusions.
- Resource consumption where measurable.
- Exact evaluator and submission revisions, example/reference identifiers, run configuration, and reproduction instructions.
- Deviations from the protocol, including any private information or other candidates' code seen during development.

Keep hidden results and reports private unless the user authorizes publication. If private results guide another development round, record a new experiment and use fresh held-out confirmation examples for new generalization claims.

## 8. Traceability

Save attributable run records and immutable submissions. Do not silently reuse cached outputs after changes to data, code, configuration, or references. Commit meaningful implementation updates in the working repository, monitor submitted jobs through completion, and distinguish executed results from hypotheses or untested proposals.

## 9. Companion problem-specification contract

Each new problem document should say that AOP applies, then define only:

1. The task, optimization objective, and scientific scope.
2. The public evaluator, examples, and reference information to construct.
3. The private examples and reference information to construct.
4. The submission interface, valid outputs, permitted operations, and resource accounting.
5. Baselines, evaluation procedure, aggregation, failures, and score formulas.
6. Required validation and result reporting.
7. Initial experiment settings and unresolved technical choices.

It should not redefine AOP's chats, permissions, publication sequence, or GitHub exchange. Domain-specific restrictions refine the common contract; if a proposed change conflicts with AOP, resolve it with the user explicitly rather than silently changing the protocol.


## Appendix A — Isolation in the Claude desktop app (Code tab)

Tested 2026-10-22 with applicants started as separate Code-tab sessions, one per `applicants/applicant_ID/` folder. Re-verify after any app or Claude Code update. Results were reported by the applicant sessions; the examiner confirmed only that the settings files were unchanged.

### A.1 What the controls are

Per-applicant project settings, `applicants/applicant_ID/.claude/settings.json`, read from the session's folder:

- **Bash sandbox (OS-enforced for shell commands only):** `sandbox.enabled`, `allowUnsandboxedCommands: false`, `failIfUnavailable: true`, `autoAllowBashIfSandboxed: true`, and an **explicit `denyRead` list**: every entry beside the path from `~` to the applicant's folder (all other entries of `~` and of each directory on the path, `examiner/`, and the other applicants). A blanket `denyRead ["~/"]` with an `allowRead` exception did not work in the app: the sandboxed shell could not list its own folder (tried `"."`, the absolute folder, the experiment root). The list is static, so files created later beside that path are not covered until the settings are regenerated.
- **Network:** `sandbox.network.allowedDomains` for GitHub, PyPI and its file hosts, arXiv, conda and readthedocs, so setup and research raise no prompts.
- **File tools (Read, Edit, etc.) are not sandboxed.** They follow permission rules: `permissions.blockReadsOutsideWorkingDirectories: true` plus explicit `Read(//abs/path/**)` denies for `examiner/`, each *other* applicant's folder, `~/.claude`, and `~/Library/Application Support/Claude`; and `Edit(./.claude/**)` so applicants cannot edit their own policy. In `Read`/`Edit` rules `//path` is absolute; sandbox paths use ordinary `/abs` and `~/`.
- **Tool surface:** the app exposes many tools that bypass file rules (session/transcript search, browsers incl. `file://`, the user's Chrome, the Terminal panel, settings and directory-grant tools). Deny them with `mcp__<server>__*` and by built-in tool name. `crossSessionInbound: "refuse"` blocks messages from other sessions.
- **Allow** `WebFetch`, `WebSearch`, `Bash` (sandboxed) and `Read`/`Edit`/`Write` on the applicant's own folder so work raises no approval prompt (an unapproved prompt stalls the applicant). Deny rules take precedence, so `Edit(./.claude/**)` and the other denials still hold.
- **Deny `Skill`** as well; user-level skills are otherwise reachable.

### A.2 Implement it

Use `aop_tools/setup_applicant_isolation.py`. The reference copy is at `/Users/robnik/Physics/AOP/aop_tools/setup_applicant_isolation.py`, next to this protocol (`/Users/robnik/Physics/AOP/`); copy it into the new experiment's `examiner/aop_tools/` (the new examiner session needs read access to the AOP folder). Keep the AOP folder outside every experiment root, and regenerate the applicants' settings after creating it so that its path is in their `denyRead` list. From the examiner session:

```sh
python3 aop_tools/setup_applicant_isolation.py --root /abs/path/to/experiment --n 3 \
    --public-repo OWNER/NAME --task "one-line task"
```

`--public-repo` downloads the public repository (branch main, no `.git`) into each `repo/` and writes `examiner/APPLICANT_PROMPT.txt`, the single prompt for every applicant. `git clone` fails inside the sandbox (it cannot write `.git/config` and `.git/hooks`, which are protected), so applicants are told not to use git. Do not loosen that protection.

It creates `applicants/applicant_ID/{repo,work,.claude/settings.json}`, `work/PUBLIC_OK.txt`, `examiner/private/CANARY.txt`, and `canary_applicant_ID_prompt.txt` per applicant, then prints the canary token and a conversation marker. **The examiner must write the marker in its own chat** so transcript retrieval can be tested. Review the denied tool lists before use: enumerate the examiner's own tool list (`mcp__*` servers and built-ins) and add any server or tool that could reach private files, transcripts, other sessions, or the user's signed-in browser (`--extra-deny`). Never use a glob such as `applicant_*` in a deny rule; it matches the applicant's own folder and blocks it.

### A.3 Verify (required before development)

1. The user opens a **fresh** Code session per applicant with `applicants/applicant_ID` as the folder (no fork, no worktree) and pastes that applicant's `canary_applicant_ID_prompt.txt`.
2. Expected: own-folder read and WebFetch succeed; examiner canary, other applicants' folders, `~/.claude`, transcript search, browser `file://`, Chrome, Terminal, `request_directory`, and appends to `.claude/settings.json` are denied.
3. The applicant writes `work/canary_results.txt`. The examiner reads it only with the user's explicit permission, for this test only; otherwise the user pastes it.
4. Confirm the settings files' checksums are unchanged. Record rounds, results, checksums and the limitations below in `examiner/private/isolation_record.md`.
5. Any failure: tighten the settings, re-test the affected applicant, and note the change.

### A.4 Limitations to disclose

- This is a runtime permission control, not an OS boundary. For an OS-level guarantee run applicants under a separate macOS user with `examiner/` set to `chmod 700` (not tested here). Do not describe the experiment as OS-isolated.
- Tool denials were sampled, not exhaustive. Untested: `Skill`, `Monitor`, `SendUserFile`, user-level skills, hooks and `CLAUDE.md` that applicants inherit, and the Write tool on the settings file.
- Applicants can reach the public internet (WebFetch, WebSearch, sandboxed Bash) and can read their own `settings.json`.
- Applicant sessions are cooperative in these tests; adversarial behavior was not tested.
- Re-run the canary tests after changing settings or updating the app.

### A.5 Lessons from the first experiment (to avoid repeating the permission back and forth)

- **One prompt, instructions on GitHub.** Applicants receive the same short prompt; everything else is in the public repository. The setup section of the brief must work end to end inside the sandbox (download as a tarball, `python3 -m venv`, `pip install --no-cache-dir -r requirements.txt`, tests, one public run). A dependency pinned with `git+https` can fail on a dropped clone; the brief should say how to install it from a tarball. `tests/test_sandbox.py` writes to `$HOME` and fails inside the applicant sandbox by design; say so in the brief.
- **Check the repository is public** (an HTTP 404 from the tarball URL means it is private).
- **Test the full applicant setup in one fresh session before the real runs**, not only the denial checks (own-folder listing, venv, pip, tests, public run, the four denials: examiner, other applicant, `~/.ssh`, `~/Documents`). Compare the settings checksums afterwards.
- **Clear leftover canary and test files** from `work/` before the real runs and remove stale `.claude/settings.local.json` files.
- **Applicants inherit the user's global `CLAUDE.md`**; check it for instructions that do not apply to them, and disclose it.
- **Dangerous commands:** applicants run as the same OS user, so `pkill -f` and similar can hit other applicants' processes. State in the brief that processes must be stopped by PID.
- **Decide before the run** the resource allowance (time, compute, per-run timeouts) and whether replicates (identical prompts for some applicants) are wanted; record the prompt variants in the decisions log. Record sandbox, network and permission changes with checksums in `private/isolation_record.md`.
- **Evaluation:** the score needs a baseline measured with fresh seeds and many chains, not only the calibration seeds; the calibrated baseline was optimistic (geometric mean 0.83 of the fresh totals). Use one fresh seed for the baseline and another for the candidates, with at least 100 chains, and report the bootstrap over chains. Run the candidates only on frozen copies pinned by SHA-256 and compliance-screen them first. Do not feed hidden-suite results back to applicants; give feedback from public-suite results only.
- **Applicant guidance:** applicants converge to similar recipes under identical prompts. For diversity, give each a different suggested direction; later rounds may add guided questions. Ask for literature notes explicitly; they were skipped when the instruction was a single sentence.

