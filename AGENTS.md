# Project Development Guide

## Project Goal

Build a Production Agentic RAG system from scratch. The primary goals are to learn system design, evolutionary architecture, and engineering practice. Follow the [project roadmap](doc/roadmap.md) from V0, a single LLM API call, through V11, a research assistant with an automated paper collection and agentic question answering.

Write all repository documentation in English, including roadmap updates, stage plans, and architecture decision records (ADRs).

## Development Principles

- Implement one roadmap stage at a time. Keep changes within the active stage and introduce middleware only when its stage requires it.
- Apply KISS, YAGNI, and SOLID in proportion to the current problem. Prefer straightforward, maintainable code. Introduce an abstraction when the current implementation needs it to isolate an external dependency, remove meaningful duplication, or support multiple implementations.
- Preserve existing user-facing behavior unless the approved stage calls for a change. Reuse working entry points and business logic as new interfaces are added.

## Stage Branches and Milestones

- Start each stage from up-to-date `main` on a dedicated branch named `stage-vNN`. Use two digits, so V0 uses `stage-v00`, V1 uses `stage-v01`, and V11 uses `stage-v11`.
- Update this root `AGENTS.md` on the stage branch. Keep the project-wide rules and add or replace a `Stage Instructions` section with the stage identifier, scope, constraints, and links to the plan and the stage workflow. Use the branch name to identify the stage in progress. On `main`, the section describes the most recently merged stage. Milestone tags preserve earlier versions of the file.
- Add a separate `.github/workflows/stage-vNN.yml` on each stage branch. Run its checks on pushes to that branch and on pull requests from it to `main`. GitHub's pull request branch filter matches the target branch, so use `main` in that filter and check `github.head_ref` in the job condition. Keep historical workflows in the repository, with their jobs skipped for pull requests from later stages.
- Open a pull request into `main`. Merge once the shared CI and stage checks pass, any required manual checks are recorded in the plan, and the stage meets its acceptance criteria. Wait for shared CI on the resulting `main` commit before tagging it.
- Create an annotated `vNN` tag on the exact commit that landed the stage in `main` and push that tag to GitHub. For example, completing `stage-v01` produces `v01`. The milestone workflow verifies that the tagged commit is in `main` and creates the corresponding GitHub Release.

The [shared CI workflow](.github/workflows/ci.yml) provides a Python syntax check. Each stage defines its own automated tests and records live or credential-dependent acceptance checks in its plan. The [milestone release workflow](.github/workflows/milestone-release.yml) delivers tagged snapshots through GitHub Releases. Add application deployment when a stage has an approved deployment target.

## Stage Plans

When implementation of a stage is requested, create its plan before changing application code. Store it directly in `doc/plans/<version>/plan.md`, using the lowercase version identifier. For example, the V0 plan belongs at `doc/plans/v0/plan.md`. Create plans as their stages begin and keep each completed plan at the same path as the historical record.

Each plan should cover the following items at a level of detail appropriate to the stage.

- **Goal and scope** — Link to the corresponding roadmap section. Describe what the user will be able to do, what is in scope, and what remains for later stages.
- **Starting point and approach** — Describe the existing behavior, the proposed changes, and how data will flow through the affected modules. List implementation steps in dependency order and identify assumptions and risks.
- **Decisions and approval** — Identify decisions that need approval and link to relevant ADRs. Capture any decisions already confirmed by the user.
- **Acceptance and verification** — Translate the roadmap criteria into concrete checks with inputs or fixtures, commands or steps, and expected results. Link the stage workflow and identify checks that require credentials, external services, or a live model.
- **Completion record** — Record relevant changes from the original plan, actual results and observations, and any checks still pending. Mark the plan complete once its acceptance criteria are met.

## Architecture Decisions and Approval

Before introducing or replacing a service, middleware component, or major dependency, or changing a core module boundary, data flow, persistence strategy, or public interface, explain the following in the stage plan or a draft ADR.

1. What limitation exists in the current system?
2. Why does the current stage need this change?
3. What alternatives are viable, including keeping the current design?
4. What complexity, operating cost, or failure modes does the proposed change add?
5. What check will show whether the change delivers the intended value?

Obtain user confirmation before implementing an architectural decision. A confirmed stage proposal covers the decisions it explicitly includes. Proceed with implementation details, local fixes, tests, and documentation inside that approved scope. If implementation reveals a material architectural change, update the plan, explain the impact, and seek confirmation before making that change.

Record significant decisions in `doc/adr/`. Include the context, alternatives, decision, consequences, status, and how the decision will be evaluated. Link the ADR from the stage plan once it exists.

## Implementation and Verification

1. Read the active roadmap section and inspect the relevant code and existing plans or ADRs.
2. Create the stage branch, update its `AGENTS.md` instructions, and write the stage plan. Present the proposed approach and resolve any required architectural approvals.
3. Implement the smallest complete change for the stage. Write or update tests for the new behavior and relevant failure cases, and add the stage workflow.
4. Run the relevant tests and acceptance checks through the stage's normal entry points. From V1 onward, maintain a fixed set of questions and expected sources so retrieval and answer quality can be compared across versions.
5. Update the stage plan with the results and update usage documentation when setup or behavior changes. Keep the final plan in its original archive path. Merge through a pull request and tag the milestone as described above before moving to the next stage.

## Learning and Reporting

At the end of a stage, explain the key module responsibilities, how data moves through the system, why the design was chosen, and which trade-offs or risks matter at this stage. Distinguish verified behavior from assumptions. Report the checks actually run, their outcomes, and anything still unverified. Identify mocked checks separately from live integration tests, and use measured data for any performance claims.
