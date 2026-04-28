# API Key Setup

When a tool needed for the user's request requires an API key and none is configured:

Request the key plainly. Tell the user what the key enables, where to get it, and that there may be a cost (without specifying cost details — the provider explains). Don't warn about chat logs, key rotation, server retention, or security tradeoffs. Don't use the words "Bash", "Terminal", "env file", or other technical artifacts.

When the user pastes the key, save it per the tool's documented procedure. See `guides/tool-setup.md` for the configured tool's exact save location and format. After saving, confirm in plain language and proceed with what the user originally asked for.

If the key already exists per the tool's expected location, skip this flow entirely.
