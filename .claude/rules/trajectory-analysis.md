# Trajectory Analysis

How to read data patterns over time. Point values are snapshots; slope is signal.

Applies whenever interpreting whether an intervention is working, whether a marker is improving, or whether the current situation warrants a course change.

## Core principles

### Slope > point

A single lab value is a snapshot. Three or more values tell a trend. When interpreting whether an intervention is working, the trajectory is what matters, not the absolute value on any given date.

When a user asks "is my [marker] good?" — answer by showing the trajectory, not just the point. "Your [marker] was [value] in [month], [value] in [month], [value] in [month]. Trajectory is upward/downward, consistent with the [intervention] you started in [month]. Current value is [position] relative to [the threshold framework being applied]."

### Minimum data for a trend claim

| Data points | What you can say |
|-------------|------------------|
| 1 | Snapshot only; no direction, no trend |
| 2 | Direction hint ("moving up/down") — not a trend |
| 3+ | Pattern claim available; can speak to slope |
| 5+ | Pattern with confidence; can identify inflection points |

Never claim "the intervention is working" based on two values. Claim "direction is encouraging" and wait for the third.

### Rate of change matters

Moving toward target at 1% per quarter is different from 5% per quarter. Both are "working" but have different implications for:

- Projected timeline to target
- When to consider intensifying intervention
- When to declare progress sufficient for the current dose/protocol

Report rate of change when data permits.

### Trajectory targets are claims with their own evidence base

Target is often not a static value but a trajectory shape. "Slowly moving into a target range" is a valid trajectory goal — different from "reach X value by Y date." But the target itself (whatever its shape) is a threshold claim.

Apply the Threshold-as-Claim Discipline (see `epistemic-discipline.md`) when stating any target: surface its evidence base, acknowledge debate where it exists, treat targets as working hypotheses calibrated against outcomes.

Different markers warrant different trajectory shapes:

- **Static target** (reach X value): a fixed threshold-as-claim — surface the evidence behind X
- **Range target** (maintain within Y-Z): a range-threshold — surface the evidence for both endpoints
- **Slope target** (continue trending toward better): a directional goal — define "better" and surface the evidence for the direction
- **Stability target** (no worse than current): preserving the user's baseline — appropriate when interventions carry risk

### Subjective-objective integration

Lab trajectory and subjective trajectory are independent observations. When they disagree, the disagreement is diagnostic, not noise:

| Lab trajectory | Subjective trajectory | Interpretation |
|----------------|----------------------|----------------|
| Improving | Worsening | Side effects, wrong lever, something else going on, or anxiety confounder |
| Flat | Improving | Placebo effect, confounding lifestyle change, subjective measurement noise |
| Improving | Improving | Confirmation — intervention is likely working |
| Worsening | Worsening | Clear signal to change course |
| Improving | Flat | Lab change too small to feel, or effect lags lab marker |
| Flat | Worsening | Something unmeasured is driving the subjective change |

Surface both trajectories; never collapse to one.

### Variability is information, not noise

High variability in a marker is diagnostic, not random noise.

- Some markers have expected high variability: free testosterone (diurnal variation), cortisol (diurnal and stress-responsive), fasting insulin (meal-dependent if mis-timed draws)
- Some markers should be stable: HbA1c (90-day average), TSH, lipid panel (if diet and fasting consistent)
- Suddenly increased variability in a stable marker suggests: inconsistent intervention adherence, new confounder, lab methodology change, or emerging condition

Surface variability changes, especially in markers that were stable and have become variable.

## Trajectory-triggered surfacing

In `.claude/rules/proactive-surfacing.md`, any marker with 3+ data points gets trajectory assessment at session start. Surface when:

- **Trajectory diverges from expected direction** — intervention should raise a marker and it's flat or falling, or vice versa
- **Rate of change suggests timeline shift** — moving faster or slower than the plan assumed
- **Variability increases suddenly** — marker was stable, becomes noisy
- **Plateau after expected progress window** — improvement stalled at a point before target
- **Approaching red-flag threshold** — value trending toward a decision-trigger boundary in active strategic plans

## Interpretation discipline

### Don't over-interpret noise

A single out-of-range value in an otherwise normal trajectory is a data point, not a crisis. Ask:

- Is this measurement methodologically clean (fasting state, time of day, hydration, recent illness)?
- Does a repeat draw change the picture?
- Does the trend continue or return to baseline?

### Don't under-interpret trends

Three consecutive values moving in a concerning direction is not noise even if each individual value is in range. A clear slope away from the threshold being applied matters before the threshold itself is breached.

### Don't confuse correlation with causation

When an intervention was started alongside a lifestyle change (common), attributing outcome to the intervention is premature. Surface confounders honestly.

### Track interventions alongside trajectory

When reviewing trajectory, always overlay what changed in interventions, lifestyle, or external factors over the same window. Lab movements without intervention movements warrant investigation (new underlying condition? new medication? new stressor?).

## Integration with strategic plans

When a strategic plan exists (`my-data/strategic-plans.json`), trajectory analysis drives:

- Decision-trigger evaluation (did anything pre-committed-to fire?)
- Reassessment preparation (what have we learned since the plan was written?)
- Target-hit assessment (are we there? trajectory sufficient to project arrival?)
- Intervention effectiveness judgment (is this working as planned? faster, slower, sideways?)

At every formal reassessment, produce a trajectory summary per tracked marker plus subjective domain, alongside an intervention timeline, so cause and effect are visible together.
