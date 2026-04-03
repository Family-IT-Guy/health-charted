# Provider Integration

How the system helps the user prepare for, navigate, and process interactions with healthcare providers.

## Philosophy

Providers are tools in the user's toolkit. They have specialized training, diagnostic access, prescribing authority, and procedural skills. The user is the integrator who coordinates across providers and makes final decisions about their own health.

This system helps the user get maximum value from every provider interaction. That means preparation, advocacy, and follow-through.

## Appointment Preparation

When a provider visit is approaching:

1. **Summarize current state.** Pull relevant data from my-data/ files: recent labs, active symptoms, current treatments, open questions, unresolved hypotheses.
2. **Generate specific questions.** Each question should have a reason: "Ask about X because your data shows Y and you need to understand Z." Don't generate generic checklists.
3. **Identify gaps.** What labs or tests should be requested? What hasn't been measured that should be? What's overdue?
4. **Frame requests in medical language.** Providers respond to clinical framing. "I'd like to check my ferritin level" is weaker than "Given my symptoms of X and Y, I'd like to rule out iron deficiency with a serum ferritin and iron panel." Help the user speak the provider's language without being adversarial.
5. **Anticipate resistance.** Common provider objections include:
   - "Your labs are normal" (when values are within standard range but not functionally optimal)
   - "That test isn't indicated" (when the provider doesn't see the connection the user sees)
   - "Where did you read that?" (dismissal of patient research)
   - "We don't do that here" (institutional limitation, not medical reasoning)
   - "Insurance won't cover it" (may or may not be true, often negotiable)

   For each likely objection, prepare a response that is kind, specific, and grounded in mechanism. Not confrontational. The goal is to help the provider help the user. Frame it as collaboration: "I understand that's outside the usual protocol. Here's why I think it's relevant for my specific situation."
6. **Prepare for the ego dynamic.** Some providers feel threatened by informed patients. The user should be coached to:
   - Lead with questions, not assertions ("I've been reading about X and wanted your perspective" rather than "I need you to order X")
   - Acknowledge the provider's expertise while advocating for their own data
   - Know when to push and when to find a different provider. Not every battle is worth fighting in every visit.

## Insurance Navigation

When a recommendation may involve insurance friction:

- Note what typically requires prior authorization for the relevant type of care
- Suggest language that supports medical necessity ("diagnosis of X with documented symptoms of Y, standard treatment has been insufficient per [dates/data]")
- Note alternatives if coverage is denied: cash pay pricing, alternative labs (direct-to-consumer lab services), different procedure codes, appeal processes
- Record insurance interactions in `my-data/provider-log.json` for reference in future encounters

## Processing Provider Interactions

After a provider visit, when the user shares what happened (recording, transcript, notes, or verbal summary):

1. **Extract clinically relevant information:** diagnoses discussed, labs ordered, medication changes, provider recommendations, referrals, follow-up plans.
2. **Create a structured summary** in `my-data/provider-log.json`: date, provider, visit type, key topics, orders, medications, recommendations, follow-up timeline.
3. **Route extracted data** to appropriate files per `guides/data-routing.md`: lab orders to session-status.json as pending items, medication changes to treatment-stack.json, etc.
4. **Flag conflicts** between the provider's assessment and the system's current understanding. Present both sides with evidence. The user decides how to proceed.
5. **Update hypotheses** if the provider interaction provided new data that validates or invalidates open hypotheses.

## Care Team Tracking

The provider roster lives in `my-data/profile.json` (the providers array). Visit history lives in `my-data/provider-log.json`. Over time, the visit history reveals which providers are responsive to the user's needs and which are not. That pattern is useful information for the user's decision-making about their care team.
