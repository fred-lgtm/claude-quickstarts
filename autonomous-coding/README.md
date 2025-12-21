# Autonomous Coding Agent Demo

A minimal harness demonstrating long-running autonomous coding with the Claude Agent SDK. This demo implements a two-agent pattern (initializer + coding agent) that can build complete applications over multiple sessions.

## Prerequisites

**Required:** Install the latest versions of both Claude Code and the Claude Agent SDK:

```bash
# Install Claude Code CLI (latest version required)
npm install -g @anthropic-ai/claude-code

# Install Python dependencies
pip install -r requirements.txt
```

Verify your installations:
```bash
claude --version  # Should be latest version
pip show claude-code-sdk  # Check SDK is installed
```

**API Key:** Set your Anthropic API key:
```bash
export ANTHROPIC_API_KEY='your-api-key-here'
```

## Quick Start

### Using the /orchestrator Slash Command (Recommended)

If you're using Claude Code CLI, you can use the `/orchestrator` slash command for easy project management. The orchestrator now supports all Claude quickstart agents:

```bash
# Autonomous Coding Agent - Start a new project
/orchestrator autonomous-coding start ./my_project

# Autonomous Coding Agent - Check project status
/orchestrator autonomous-coding status ./my_project

# Autonomous Coding Agent - For testing with limited iterations
/orchestrator autonomous-coding start ./my_project --max-iterations 3

# Customer Support Agent - Quick setup
/orchestrator customer-support setup-env
/orchestrator customer-support install
/orchestrator customer-support dev

# Financial Data Analyst - Quick setup
/orchestrator financial-analyst setup-env
/orchestrator financial-analyst install
/orchestrator financial-analyst dev
```

See [Slash Command Reference](#slash-command-reference) below for all available commands.

### Using Python Directly

```bash
python autonomous_agent_demo.py --project-dir ./my_project
```

For testing with limited iterations:
```bash
python autonomous_agent_demo.py --project-dir ./my_project --max-iterations 3
```

## Important Timing Expectations

> **Warning: This demo takes a long time to run!**

- **First session (initialization):** The agent generates a `feature_list.json` with 200 test cases. This takes several minutes and may appear to hang - this is normal. The agent is writing out all the features.

- **Subsequent sessions:** Each coding iteration can take **5-15 minutes** depending on complexity.

- **Full app:** Building all 200 features typically requires **many hours** of total runtime across multiple sessions.

**Tip:** The 200 features parameter in the prompts is designed for comprehensive coverage. If you want faster demos, you can modify `prompts/initializer_prompt.md` to reduce the feature count (e.g., 20-50 features for a quicker demo).

## How It Works

### Two-Agent Pattern

1. **Initializer Agent (Session 1):** Reads `app_spec.txt`, creates `feature_list.json` with 200 test cases, sets up project structure, and initializes git.

2. **Coding Agent (Sessions 2+):** Picks up where the previous session left off, implements features one by one, and marks them as passing in `feature_list.json`.

### Session Management

- Each session runs with a fresh context window
- Progress is persisted via `feature_list.json` and git commits
- The agent auto-continues between sessions (3 second delay)
- Press `Ctrl+C` to pause; run the same command to resume

## Security Model

This demo uses a defense-in-depth security approach (see `security.py` and `client.py`):

1. **OS-level Sandbox:** Bash commands run in an isolated environment
2. **Filesystem Restrictions:** File operations restricted to the project directory only
3. **Bash Allowlist:** Only specific commands are permitted:
   - File inspection: `ls`, `cat`, `head`, `tail`, `wc`, `grep`
   - Node.js: `npm`, `node`
   - Version control: `git`
   - Process management: `ps`, `lsof`, `sleep`, `pkill` (dev processes only)

Commands not in the allowlist are blocked by the security hook.

## Project Structure

```
autonomous-coding/
├── autonomous_agent_demo.py  # Main entry point
├── agent.py                  # Agent session logic
├── client.py                 # Claude SDK client configuration
├── security.py               # Bash command allowlist and validation
├── progress.py               # Progress tracking utilities
├── prompts.py                # Prompt loading utilities
├── prompts/
│   ├── app_spec.txt          # Application specification
│   ├── initializer_prompt.md # First session prompt
│   └── coding_prompt.md      # Continuation session prompt
└── requirements.txt          # Python dependencies
```

## Generated Project Structure

After running, your project directory will contain:

```
my_project/
├── feature_list.json         # Test cases (source of truth)
├── app_spec.txt              # Copied specification
├── init.sh                   # Environment setup script
├── claude-progress.txt       # Session progress notes
├── .claude_settings.json     # Security settings
└── [application files]       # Generated application code
```

## Running the Generated Application

After the agent completes (or pauses), you can run the generated application:

```bash
cd generations/my_project

# Run the setup script created by the agent
./init.sh

# Or manually (typical for Node.js apps):
npm install
npm run dev
```

The application will typically be available at `http://localhost:3000` or similar (check the agent's output or `init.sh` for the exact URL).

## Command Line Options

| Option | Description | Default |
|--------|-------------|---------|
| `--project-dir` | Directory for the project | `./autonomous_demo_project` |
| `--max-iterations` | Max agent iterations | Unlimited |
| `--model` | Claude model to use | `claude-sonnet-4-5-20250929` |

## Slash Command Reference

The `/orchestrator` slash command provides a unified interface for managing all Claude quickstart agents when using Claude Code CLI.

### Command Structure

```bash
/orchestrator [agent-type] [action] [options]
```

**Agent Types:**
- `autonomous-coding` - Long-running autonomous coding agent
- `customer-support` - Customer support chat interface
- `financial-analyst` - Financial data analysis tool

### Autonomous Coding Commands

#### `/orchestrator autonomous-coding start`
Start a new autonomous coding session or resume an existing one.

```bash
/orchestrator autonomous-coding start [project-dir] [--model MODEL] [--max-iterations N]
```

**Examples:**
```bash
/orchestrator autonomous-coding start ./my_project
/orchestrator autonomous-coding start ./my_project --model claude-sonnet-4-5-20250929
/orchestrator autonomous-coding start ./my_project --max-iterations 5
```

#### `/orchestrator autonomous-coding status`
Check the current status and progress of a project.

```bash
/orchestrator autonomous-coding status [project-dir]
```

Shows:
- Number of passing vs. failing tests
- Recent progress notes
- Recent git commits

#### `/orchestrator autonomous-coding setup`
Initialize a new project structure.

```bash
/orchestrator autonomous-coding setup [project-dir]
```

Creates the project directory and copies the app specification template.

#### `/orchestrator autonomous-coding run-init`
Run the initialization script (init.sh) to start the generated application.

```bash
/orchestrator autonomous-coding run-init [project-dir]
```

#### `/orchestrator autonomous-coding verify`
Verify the project by running a sample of passing tests.

```bash
/orchestrator autonomous-coding verify [project-dir]
```

### Customer Support Agent Commands

#### `/orchestrator customer-support dev`
Start the customer support agent in development mode.

```bash
/orchestrator customer-support dev [--port PORT]
```

#### `/orchestrator customer-support setup-env`
Guide through setting up environment variables (.env.local).

```bash
/orchestrator customer-support setup-env
```

#### `/orchestrator customer-support install`
Install dependencies for the customer support agent.

```bash
/orchestrator customer-support install
```

#### `/orchestrator customer-support build`
Build the customer support agent for production.

```bash
/orchestrator customer-support build
```

### Financial Data Analyst Commands

#### `/orchestrator financial-analyst dev`
Start the financial data analyst in development mode.

```bash
/orchestrator financial-analyst dev [--port PORT]
```

#### `/orchestrator financial-analyst setup-env`
Guide through setting up environment variables (.env.local).

```bash
/orchestrator financial-analyst setup-env
```

#### `/orchestrator financial-analyst install`
Install dependencies for the financial data analyst.

```bash
/orchestrator financial-analyst install
```

#### `/orchestrator financial-analyst build`
Build the financial data analyst for production.

```bash
/orchestrator financial-analyst build
```

### Usage Examples

**Autonomous Coding - Create and start a new project:**
```bash
/orchestrator autonomous-coding setup ./my_chat_app
# Edit ./my_chat_app/app_spec.txt with your requirements
/orchestrator autonomous-coding start ./my_chat_app
```

**Autonomous Coding - Check progress and resume:**
```bash
/orchestrator autonomous-coding status ./my_chat_app
/orchestrator autonomous-coding start ./my_chat_app
```

**Customer Support - Quick start:**
```bash
/orchestrator customer-support setup-env
/orchestrator customer-support install
/orchestrator customer-support dev
```

**Financial Analyst - Quick start:**
```bash
/orchestrator financial-analyst setup-env
/orchestrator financial-analyst install
/orchestrator financial-analyst dev
```

For complete documentation, see `.claude/commands/orchestrator.md`.

## Customization

### Changing the Application

Edit `prompts/app_spec.txt` to specify a different application to build.

### Adjusting Feature Count

Edit `prompts/initializer_prompt.md` and change the "200 features" requirement to a smaller number for faster demos.

### Modifying Allowed Commands

Edit `security.py` to add or remove commands from `ALLOWED_COMMANDS`.

## Troubleshooting

**"Appears to hang on first run"**
This is normal. The initializer agent is generating 200 detailed test cases, which takes significant time. Watch for `[Tool: ...]` output to confirm the agent is working.

**"Command blocked by security hook"**
The agent tried to run a command not in the allowlist. This is the security system working as intended. If needed, add the command to `ALLOWED_COMMANDS` in `security.py`.

**"API key not set"**
Ensure `ANTHROPIC_API_KEY` is exported in your shell environment.

## License

Internal Anthropic use.
