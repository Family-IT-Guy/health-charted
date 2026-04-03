# Onboarding

How to welcome a new user and build their health profile. Read this guide only when `onboarding` in `my-data/session-status.json` is null or has `status: "in_progress"`.

## Initialization

If `onboarding` is null, this is a first run:

1. Write the onboarding tracker to `my-data/session-status.json`:
   ```json
   "onboarding": {
     "status": "in_progress",
     "phase_1": {
       "welcome": null,
       "intake": null,
       "privacy_disclosure": null
     },
     "phase_2": {
       "backup_setup": null,
       "tool_setup": null,
       "initial_priorities_set": null
     }
   }
   ```

2. Initialize git versioning per `guides/data-backup.md` (if `.git` directory does not exist).

## Phase 1: First visit

Model this on a first visit with a private doctor who has no time or budget constraints. The goal is to understand the full health picture before making any recommendations.

### Step 1: Welcome and capabilities

Introduce what this is and what you can do. Cover these capabilities, framed around how they help the user:

- **Data tracking**: capture health data (labs, symptoms, medications, lifestyle) and cross-reference it over time. If labs shift in a direction that correlates with a treatment change from three months ago, you'll catch that.
- **Research**: understand any condition, treatment, or supplement from the ground up. Mechanism-level analysis with cited sources, not summaries of consensus positions. Research files are kept and age-tracked so you know when to check for updates.
- **Provider prep**: before a doctor's appointment, generate specific questions based on accumulated data, frame requests in language the provider will respond to, and anticipate common pushback. After the visit, process what happened and route it into the records.
- **Pattern detection**: as months of data accumulate, check for correlations the user would miss. Symptom patterns that align with environment changes, lab trends that track with treatments, seasonal cycles.
- **Ongoing monitoring**: at the start of each session, check for overdue labs, aging research, stale hypotheses, upcoming appointments, and anything that needs attention.

Then set the expectation: to be most useful, you need their full health picture, same as a doctor would at a first visit.

Do not use a scripted welcome. Generate it naturally from these constraints. Must work equally for someone managing a condition, investigating something unexplained, or optimizing general health. Do not: describe file structure, use the word "system," start with "Hey" or "Hello!" or "Welcome to Health Charted," or overpromise. Timestamp the step.

### Step 2: Who is this for?

Before going further, ask who this health tracking is for. This is a routing question that shapes everything else.

- **Self:** proceed normally.
- **Someone else (child, aging parent, dependent):** ask about their relationship to the person. The user is the manager; the subject is the person whose health is being tracked. Adjust the intake to gather information about the subject. Record both in `my-data/profile.json` (the subject/user fields are designed for this split).
- **Multiple people:** explain that each person should have their own separate Health Charted folder. One person per folder keeps data clean and prevents cross-contamination. Offer to help create and initialize a second folder for the other person. Do not add a second person's data to this folder.

### Step 3: Chief complaint

Ask what brings them here (or what's going on with the person they're tracking). Listen. If they have a specific concern, address it: ask clarifying questions, provide an initial assessment, trace the mechanism. Don't make them wait through a full intake before addressing what brought them in. If they say "nothing specific, just general health," skip the focused assessment and move to the intake.

### Step 4: Structured intake

After addressing their immediate concern (or if they have none), transition to filling in the full picture: "Now let me get the rest of your/their health background so I can be more useful going forward."

Work through these categories one question at a time, conversationally, skipping anything already covered. If tracking someone else, ask about the subject, not the user:

- Age and sex (needed for lab range interpretation)
- Medical history (diagnosed conditions, surgeries, major health events)
- Current medications (name, dose, frequency, why, who prescribed)
- Current supplements (name, dose, form)
- Allergies
- Family history (significant conditions in parents, siblings, grandparents)
- Lifestyle (exercise, diet, sleep quality, stress, substances)
- Current providers (who, what role, how often)
- Recent labs or imaging (if the platform supports filesystem access, offer to search their computer for lab PDFs)
- Goals (what they want from this, beyond any immediate concern)

Write data to the appropriate files as you go. Don't batch updates to the end. Each category maps to a specific data file per `guides/data-routing.md`. Timestamp when the intake is substantially complete.

### Step 5: Privacy disclosure

Weave into the conversation naturally, not as a blocker. When the user shares their first piece of health information, briefly note: conversations go through the AI provider's servers. Most platforms let you opt out of training, but it's worth knowing. Data files stay on their computer. Timestamp after disclosure.

Phase 1 is complete when welcome, intake, and privacy are all timestamped.

## Phase 2: Infrastructure

Phase 2 happens after Phase 1, or at the start of the next session. The user has seen what the system can do and is motivated to invest in setup.

### Step 6: Backup setup

Explain why it matters in plain language per `guides/data-backup.md`. Walk through setup if the user wants. Timestamp after discussion (even if deferred).

### Step 7: Tool setup

Walk through each tool individually per `guides/recommended-tools.md`. For each tool, explain what it does, why it helps, and offer to set it up now. Perplexity API first (most impactful), then sequential thinking MCP, then yt-dlp. Don't batch them into one message. Timestamp after each is offered (whether installed or deferred).

### Step 8: Initial priorities set

If not already set during Phase 1, review what was discussed and set priorities in `my-data/session-status.json`. Timestamp when done.

## Completion

When all Phase 1 and Phase 2 steps have timestamps, set `onboarding.status` to `"completed"`.

## Resuming incomplete onboarding

If `onboarding.status` is `in_progress` at session start:
- Check which steps have null timestamps across both phases.
- Phase 1 incomplete: weave remaining steps into the conversation naturally.
- Phase 2 incomplete: surface them explicitly: "We still haven't set up [backup/tools/priorities]. Want to handle that now, or after your question?" The user may defer, but incomplete Phase 2 steps must be raised at every session start until completed.
