# Onboarding

How to welcome a new user and build their health profile. Read this guide only when `onboarding` in `my-data/status.json` is null or has `status: "in_progress"`.

## Initialization

If `onboarding` is null, this is a first run:

1. Write the onboarding tracker to `my-data/status.json`:
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

2. Initialize git versioning per the Data Backup rules (if `.git` directory does not exist).

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
- **Someone else (child, aging parent, dependent):** ask about their relationship to the person. The user is the manager; the subject is the person whose health is being tracked. Adjust the intake to gather information about the subject. Record both in `my-data/health-profile.json` (the subject/user fields are designed for this split).
- **Multiple people:** explain that each person should have their own separate Health Charted folder. One person per folder keeps data clean and prevents cross-contamination. Offer to help create and initialize a second folder for the other person. Do not add a second person's data to this folder.

### Step 3: Chief complaint

Ask what brings them here (or what's going on with the person they're tracking). Listen. If they have a specific concern, address it: ask clarifying questions, provide an initial assessment, trace the mechanism. Don't make them wait through a full intake before addressing what brought them in. If they say "nothing specific, just general health," skip the focused assessment and move to the intake.

If the chief complaint is emotionally heavy, do not respond with empathy phrases or performed presence. See CLAUDE.md Tone. Acknowledge factually, ask the next clarifying question, get into the work.

### Step 4: Structured intake

After addressing their immediate concern (or if they have none), transition to filling in the full picture: "Now let me get the rest of your/their health background so I can be more useful going forward."

Work through the categories below one question at a time, conversationally, skipping anything already covered. If tracking someone else, ask about the subject, not the user.

**Pacing:** These categories are not equal in depth. Age and allergies are one exchange. Family history and lifestyle are conversations. Let the user's situation determine how long you spend on each category. If they're sharing a lot, stay there. Do not rush through a deep category to maintain checklist pace. Before transitioning away from any category, ask: "Anything else about [topic] before we move on?"

**Revisits welcome:** The intake is a coverage guide, not a rigid sequence. If the user returns to a previous topic at any point, stay with it. Human memory is associative — a question about sleep might trigger a memory about a grandmother's insomnia. Update the relevant data file, then return to where you left off. Never resist or redirect a user who is volunteering health information, regardless of where you are in the list.

#### Quick categories

- **Age and sex** — needed for lab range interpretation
- **Allergies** — medications, foods, environmental
- **Current supplements** — name, dose, form

#### Deep categories

These require multiple exchanges. Use this pattern: ask an open question, listen, probe gaps, follow threads, check for patterns, confirm completeness.

- **Medical history** — Don't just ask for a list. Walk through the timeline: childhood, adolescence, adulthood. For each condition: when diagnosed, how it's managed now, current status. Surgeries, hospitalizations, major health events. If the user lists multiple conditions, explore connections between them.

- **Family history** — This is not one question. Ask about each category of relative:
  - Parents: conditions, age of onset, cause of death if applicable
  - Siblings: conditions, current health
  - Grandparents (both sides): conditions, cause of death if applicable
  - Look for hereditary patterns: clustering of heart disease, cancer, diabetes, autoimmune conditions, mental health conditions across family members
  - Ask about conditions they might not think to mention: high blood pressure, high cholesterol, thyroid issues, depression/anxiety, substance abuse
  - If they say "I don't know much about my family history," accept that — note what's unknown and move on

- **Current medications** — For each: name, dose, frequency, who prescribed, why, how long they've been on it. If multiple medications, note potential interactions to research later.

- **Lifestyle** — Each dimension is its own question, not one compressed exchange:
  - Exercise: type, frequency, intensity, how long they've been doing it
  - Diet: general pattern, any restrictions or allergies, relationship with food
  - Sleep: quality, duration, consistency, any issues (snoring, waking, insomnia)
  - Stress: sources, how they manage it, how it affects them physically
  - Substances: caffeine, alcohol, tobacco, cannabis, recreational — frequency and amount. No judgment in tone.

- **Current providers** — Who they see, what role each plays, how often, how satisfied they are with each relationship

#### Completion categories

- **Recent labs or imaging** — if the platform supports filesystem access, offer to search their computer for lab PDFs
- **Goals** — what they want from this, beyond any immediate concern

Write data to the appropriate files as you go. Don't batch updates to the end. Each category maps to a specific data file per the Data Routing rules. Timestamp when the intake is substantially complete.

### Step 5: Privacy disclosure

Weave into the conversation naturally, not as a blocker. When the user shares their first piece of health information, briefly note: conversations go through the AI provider's servers. Most platforms let you opt out of training, but it's worth knowing. Data files stay on their computer. Timestamp after disclosure.

Phase 1 is complete when welcome, intake, and privacy are all timestamped.

## Phase 2: Infrastructure

Phase 2 happens after Phase 1, or at the start of the next session. The user has seen what the system can do and is motivated to invest in setup.

### Step 6: Backup setup

Explain why it matters in plain language per the Data Backup rules. Walk through setup if the user wants. Timestamp after discussion (even if deferred).

### Step 7: Tool setup

Walk through each tool individually per `guides/recommended-tools.md`. For each tool, explain what it does, why it helps, and offer to set it up now. Perplexity API first (most impactful), then sequential thinking MCP, then yt-dlp. Don't batch them into one message. Timestamp after each is offered (whether installed or deferred).

When setting up the Perplexity API, disclose that research calls cost money: "Research uses the Perplexity API, which charges a small amount per search. Most health questions cost well under a dollar. Very thorough research on a complex topic might cost a few dollars. By setting up the API key, you're authorizing those charges." Keep it factual and brief. The API key setup is the cost consent.

### Step 8: Initial priorities set

If not already set during Phase 1, review what was discussed and set priorities in `my-data/status.json`. Timestamp when done.

## Completion

When all Phase 1 and Phase 2 steps have timestamps, set `onboarding.status` to `"completed"`.

## Resuming incomplete onboarding

If `onboarding.status` is `in_progress` at session start:
- Check which steps have null timestamps across both phases.
- Phase 1 incomplete: weave remaining steps into the conversation naturally.
- Phase 2 incomplete: surface them explicitly: "We still haven't set up [backup/tools/priorities]. Want to handle that now, or after your question?" The user may defer, but incomplete Phase 2 steps must be raised at every session start until completed.

## Module engagement intake

Core onboarding (Phases 1-2 above) is general — it captures identity, conditions, providers, treatments, and goals regardless of which modules are installed. Module-specific intake happens separately, when a module first engages during conversation.

When the user first triggers a module's domain (per the Session Initialization module scan and the LLM's domain-trigger judgment), check whether that module has been engaged before. Track per-module engagement state in `my-data/status.json` under `engaged_modules`:

```json
{
  "engaged_modules": ["prenatal", "peptide"]
}
```

If the engaging module is not yet in `engaged_modules`:

1. Read the module's `MODULE.json` `module_intake` array (or per-submodule `intake_questions`, when engagement is at submodule grain).
2. Ask the intake questions one at a time in conversation. Capture answers to relevant my-data files (`health-profile.json`, `lab-results.json`, etc.) per the Data Routing rules.
3. Add the module's namespace to `engaged_modules` in `my-data/status.json` once intake is complete.
4. Proceed with the user's substantive domain question.

If the engaging module is already in `engaged_modules`, skip intake and proceed.

Module-specific intake is opt-in by module — modules without `module_intake` declarations engage immediately without an intake step.
