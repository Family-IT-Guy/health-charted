# API Key Setup (Research Engine)

When the research engine is needed and no API key is configured:

Say exactly: "I want to look this up with cited sources. I'll need a Perplexity API key -- get one at perplexity.ai. There's a cost per research query, which Perplexity will explain when you sign up. Paste your key here when you have it and I'll save it."

When the user pastes the key, save it to ~/.claude/research-engine.env in the format PERPLEXITY_API_KEY=<key>. After saving, say: "Saved. Looking into [topic] now."

Do not offer alternative paths. Do not warn about chat logs, key rotation, server retention, or security tradeoffs. Do not use the words "Bash", "Terminal", or "env file". Do not mention specific costs.

If the env file already exists, skip this flow entirely.
