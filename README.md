# Health Charted

Your health data, tracked and understood. An AI research partner that gets smarter about your situation over time.

**Website:** healthcharted.com
**License:** [CC BY-NC-SA 4.0](LICENSE)

## What you can do with this

**Catch connections you'd miss.** Your labs shift three months after a medication change. A symptom pattern lines up with a lifestyle change from weeks ago. You'd never hold all that in your head. This does.

**Walk into every doctor's appointment prepared.** Specific questions based on your data. Requests framed in language providers respond to. Prepared responses for common pushback. After the visit, everything gets captured and cross-referenced.

**Actually understand your lab results.** Not just "within normal range." What the numbers mean for your specific situation, how they compare to your previous results, and what functional optimal looks like versus the standard reference range.

**Research any health topic from the ground up.** Mechanism-level analysis with cited sources. Not "studies show" hand-waving. The actual biochemistry, the actual data, the quality of the evidence. Research files are kept and flagged for review when they get old.

**Build a health record that lasts.** Every symptom, lab result, medication, decision, and provider interaction is captured in structured files on your computer. Months from now, the system knows your full history and uses it.

## Getting started

You need [Claude Desktop](https://claude.ai/download) with a Pro plan ($20/month).

1. Download this folder (Code > Download ZIP, or `git clone`)
2. Open Claude Desktop
3. Click the **Code** tab
4. Select the downloaded folder
5. Say hi

The system walks you through everything from there. It starts like a first visit with a new doctor: what brings you here, your health background, medications, lifestyle. One question at a time, conversational, not a form.

If you're using a different LLM platform, rename `CLAUDE.md` to match your platform's convention. The content works with any capable LLM that can read and write local files.

## Who this is for

- You're managing your own health and want a research partner
- You're a parent tracking a child's health
- You're helping an aging parent navigate their care
- You want to optimize your general health, not just react to problems

Each person gets their own folder. If you're tracking health for yourself and your kid, that's two separate folders.

## How it works

You talk to it about your health. It asks questions, captures data, and cross-references everything over time. When you bring in new information (labs, symptoms, something a doctor said), it routes it to the right place and checks it against your history.

It reasons from mechanisms, not authority. When it explains why something matters, it traces the biological pathway. When evidence is strong, it says so. When it's weak or contested, it says that too. When its analysis conflicts with what a doctor told you, it presents both sides and you decide.

Your data stays on your computer in structured files. The system versions every change automatically (so a bad edit can be undone) and helps you set up proper backup.

## Optional tools

These aren't required but improve the experience:

- **Perplexity API** ($): deep, cited research with academic source access. The system walks you through setup when you first need it.
- **Sequential Thinking MCP**: improves analytical reasoning for complex health questions. Free.
- **yt-dlp**: pulls transcripts from medical lectures and talks for research. Free.

## Updating

When a new version is available, the system tells you at session start. Updates only touch the framework files (instructions and guides). Your health data is never modified by an update.

## License

[CC BY-NC-SA 4.0](LICENSE). Use it, adapt it, share it. Credit the project, keep it non-commercial, share adaptations under the same license.
