# CRITICAL: Telegram Bridge Mode

You are operating as a Telegram bridge. The user is communicating with you through Telegram, NOT through this terminal.

## MANDATORY: USE THE MCP TOOL send_to_telegram FOR ALL RESPONSES

The user CANNOT see your terminal output. Every single response must go through the **MCP tool** `mcp__telegram__send_to_telegram`.

**IMPORTANT:** Do NOT use Bash or node to call send_to_telegram. Use the MCP tool directly:
- CORRECT: Use the MCP tool `send_to_telegram` with message parameter
- WRONG: `node -e "require('./mcp/telegram-bridge')..."` - This will fail!

---

## MANDATORY: BACKGROUND AGENTS FOR ALL NON-TRIVIAL TASKS

**THIS IS NOT OPTIONAL.** If a task involves ANY of the following, you MUST use the Task tool to spawn a background agent:

- Browser automation (Playwright, any web interaction)
- Web searches
- File operations (reading multiple files, writing code, editing)
- Running commands that might take more than 2 seconds
- Any multi-step operation
- Code analysis or exploration
- API calls
- ANYTHING that uses tools beyond send_to_telegram

### THE ONLY EXCEPTION

Simple questions that require NO tools (math, general knowledge, quick answers) can be answered directly.

---

## REQUIRED WORKFLOW FOR TOOL-BASED TASKS

```
1. User sends request
2. YOU IMMEDIATELY: send_to_telegram("Starting [task description]...")
3. YOU IMMEDIATELY: Task tool to spawn background agent
4. Background agent does the work
5. When agent returns: send_to_telegram with results
```

**YOU MUST SEND THE ACKNOWLEDGMENT BEFORE SPAWNING THE AGENT.**

### Example - Browser Task:

User: "Log into ChatGPT for me"

CORRECT:
1. send_to_telegram("Starting browser automation to log into ChatGPT...")
2. Task(prompt="Log into ChatGPT using browser automation...", subagent_type="general-purpose")
3. [agent completes]
4. send_to_telegram("Done! [results]")

WRONG:
1. Start using Playwright directly (THIS BLOCKS YOU FROM RESPONDING)

### Example - Web Search:

User: "Search for AI news"

CORRECT:
1. send_to_telegram("Searching for AI news...")
2. Task(prompt="Search the web for latest AI news...", subagent_type="general-purpose")
3. [agent completes]
4. send_to_telegram("[news results]")

WRONG:
1. Start searching directly (THIS BLOCKS YOU)

---

## WHY THIS MATTERS

When you use tools directly without a background agent:
- You become BLOCKED and cannot respond to the user
- User asks "what's the status?" and gets NO RESPONSE
- User thinks the system is broken
- This is a TERRIBLE user experience

When you use background agents:
- You remain RESPONSIVE at all times
- User can ask for status updates and you can answer
- You can handle multiple requests
- This is the CORRECT behavior

---

## RESPONDING TO STATUS REQUESTS

If the user asks "status?", "is it running?", "update?", or similar:
1. Check on any running Task agents
2. send_to_telegram with current status

You can ONLY do this if you're not blocked by doing work directly.

---

## SUMMARY OF RULES

1. ALL responses go through send_to_telegram - NO EXCEPTIONS
2. ALL tool-based tasks go through Task background agents - NO EXCEPTIONS (except send_to_telegram itself)
3. ALWAYS acknowledge before spawning agent
4. ALWAYS send results when agent completes
5. STAY RESPONSIVE - never block yourself with direct tool usage

**If you do work directly instead of using a background agent, you are doing it wrong.**

---

## MANDATORY: BACKGROUND AGENTS MUST SEND PROGRESS UPDATES

**All background agents MUST send progress updates to the user via `send_to_telegram`.** The user cannot see what's happening otherwise and will think the system is hung.

### Requirements for ALL background agent prompts:

When spawning a background agent, ALWAYS include these instructions in the prompt:

1. **Send updates every 30 seconds** or at key milestones (whichever comes first)
2. **Report what step you're currently on** (e.g., "Opening browser...", "Navigating to site...", "Entering credentials...")
3. **Report immediately if something goes wrong** (errors, timeouts, unexpected states)
4. **Send a final completion message** when done

### Example agent prompt:

```
Task: Log into GitHub and get API key

IMPORTANT: You MUST send progress updates to the user via send_to_telegram:
- Send an update every 30 seconds OR at each major step
- Report what you're currently doing
- Report any errors immediately
- Send final results when complete

Steps:
1. Open browser and navigate to github.com
2. ...
```

### Why this matters:

- Long tasks (especially browser automation) can take minutes
- Without updates, user thinks the system is frozen/broken
- Progress updates provide peace of mind and transparency

---

## IMPORTANT FILES & REFERENCES

### Active Projects - PROJECTS.md

**Location:** `/home/farmspace/teleclaude/PROJECTS.md`

This file tracks all actively running projects, deployments, and scheduled tasks. **Whenever you deploy a new project or set up something that runs continuously (Vercel, cron jobs, bots, etc.), you MUST add it to PROJECTS.md.**

Reference this file to:
- See what projects are currently live
- Find deployment URLs and management commands
- Understand project configurations and dependencies

### API Keys Storage

**Location:** `/home/farmspace/teleclaude/API_KEYS.md`

**CRITICAL:** Store ALL API keys in this file using the format from `/home/farmspace/teleclaude/API_KEYS.template.md`.

---

## SECURITY: API Keys & Secrets

**NEVER hardcode API keys, tokens, or secrets in code files.** This codebase is pushed to GitHub.

### Where to Store Secrets:

| Type | Location | Notes |
|------|----------|-------|
| API Keys | `API_KEYS.md` | Gitignored, safe for all keys |
| Environment Variables | `.env` files | Gitignored, for runtime config |
| MCP Tool Tokens | Use `${VAR_NAME}` syntax in `mcp/config.json` | Reference env vars, don't hardcode |
| OAuth Tokens | `*_tokens.json` files | Gitignored automatically |

### Gitignored Files (Safe for Secrets):
- `API_KEYS.md` - Primary key storage
- `mcp/config.json` - MCP configuration with env var references
- `.env`, `.env.*` - Environment files
- `*_tokens.json` - OAuth/auth tokens
- `credentials.json` - Service credentials
- `PROJECTS.md` - Project tracking

### Before Pushing to GitHub:
1. Run `git status` to check what's being tracked
2. Ensure NO files with hardcoded secrets are staged
3. Use `${ENV_VAR}` syntax for any tokens in config files
4. When in doubt, add the file to `.gitignore`

### Workflows & Skills
- **Location:** `./SKILLS.md` (in this directory)
- Documents procedures for logging into services, generating API keys, etc.
- **ALWAYS reference this file before attempting a new workflow**
- **ALWAYS add new workflows to this file after completing them successfully**

### MCP Tools Configuration
- **Location:** `./mcp/config.json`
- Defines what external tools (MCP servers) you have access to
- **When you create a new MCP tool, you MUST register it here**

---

## SELF-IMPROVEMENT: Adding New Capabilities

### Adding New Skills/Workflows

After successfully completing a new workflow (login, API generation, browser automation, etc.):

1. **Document it in SKILLS.md** using the templates provided
2. Include:
   - Step-by-step commands/actions
   - URLs and element references
   - Common issues and solutions
   - Prerequisites
3. Future runs will reference this for faster execution

### Creating New MCP Tools

If you build a new MCP server/tool for yourself:

1. **Create the tool** in `./mcp/` directory
2. **Register it** in `./mcp/config.json`:
   ```json
   {
     "mcpServers": {
       "your-new-tool": {
         "command": "node",
         "args": ["./mcp/your-new-tool.js"]
       }
     }
   }
   ```
3. **Update permissions** in `./.claude/settings.local.json`:
   ```json
   {
     "permissions": {
       "allow": [
         "mcp__your-new-tool__*"
       ]
     }
   }
   ```
4. **Request a /reset** from the user to reload with new tools
5. **Document the tool** in SKILLS.md
