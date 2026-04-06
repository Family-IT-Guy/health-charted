# Health Charted

Your health, understood. An AI research partner that learns your situation and helps you make sense of it over time.

**Website:** healthcharted.com
**License:** [CC BY-NC-SA 4.0](LICENSE)

## What you can do with this

**Understand what's going on with your body.** Not just "your levels are normal." What the numbers mean for your specific situation, how they compare to your previous results, and what to watch for next.

**Walk into appointments prepared.** Specific questions based on your data. Context your doctor needs. After the visit, everything gets captured and connected to what the system already knows.

**Research any health topic from the ground up.** When something comes up (a diagnosis, a medication, a symptom that won't go away), the system researches the biology, traces the mechanisms, and cites its sources. You get the actual evidence, not a summary of what "studies show."

**Keep track of everything in one place.** Labs, medications, symptoms, provider visits, decisions, and the reasoning behind them. Across months and years, the system connects things you wouldn't think to connect.

**Manage health for your family.** Each person gets their own folder. If you're tracking your own health and helping with your kid's and your mom's, that's three separate instances. Each one learns that person's specific situation independently.

## Who this is for

- You want to understand your own health better
- You're managing a child's health
- You're helping an aging parent navigate their care
- You're dealing with something specific (a condition, unexplained symptoms, a new diagnosis)
- You just want to be healthier and don't know where to start

## Getting started

You need [Claude Desktop](https://claude.ai/download) with a Pro plan ($20/month).

1. Download Health Charted (green **Code** button above, then **Download ZIP**)
2. Unzip it somewhere on your computer
3. Open Claude Desktop, click the **Code** tab, and select the folder you just unzipped
4. Start a conversation. You might say something like "I'd like to set up my health profile" or just describe what's going on with your health. The system takes it from there.

Health Charted is built for Claude Desktop. It works with other AI platforms that can read and write local files, but setting that up requires some technical comfort.

## How it works

You own your health information. All of it. Every file this system creates is plain text on your computer. There's no account, no app, no company, no privacy policy. Nothing is stored anywhere except your computer.

The system is an education and research partner. You talk to it about what's going on with your health. Maybe you're investigating a specific condition. Maybe you want to understand your lab results better or figure out the right questions to ask your doctor. Maybe you just want to get healthier and don't know where to start. Whatever it is, the system helps you understand it from the ground up: what's happening in your body, why it's happening, and what your options are.

When you share information (lab results, medications, symptoms, what a provider told you), it keeps track of everything and connects it over time. A lab result today gets checked against your results from six months ago. A new medication gets checked against everything else you're taking. A symptom gets connected to possible causes based on your specific situation.

It traces the biology, not just the recommendation. Instead of "take calcium and iron separately," for example, it explains how the two compete for absorption so you understand why the timing matters. When evidence is strong, it says so. When researchers disagree, it tells you that too. When its analysis differs from what your doctor said, it shows you both perspectives. You decide what to do.

Your doctors are part of your team. The system helps you prepare for appointments with specific questions based on your data, and captures what happened afterward. If something a provider says conflicts with the research on file, it flags that for you so you have the full picture.

When you start a new medication or a new question comes up, the system checks whether it has research on file. If there's a gap, it lets you know and offers to look into it.

When your situation changes, the system identifies which parts of your health picture might need revisiting and walks you through it.

The longer you use it, the more useful it gets. Patterns emerge across months and years that no single doctor's visit can see. Your health picture builds on itself instead of resetting every time you see a new provider.

## Research tools

The system can research health topics on its own. For deeper research with cited sources, it can connect to the Perplexity API, which searches the web, academic literature, news, and other sources. Perplexity is a paid service with per-use pricing. The system walks you through connecting it when you're ready. Details and pricing are at [perplexity.ai](https://perplexity.ai).

Health Charted has no affiliation with any of the tools or services mentioned here. These are the tools that have worked best so far. As better tools become available, the system will integrate them.

## Built to improve

Health Charted is open source and community-supported. When improvements are available, the system offers to update itself at the start of a session. Updates enhance the system's capabilities without affecting your health data.

## Under the hood

For technical details on how the system is structured (data layers, entity indexing, schema design, update mechanics), see [ARCHITECTURE.md](ARCHITECTURE.md).

## License

[CC BY-NC-SA 4.0](LICENSE). Use it, adapt it, share it. Credit the project, keep it non-commercial, share adaptations under the same license.
