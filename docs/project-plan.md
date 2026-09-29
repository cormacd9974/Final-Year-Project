# Project Plan: Variations on Gale-Shapley for Project Allocation

Supervisor: Charles Markham, Maynooth University Computer Science.
Student: Cormac Dunne.
Last updated: 2026-09-28.

This document is a working plan, not a fixed spec. Update it as scope gets confirmed with Charles and as reality intervenes.

## The framing

The research contribution here isn't a new matching algorithm. Gale-Shapley and its many-to-one, ties, and incomplete-list extensions are solved and published. What's open is what happens when those extensions get combined for a real, messy use case, measured honestly, with a human kept in meaningful control of the result. That's the actual thesis of the project. Time goes on the constraints, the evaluation, and the ethics, not on inventing new algorithmic theory.

## Status

- [ ] Scope confirmed with Charles
- [ ] Key dates confirmed (proposal, interim, final report, demo/viva)
- [ ] AI-assistance policy for the project confirmed
- [ ] Real vs synthetic data question resolved
- [ ] Primary worked domain confirmed (proposing: student-to-project/supervisor allocation)

## Decision log

**2026-09-29: Charles vs Emlyn/portal fork.** Charles offered two directions: a research-investigation project with him (algorithm variations, evaluation, ethics, matches the rest of this plan), or moving over to Emlyn to focus on integrating the allocator into MU's actual portal. Leaning towards staying with Charles: it's the scope already planned in detail here, the risk stays inside my own control rather than depending on portal access I haven't scoped, and real deployment was already a stretch goal below rather than something the core project was betting on. Portal integration stays on the table as a stretch item if time allows and Emlyn's happy to advise on just that piece. Not yet confirmed with Charles.

## Timeline

**Late Sept to mid-Oct: landing phase**
Read the two "read first" papers below. Turn this plan into a one-page scope document, get it signed off or redirected by Charles, before writing real code. Set up repo, CI, basic skeleton. Warm-up exercise: plain one-to-one Gale-Shapley against a textbook worked example, written by hand to build intuition before the harder version.

**Mid-Oct to mid-Nov: the engine**
Many-to-one matching with capacity at both project and supervisor level, incomplete lists, one working tie-break method, both proposing directions. Property-based tests (no blocking pairs, capacities respected) plus golden tests against known worked examples. Natural point for any proposal deliverable MU requires.

**Mid-Nov to mid-Dec: data and skeleton app**
Synthetic data generator, backend API, data model, bare frontend (enter preferences, trigger a run, see raw results). Demo to Charles before the break.

**January: buffer**
Assume exams. Nothing load-bearing planned here. Interim report if one is due around this point.

**Feb to mid-March: the actual investigation**
Explanation view, coordinator override with instability warnings, two or three ranking strategies, evaluation harness comparing variants over synthetic data. Most of the grade lives here; budget the most time here.

**Mid-March to mid-April: ethics, worked examples, writing**
Ethics and moral hazard chapter grounded in what the experiments actually showed. Worked examples for the "explain this simply" requirement. UI polish.

**Late April to submission: buffer, deliberately**
Final testing, dissertation edits, demo prep. Left empty on purpose.

Write the dissertation continuously from October, not all at the end. Get a standing biweekly slot with Charles rather than ad hoc meetings, and bring one specific decision to each one.

## Scope

### Must-haves

- Many-to-one engine matching the Student-Project Allocation (SPA) model: students rank projects, projects have their own capacity, each project belongs to a supervisor with an aggregate capacity across their projects, supervisors rank students.
- Incomplete lists on both sides.
- Ties, handled with weak stability at minimum.
- Both proposing directions (student-proposing and supervisor-proposing).
- Two or three ways for supervisors to rank students (manual, grades-based, lottery tie-break), not all five originally listed.
- The "why did I get this" explanation view.
- Coordinator override screen that detects and surfaces instability (blocking pairs) when a manual change is made.
- Synthetic data generator with tunable parameters (student count, project count, capacity distribution, list length, tie density).
- Evaluation harness comparing variants over the same synthetic data: percentage getting 1st/2nd/3rd choice, average rank, blocking pairs under override, runtime at scale.
- The ethics analysis, with real time budgeted for it across the year, not written in the last week.

### Stretch (pick two or three, not all)

- Generalising the engine to generic task allocation (keep core types abstract: agents and posts, not students and projects, at the engine layer). Cheap if done from the start.
- Strong or super-stability with ties. Real theory, weak effort-to-payoff ratio for an undergrad project. Good as a "road not taken" discussion in the literature review.
- Skills-match ranking via free text or embeddings. NLP scope creep, avoid unless there's spare time.
- Multi-round or dynamic reallocation (waitlists, dropouts, late joiners).
- Real department deployment with real data. Needs ethics approval and data protection sign-off; usually too slow to land inside one academic year.
- Education conference paper as an actual submission this year rather than a write-up afterward.

## Architecture and stack

React + TypeScript frontend, Node/Express + TypeScript backend, MongoDB, Docker. No Kubernetes, it solves problems this project doesn't have and costs ops time it doesn't have. Docker Compose locally; a single small instance or a PaaS (Render, Fly.io) if a hosted demo is needed.

Three deliberate design decisions:

1. **Matching engine as a pure, isolated module.** No HTTP, no database, no UI inside it. Input: students, projects, supervisors, preferences, capacities. Output: a matching plus a full trace of the algorithm's decisions. Makes it unit-testable in isolation and makes the task-allocation stretch goal nearly free.
2. **Instrument the algorithm to log its own decisions**, not just the final result. Every proposal, every rejection, every bump. Do this from the start. This is what makes the explanation feature tractable rather than painful.
3. **Model data the way the SPA literature models it**, not however is locally convenient: Students, Projects (capacity, belongs to one Supervisor), Supervisors (own aggregate capacity), Preferences, AllocationRun (variant, input snapshot, result, trace), Override (original result, human's change, who, why, resulting blocking pairs).

Testing: property-based tests (fast-check) for the core invariants (no blocking pairs, capacities respected, matches come from someone's own list), plus golden tests from small hand-built worked examples (4-6 students, 3-4 projects). Reuse those same worked examples three times: as test fixtures, as the explanation UI's walkthrough content, and as the dissertation/demo example.

## Literature

**Read first**

- Gale, D. and Shapley, L.S., "College Admissions and the Stability of Marriage," *American Mathematical Monthly*, Vol. 69, No. 1, pp. 9-15, 1962.
- Manlove, D.F., *Algorithmics of Matching Under Preferences*, World Scientific, 2013. Read the Hospitals/Residents and Student-Project Allocation chapters, not cover to cover. Manlove's group has deployed systems like this in practice, including allocating students to courses and projects, and matching UK kidney patients to donors.

**Core technical foundation**

- Abraham, D.J., Irving, R.W. and Manlove, D.F., "Two Algorithms for the Student-Project Allocation Problem," *Journal of Discrete Algorithms*, 2007 (preliminary version: ISAAC 2003). The SPA model this project is built against.
- Gusfield, D. and Irving, R.W., *The Stable Marriage Problem: Structure and Algorithms*, MIT Press, 1989. More proof detail than Manlove's book where needed.
- Stretch reading on ties: the follow-up papers on super-stability and strong stability in SPA with ties (both on arXiv, both building on the Abraham/Irving/Manlove model).

**Ethics and the human side**

- Roth, A.E., *Who Gets What and Why: The New Economics of Matchmaking and Market Design*, Mariner Books, 2015.
- Abdulkadiroğlu, A. and Sönmez, T., "School Choice: A Mechanism Design Approach," *American Economic Review*, Vol. 93, No. 3, pp. 729-747, 2003.

**Useful contrast**

- Charlin, L. and Zemel, R., "The Toronto Paper Matching System," 2013. Reviewer-paper assignment via optimization over a relevance score rather than stability. Good contrast for the "most stable isn't always most people's first choice" discussion.

## Questions for Charles

1. Exact dates and format for proposal, interim deliverable, final report, demo/viva, and how the grade splits between written work and the working system.
2. Is any real, anonymised historical allocation data available, even just for comparison? If real data becomes relevant, does it need ethics approval, and how long does that take?
3. Confirm student-to-project/supervisor allocation as the primary worked domain, with assignments/examiners discussed as generalisations rather than fully built.
4. Is the education conference paper a real target for submission this year, or a post-dissertation aim?
5. Is there a description of how allocation happens in the department today, even informally, to use as a real baseline?
6. Anything beyond the official department deliverables Charles wants to check in on, and preferred meeting cadence.
7. Any existing systems, at Maynooth or elsewhere, worth looking at or specifically avoiding re-treading.
8. What's the department's / MU's policy on using AI coding assistants for final year project work, and does anything need to be disclosed in the submission.
