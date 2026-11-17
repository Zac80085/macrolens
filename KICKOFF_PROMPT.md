# Kickoff prompt for Claude Code

Paste everything inside the box below as your first message to Claude Code, from the `macrolens` folder.

```text
You are building Macrolens, my personal calorie tracker PWA, from start to finish with as little input from me as possible.

READ FIRST, IN THIS ORDER
1. CLAUDE.md
2. docs/PLAN.md (the full plan: requirements, design, errors, tests, phases, blockers)
3. docs/ENGINEERING_QA_PLAYBOOK.md v1.0.0 (quality rules for every artifact)
If docs/STATUS.md exists, read it and resume from its "Next step" instead of starting again.

YOUR ROLES, IN ORDER
- Requirements analyst (Phase 1): verify and complete the requirements, then lock them.
- Designer (Phase 2): lock the design and write the ADRs.
- Developer (Phases 3 to 6): build test-first.
- Tester (Phases 4 to 8): unit, integration, component, end-to-end, accessibility, offline, performance and live smoke tests.
- Release manager (Phases 8 and 9): run the gates, deploy to GitHub Pages, run my acceptance test, act on my feedback, release v1.0.0.

HOW TO WORK
- Follow the phases P0 to P9 in docs/PLAN.md Section 13, in order. Do not start a phase until the previous phase's exit criteria pass.
- Create a task list with one item per phase and keep it updated.
- At the end of every phase: commit, push, and append one line to docs/STATUS.md with the phase, date, gate results and next step.
- Use the IDs from docs/PLAN.md (REQ, NFR, ASM, OQ, HLD, LLD, ADR, FM, TC, error codes) everywhere: in docs, code comments where useful, test names and commit messages.
- At Phases 1, 2 and 8, start a separate reviewer subagent that did not write the artifact. Give it the artifact, docs/PLAN.md and the playbook, and ask it to list defects by playbook catalogue ID. Fix every defect or record a one-line reason in docs/STATUS.md.
- Before each phase's commit, run the three playbook gates (Traceability, Syntax & Command, Precision) on what that phase produced. Never report PASS for a check you did not run; report FAIL with the reason.
- Accuracy of nutrient data is the top priority. When a choice trades accuracy for convenience, choose accuracy and note it in docs/STATUS.md.
- Verify facts that may have changed since the plan was written (package versions, SDK option names, API fields, Gemini model IDs) by reading current docs or running the code, and record what you found in docs/spikes.md.

WHEN TO STOP AND ASK ME
Only for blockers B-01 to B-07 in docs/PLAN.md Section 13. For anything else: pick the option that best serves accuracy, then the plan, record the choice in docs/STATUS.md, and keep going. When you stop for a blocker, tell me in at most 5 lines exactly what to do, then wait.

DONE MEANS
- All three gates PASS on the final build, with the gate report in docs/STATUS.md.
- npm run gates exits 0 and CI is green on main.
- The app is live on GitHub Pages and the live smoke tests TC-090 to TC-093 pass.
- I have run docs/UAT.md on my iPhone, every docs/FEEDBACK.md item has a status, and I have written "accepted".
- v1.0.0 is tagged and docs/USER_GUIDE.md exists.

Start with Phase 0 now. Your first message to me should be at most 5 lines: what you are checking in Phase 0, and any blocker you already see.
```

## Before you paste it (one-time setup, about 15 minutes)

1. Install the current Node.js LTS, Git and the GitHub CLI (`gh`) on your Mac, then run `gh auth login`.
2. Create a folder named `macrolens`, put these files in it with this layout, and run `git init` inside it:
   ```text
   macrolens/
     CLAUDE.md
     KICKOFF_PROMPT.md
     docs/
       PLAN.md
       ENGINEERING_QA_PLAYBOOK.md
   ```
3. Get your keys and keep them ready (Claude Code will ask for them as blocker B-03):
   - Gemini API key: Google AI Studio → Get API key.
   - USDA FoodData Central key: the FDC API key sign-up page (free).
4. Start Claude Code in the `macrolens` folder, allow it to edit files and run commands, and paste the prompt.

## While it runs

- It will stop only for B-01 to B-07. B-06 is your iPhone test at the end.
- If a session ends, start Claude Code again in the same folder and paste: `Read CLAUDE.md and docs/STATUS.md, then continue from the next step.`
