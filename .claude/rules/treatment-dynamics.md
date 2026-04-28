# Treatment Dynamics

How interventions behave over time. These principles apply to any treatment with phases, schedules, or time-dynamic effects: medications, supplements, hormone protocols, nutritional interventions, lifestyle changes, behavioral interventions, peptide protocols, anything where the timeline matters.

## Schedule-as-Hypothesis

Published intervention schedules describe what was tested in specific populations under specific conditions. They are starting hypotheses for the individual user, not prescriptions.

When a published schedule appears in research (titration plans, dose-response curves, treatment durations, monitoring cadences):

- Treat it as evidence about what was studied — population, conditions, endpoints, methodology
- Treat the user's individual schedule as a decision to be calibrated against this user's response
- Don't transfer trial schedules to individual prescriptions silently

Trial schedules reflect specific safety/tolerability profiles in specific populations:

- The population studied may differ from this user (age, sex, comorbidity, concurrent medication, baseline state)
- The endpoint measured may not match the user's goal
- The duration may not capture the user's relevant timeframe
- The titration may have been chosen for trial logistics, not optimality

Individual schedules calibrate to this user. The trial schedule is the starting hypothesis. Subsequent monitoring and response inform whether the schedule should be adjusted (faster, slower, sustained, paused, reversed).

## Phase-Distinct Calibration

Different goals along an intervention's timeline require their own mechanism analysis. Inducing change, sustaining change, and reversing change are distinct phases with distinct dose-response and pharmacodynamic profiles. Do not assume continuity across phases.

Common phase distinctions:

- **Induction**: dose-response for producing the desired change. Often higher intensity. Driven by the gap between current and target state.
- **Maintenance**: dose-response for preserving the change once achieved. Often different intensity (sometimes lower, sometimes the same, occasionally higher) than induction. Driven by ongoing physiology, not the original gap.
- **Discontinuation**: pharmacodynamics of stopping. May involve rebound, withdrawal, downregulation reversal, return-to-baseline timeline. Distinct from "no intervention."
- **Cycling**: alternating intervention with non-intervention. Mechanism-justified for some interventions (receptor desensitization recovery, hormonal axis preservation) and folk-justified for others. Each cycling claim requires its own mechanism evaluation.

When the user transitions between phases:

- Don't assume the current dose continues to fit
- Reason about the mechanism of the new phase explicitly
- Surface phase-specific monitoring and decision triggers
- Document the phase transition as its own decision in `my-data/decisions.json`

## Adaptation, Plateau, and Failure

Apparent non-response in the data has multiple distinct mechanism causes:

- **Adaptation**: dose-response equilibrium reached for current dose; may need adjustment to continue progress, or may be the new steady state
- **Plateau**: the intervention has worked at its mechanism limit; further change requires different lever
- **Failure**: the intervention isn't engaging the mechanism it should be (wrong dose, wrong target, confounder masking)

Surface the distinction. Don't collapse to "it stopped working." Each requires different next steps.
