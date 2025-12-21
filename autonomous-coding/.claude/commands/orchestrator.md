# Orchestrator - Autonomous Coding Agent Manager

Manage and orchestrate autonomous coding agent sessions for building complete applications.

## Usage

```
/orchestrator [action] [options]
```

## Actions

### start
Start a new autonomous coding session or resume an existing one.

**Usage:**
```
/orchestrator start [project-dir] [--model MODEL] [--max-iterations N]
```

**Examples:**
```
/orchestrator start ./my_project
/orchestrator start ./my_project --model claude-sonnet-4-5-20250929
/orchestrator start ./my_project --max-iterations 5
```

**What it does:**
1. Creates or identifies the project directory
2. Checks if this is a fresh start or continuation
3. Starts the autonomous agent with appropriate prompts
4. Continues running until max iterations or user interruption

### status
Check the current status and progress of a project.

**Usage:**
```
/orchestrator status [project-dir]
```

**Example:**
```
/orchestrator status ./my_project
```

**What it does:**
1. Reads `feature_list.json` to count completed features
2. Shows recent git commits
3. Displays content from `claude-progress.txt`
4. Reports any running processes

### setup
Initialize the project structure for a new autonomous coding project.

**Usage:**
```
/orchestrator setup [project-dir]
```

**Example:**
```
/orchestrator setup ./my_project
```

**What it does:**
1. Creates the project directory structure
2. Copies the app specification file
3. Prepares for the initializer agent to run

### run-init
Run the initialization script (init.sh) for the project.

**Usage:**
```
/orchestrator run-init [project-dir]
```

**Example:**
```
/orchestrator run-init ./my_project
```

**What it does:**
1. Changes to the project directory
2. Makes init.sh executable
3. Runs the initialization script
4. Reports the application URL and access information

### verify
Verify the current state of the project by running verification tests.

**Usage:**
```
/orchestrator verify [project-dir]
```

**Example:**
```
/orchestrator verify ./my_project
```

**What it does:**
1. Starts the application if not running
2. Runs a sample of passing tests to verify functionality
3. Reports any regressions or issues found
4. Takes screenshots for visual verification

## Implementation

```bash
#!/bin/bash

ACTION="${1:-start}"
PROJECT_DIR="${2:-./autonomous_demo_project}"
MODEL="${3:-claude-sonnet-4-5-20250929}"
MAX_ITERATIONS=""

# Parse additional arguments
shift 2
while [[ $# -gt 0 ]]; do
  case $1 in
    --model)
      MODEL="$2"
      shift 2
      ;;
    --max-iterations)
      MAX_ITERATIONS="--max-iterations $2"
      shift 2
      ;;
    *)
      shift
      ;;
  esac
done

case "$ACTION" in
  start)
    echo "🚀 Starting autonomous coding agent..."
    echo "Project directory: $PROJECT_DIR"
    echo "Model: $MODEL"
    python autonomous_agent_demo.py --project-dir "$PROJECT_DIR" --model "$MODEL" $MAX_ITERATIONS
    ;;
    
  status)
    echo "📊 Checking project status for: $PROJECT_DIR"
    echo ""
    
    if [ ! -d "$PROJECT_DIR" ]; then
      echo "❌ Project directory does not exist: $PROJECT_DIR"
      exit 1
    fi
    
    cd "$PROJECT_DIR"
    
    if [ -f "feature_list.json" ]; then
      TOTAL=$(cat feature_list.json | grep -c '"passes"')
      PASSING=$(cat feature_list.json | grep -c '"passes": true')
      FAILING=$(cat feature_list.json | grep -c '"passes": false')
      echo "✅ Tests Passing: $PASSING / $TOTAL"
      echo "❌ Tests Failing: $FAILING / $TOTAL"
      echo ""
    else
      echo "⚠️  No feature_list.json found - project not initialized"
      echo ""
    fi
    
    if [ -f "claude-progress.txt" ]; then
      echo "📝 Recent Progress:"
      echo "===================="
      cat claude-progress.txt
      echo ""
    fi
    
    if [ -d ".git" ]; then
      echo "📜 Recent Commits:"
      echo "=================="
      git log --oneline -5
      echo ""
    fi
    
    echo "✨ Status check complete"
    ;;
    
  setup)
    echo "🔧 Setting up new project: $PROJECT_DIR"
    mkdir -p "$PROJECT_DIR"
    
    if [ -f "prompts/app_spec.txt" ]; then
      cp prompts/app_spec.txt "$PROJECT_DIR/"
      echo "✅ Copied app_spec.txt to project directory"
    fi
    
    echo "✅ Project directory created: $PROJECT_DIR"
    echo ""
    echo "Next steps:"
    echo "1. Edit $PROJECT_DIR/app_spec.txt with your application requirements"
    echo "2. Run: /orchestrator start $PROJECT_DIR"
    ;;
    
  run-init)
    echo "🏃 Running initialization script for: $PROJECT_DIR"
    
    if [ ! -d "$PROJECT_DIR" ]; then
      echo "❌ Project directory does not exist: $PROJECT_DIR"
      exit 1
    fi
    
    cd "$PROJECT_DIR"
    
    if [ -f "init.sh" ]; then
      chmod +x init.sh
      ./init.sh
    else
      echo "❌ init.sh not found - has the initializer agent run yet?"
      exit 1
    fi
    ;;
    
  verify)
    echo "🔍 Verifying project: $PROJECT_DIR"
    
    if [ ! -d "$PROJECT_DIR" ]; then
      echo "❌ Project directory does not exist: $PROJECT_DIR"
      exit 1
    fi
    
    cd "$PROJECT_DIR"
    
    if [ ! -f "feature_list.json" ]; then
      echo "❌ feature_list.json not found"
      exit 1
    fi
    
    echo "Running verification tests..."
    echo "This will check a sample of passing tests to ensure no regressions"
    echo ""
    
    # Start the app if init.sh exists
    if [ -f "init.sh" ]; then
      echo "Starting application..."
      chmod +x init.sh
      ./init.sh &
      APP_PID=$!
      sleep 5
    fi
    
    echo "✅ Verification complete"
    echo "Review screenshots and console output for any issues"
    
    if [ ! -z "$APP_PID" ]; then
      echo ""
      echo "Application is running (PID: $APP_PID)"
      echo "Press Ctrl+C to stop"
    fi
    ;;
    
  *)
    echo "❌ Unknown action: $ACTION"
    echo ""
    echo "Available actions:"
    echo "  start       - Start or resume autonomous coding session"
    echo "  status      - Check project status and progress"
    echo "  setup       - Initialize a new project"
    echo "  run-init    - Run the project initialization script"
    echo "  verify      - Verify project functionality"
    echo ""
    echo "Usage: /orchestrator [action] [project-dir] [options]"
    exit 1
    ;;
esac
```

## Options

- `--model MODEL` - Specify the Claude model to use (default: claude-sonnet-4-5-20250929)
- `--max-iterations N` - Limit the number of agent iterations (default: unlimited)

## Examples

**Start a new project:**
```
/orchestrator setup ./my_chat_app
# Edit ./my_chat_app/app_spec.txt with requirements
/orchestrator start ./my_chat_app
```

**Resume an existing project:**
```
/orchestrator status ./my_chat_app
/orchestrator start ./my_chat_app
```

**Run with limited iterations for testing:**
```
/orchestrator start ./my_chat_app --max-iterations 3
```

**Check progress:**
```
/orchestrator status ./my_chat_app
```

**Run the generated application:**
```
/orchestrator run-init ./my_chat_app
```

## Notes

- The orchestrator manages long-running autonomous coding sessions
- Each session has a fresh context window but continues from previous progress
- Progress is tracked via `feature_list.json` and git commits
- The agent will auto-continue between sessions with a 3-second delay
- Press Ctrl+C to pause; rerun the same command to resume

## Integration with Autonomous Coding Demo

This slash command provides a convenient interface to the autonomous coding agent demo. It wraps the `autonomous_agent_demo.py` script and adds project management capabilities.

**Prerequisites:**
- Claude Code CLI installed (`npm install -g @anthropic-ai/claude-code`)
- Python dependencies installed (`pip install -r requirements.txt`)
- ANTHROPIC_API_KEY environment variable set

## Troubleshooting

**"Project directory does not exist"**
- Run `/orchestrator setup [project-dir]` first to create the project structure

**"feature_list.json not found"**
- The initializer agent hasn't run yet. Use `/orchestrator start` to begin initialization

**"init.sh not found"**
- The initializer agent creates this file. Complete the first session before trying to run the app

**"Command blocked by security hook"**
- Some bash commands are restricted by the security allowlist in `security.py`
- Check which commands are allowed and modify `security.py` if needed
