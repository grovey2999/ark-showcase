# How ARK is built

One founder directs. AI agents write and check the code. This page describes that method as it exists in the private repository, with its scale and its limits. It was written on 2026-11-24 from the repository's agent definitions, project instructions, commit history and merge scripts.

## What is and is not AI

The agents are the build crew. They are Claude models run through Claude Code, each with a written job description checked into the repository.

The ARK product makes no calls to any AI service. A search of the application source, scripts and tools found no AI provider, model name or API call. No AI feature is planned in this repository, and none is described here.

## The roles

The repository defines eleven agents. Each has one job, a limited set of tools and a written list of the rulings it must follow. They are named here by what they do.

| Role | What it does | Can it change code? |
|---|---|---|
| Root-cause investigator | Goes first on any change to a shared, widely used file, or when a second fix is about to be started on the same symptom. Reads only. Names the exact file and line behind a symptom and says what a surface patch would miss. | No |
| Builders | Write the code. Page layout, data and migrations, authentication and security, deployment and backups each have a specialist. | Yes, in their own area |
| Test engineer | Writes and triages tests, picks which subset of the suite to run, and reads suite output correctly. | Yes, tests |
| Security specialist | Owns sign-in, sessions, second factors, CSRF, the permission table and rate limits. Asks of each route what happens with a wrong password, a stolen cookie, a replayed token or a role that should not have the grant. | Yes, in its area |
| Migrations specialist | Owns schema changes. Every change is a new, idempotent version. A module is rows of data, not a copied page. | Yes, in its area |
| UI builder | Builds screens to a written design lock. | Yes, in its area |
| UI judge | Looks at every screen at phone and desktop widths before the founder does. It cannot write code and cannot edit the source. Its output is a list of problems, and it can reject a screen. | No |
| Refuter | Re-runs a claimed fix. It is handed the founder's original request word for word, never another agent's summary of it, and it must run the thing, not read the diff. It reports how many things it checked. | No |
| Rehearsal harness specialist | Maintains a scripted browser run that plays each role through the app and records before and after screenshots and the database change for every button. | Yes, in its area |
| Manual writer | After any change to a signed-in screen, regenerates the user manual by rendering the real page as the right role, so the manual changes with every screen. | Yes, manual |
| Release assembler | Combines finished feature branches into one stack branch, runs the merge and release-note checks, and prepares the final merge command. It does not run it. | Yes, on the stack branch only |

## How a change moves

1. The founder decides what to build and states it in plain language. Rulings are recorded in an append-only file that every agent is told to read first.
2. If the change touches a shared file, the root-cause investigator reads first and reports.
3. Builders work on their own branch in their own working copy of the repository. Each file has one owner at a time, so two agents are not editing the same file. Each builder commits its work in steps and pushes its branch.
4. A refuter re-runs every claimed fix against the founder's original wording and reports the number of things it examined. "Zero checked" must never look like "nothing wrong."
5. The test engineer runs the suite. One full suite runs at a time. A run counts as a pass only with an exit code of zero and no failing test markers.
6. The UI judge reviews every changed screen. The manual writer updates the manual.
7. The release assembler builds the stack branch and checks it. A git hook rejects a merge to the main line unless the message carries a session link and a suite report path. Another check caps the size of a branch and requires a release note for any commit that changes a screen.
8. The founder approves. Only the founder's own session merges to the main line. The assembler prepares the command and stops.

## Scale

Counted from the repository on 2026-11-24, whose last commit is dated 2026-10-31.

- 4,606 commits across all branches, from 2026-09-01 to 2026-10-31. 2,701 are on the main line.
- Of those 2,701 main-line commits, 2,322 carry a co-author trailer naming the AI model that wrote it, and 2,196 carry a link to the agent session that produced it.
- 1,142 remote branches. Branch names show feature branches, stack branches, snapshots and rescue branches.
- 777 test files, about 185,000 lines of tests against about 176,000 lines of application source.
- 122 schema migrations and 474 route registrations.

## What this method is good at

- Volume with structure. One person produced a multi-district system with a control plane, payments, messaging and a 777-file test suite in about two months, because agents worked in parallel on separate branches.
- Checks that do not depend on the author. The refuter, the judge and the test suite are separate from the builder that made the change.
- Repeatable procedure. The rulings, branch limits, merge rules and release-note rules are written down and enforced by scripts, not remembered.
- A manual and release notes that track the product, because producing them is a named job with a check behind it.

## Where it is limited

- The founder is one person approving the work of many agents. The method places the founder's attention, not an engineering team's, at the final gate. This page does not claim line by line human review of every change.
- Agents can be wrong together. Several agents agreeing that a fix works is not evidence that it works. This is why the refuter must run the thing and not read the diff.
- A passing suite is only as good as what it covers. The count of 777 files says nothing about coverage or pass rate, and no pass rate is claimed here.
- The history is noisy. A thousand branches and a heavy agent-written log are harder to read than a conventional history.
- The product is young. Its first commit is dated 2026-09-01, and it has had less time in front of real use than its size suggests.
- No outside security review is claimed. See [SECURITY.md](SECURITY.md) for the list of what is not proven.

## Related

[README.md](README.md) summarizes the product. [ARCHITECTURE.md](ARCHITECTURE.md) describes the system the agents built.
