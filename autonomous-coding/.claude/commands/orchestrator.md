# Orchestrator - Claude Quickstarts Agent Manager

Manage and orchestrate multiple Claude agent types including autonomous coding, customer support, and financial data analysis.

## Usage

```
/orchestrator [agent-type] [action] [options]
```

## Agent Types

- **autonomous-coding** - Long-running autonomous coding agent for building complete applications
- **customer-support** - Customer support chat interface with knowledge base integration
- **financial-analyst** - Financial data analysis with interactive visualization

## Actions

### Autonomous Coding Agent Actions

#### start
Start a new autonomous coding session or resume an existing one.

**Usage:**
```
/orchestrator autonomous-coding start [project-dir] [--model MODEL] [--max-iterations N]
```

**Examples:**
```
/orchestrator autonomous-coding start ./my_project
/orchestrator autonomous-coding start ./my_project --model claude-sonnet-4-5-20250929
/orchestrator autonomous-coding start ./my_project --max-iterations 5
```

**What it does:**
1. Creates or identifies the project directory
2. Checks if this is a fresh start or continuation
3. Starts the autonomous agent with appropriate prompts
4. Continues running until max iterations or user interruption

#### status
Check the current status and progress of a project.

**Usage:**
```
/orchestrator autonomous-coding status [project-dir]
```

**Example:**
```
/orchestrator autonomous-coding status ./my_project
```

**What it does:**
1. Reads `feature_list.json` to count completed features
2. Shows recent git commits
3. Displays content from `claude-progress.txt`
4. Reports any running processes

#### setup
Initialize the project structure for a new autonomous coding project.

**Usage:**
```
/orchestrator autonomous-coding setup [project-dir]
```

**Example:**
```
/orchestrator autonomous-coding setup ./my_project
```

**What it does:**
1. Creates the project directory structure
2. Copies the app specification file
3. Prepares for the initializer agent to run

#### run-init
Run the initialization script (init.sh) for the project.

**Usage:**
```
/orchestrator autonomous-coding run-init [project-dir]
```

**Example:**
```
/orchestrator autonomous-coding run-init ./my_project
```

**What it does:**
1. Changes to the project directory
2. Makes init.sh executable
3. Runs the initialization script
4. Reports the application URL and access information

#### verify
Verify the current state of the project by running verification tests.

**Usage:**
```
/orchestrator autonomous-coding verify [project-dir]
```

**Example:**
```
/orchestrator autonomous-coding verify ./my_project
```

**What it does:**
1. Starts the application if not running
2. Runs a sample of passing tests to verify functionality
3. Reports any regressions or issues found
4. Takes screenshots for visual verification

### Customer Support Agent Actions

#### dev
Start the customer support agent in development mode.

**Usage:**
```
/orchestrator customer-support dev [--port PORT]
```

**Example:**
```
/orchestrator customer-support dev
/orchestrator customer-support dev --port 3001
```

**What it does:**
1. Navigates to customer-support-agent directory
2. Runs `npm run dev` to start development server
3. Opens the application at http://localhost:3000 (or specified port)

#### build
Build the customer support agent for production.

**Usage:**
```
/orchestrator customer-support build
```

**What it does:**
1. Navigates to customer-support-agent directory
2. Runs `npm run build` to create production build
3. Reports build status and output location

#### install
Install dependencies for the customer support agent.

**Usage:**
```
/orchestrator customer-support install
```

**What it does:**
1. Navigates to customer-support-agent directory
2. Runs `npm install` to install all dependencies
3. Reports installation status

#### setup-env
Guide through setting up environment variables.

**Usage:**
```
/orchestrator customer-support setup-env
```

**What it does:**
1. Checks for existing .env.local file
2. Provides instructions for required environment variables:
   - ANTHROPIC_API_KEY
   - BAWS_ACCESS_KEY_ID
   - BAWS_SECRET_ACCESS_KEY
3. Creates template .env.local if it doesn't exist

### Financial Data Analyst Actions

#### dev
Start the financial data analyst in development mode.

**Usage:**
```
/orchestrator financial-analyst dev [--port PORT]
```

**Example:**
```
/orchestrator financial-analyst dev
/orchestrator financial-analyst dev --port 3002
```

**What it does:**
1. Navigates to financial-data-analyst directory
2. Runs `npm run dev` to start development server
3. Opens the application at http://localhost:3000 (or specified port)

#### build
Build the financial data analyst for production.

**Usage:**
```
/orchestrator financial-analyst build
```

**What it does:**
1. Navigates to financial-data-analyst directory
2. Runs `npm run build` to create production build
3. Reports build status and output location

#### install
Install dependencies for the financial data analyst.

**Usage:**
```
/orchestrator financial-analyst install
```

**What it does:**
1. Navigates to financial-data-analyst directory
2. Runs `npm install` to install all dependencies
3. Reports installation status

#### setup-env
Guide through setting up environment variables.

**Usage:**
```
/orchestrator financial-analyst setup-env
```

**What it does:**
1. Checks for existing .env.local file
2. Provides instructions for required environment variable:
   - ANTHROPIC_API_KEY
3. Creates template .env.local if it doesn't exist

## Implementation

```bash
#!/bin/bash

# Get the root directory of the quickstarts repo
SCRIPT_DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" &> /dev/null && pwd )"
QUICKSTARTS_ROOT="$(cd "$SCRIPT_DIR/../../.." && pwd)"

AGENT_TYPE="${1:-autonomous-coding}"
ACTION="${2:-dev}"
PORT="${PORT:-3000}"

# Shift to get remaining arguments
shift 2

# Parse additional arguments
PROJECT_DIR="./autonomous_demo_project"
MODEL="claude-sonnet-4-5-20250929"
MAX_ITERATIONS=""

while [[ $# -gt 0 ]]; do
  case $1 in
    --port)
      PORT="$2"
      shift 2
      ;;
    --model)
      MODEL="$2"
      shift 2
      ;;
    --max-iterations)
      MAX_ITERATIONS="--max-iterations $2"
      shift 2
      ;;
    *)
      # Assume it's a project directory for autonomous-coding
      if [[ "$AGENT_TYPE" == "autonomous-coding" ]]; then
        PROJECT_DIR="$1"
      fi
      shift
      ;;
  esac
done

case "$AGENT_TYPE" in
  autonomous-coding)
    cd "$QUICKSTARTS_ROOT/autonomous-coding" || exit 1
    
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
        echo "2. Run: /orchestrator autonomous-coding start $PROJECT_DIR"
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
        echo "❌ Unknown action for autonomous-coding: $ACTION"
        echo "Available actions: start, status, setup, run-init, verify"
        exit 1
        ;;
    esac
    ;;
    
  customer-support)
    cd "$QUICKSTARTS_ROOT/customer-support-agent" || exit 1
    
    case "$ACTION" in
      dev)
        echo "🚀 Starting Customer Support Agent in development mode..."
        echo "Port: $PORT"
        PORT=$PORT npm run dev
        ;;
        
      build)
        echo "🏗️  Building Customer Support Agent for production..."
        npm run build
        echo "✅ Build complete"
        ;;
        
      install)
        echo "📦 Installing dependencies for Customer Support Agent..."
        npm install
        echo "✅ Dependencies installed"
        ;;
        
      setup-env)
        echo "⚙️  Setting up environment variables..."
        
        if [ -f ".env.local" ]; then
          echo "⚠️  .env.local already exists"
          echo "Current contents:"
          cat .env.local | sed 's/=.*/=***/'
        else
          echo "Creating .env.local template..."
          cat > .env.local << 'EOF'
ANTHROPIC_API_KEY=your_anthropic_api_key_here
BAWS_ACCESS_KEY_ID=your_aws_access_key_here
BAWS_SECRET_ACCESS_KEY=your_aws_secret_key_here
EOF
          echo "✅ Created .env.local template"
        fi
        
        echo ""
        echo "📝 Required environment variables:"
        echo "  - ANTHROPIC_API_KEY: Get from https://console.anthropic.com"
        echo "  - BAWS_ACCESS_KEY_ID: AWS access key for Bedrock"
        echo "  - BAWS_SECRET_ACCESS_KEY: AWS secret key for Bedrock"
        echo ""
        echo "Edit .env.local and add your keys, then run:"
        echo "  /orchestrator customer-support install"
        echo "  /orchestrator customer-support dev"
        ;;
        
      *)
        echo "❌ Unknown action for customer-support: $ACTION"
        echo "Available actions: dev, build, install, setup-env"
        exit 1
        ;;
    esac
    ;;
    
  financial-analyst)
    cd "$QUICKSTARTS_ROOT/financial-data-analyst" || exit 1
    
    case "$ACTION" in
      dev)
        echo "🚀 Starting Financial Data Analyst in development mode..."
        echo "Port: $PORT"
        PORT=$PORT npm run dev
        ;;
        
      build)
        echo "🏗️  Building Financial Data Analyst for production..."
        npm run build
        echo "✅ Build complete"
        ;;
        
      install)
        echo "📦 Installing dependencies for Financial Data Analyst..."
        npm install
        echo "✅ Dependencies installed"
        ;;
        
      setup-env)
        echo "⚙️  Setting up environment variables..."
        
        if [ -f ".env.local" ]; then
          echo "⚠️  .env.local already exists"
          echo "Current contents:"
          cat .env.local | sed 's/=.*/=***/'
        else
          echo "Creating .env.local template..."
          cat > .env.local << 'EOF'
ANTHROPIC_API_KEY=your_anthropic_api_key_here
EOF
          echo "✅ Created .env.local template"
        fi
        
        echo ""
        echo "📝 Required environment variable:"
        echo "  - ANTHROPIC_API_KEY: Get from https://console.anthropic.com"
        echo ""
        echo "Edit .env.local and add your key, then run:"
        echo "  /orchestrator financial-analyst install"
        echo "  /orchestrator financial-analyst dev"
        ;;
        
      *)
        echo "❌ Unknown action for financial-analyst: $ACTION"
        echo "Available actions: dev, build, install, setup-env"
        exit 1
        ;;
    esac
    ;;
    
  *)
    echo "❌ Unknown agent type: $AGENT_TYPE"
    echo ""
    echo "Available agent types:"
    echo "  autonomous-coding   - Long-running autonomous coding agent"
    echo "  customer-support    - Customer support chat interface"
    echo "  financial-analyst   - Financial data analysis tool"
    echo ""
    echo "Usage: /orchestrator [agent-type] [action] [options]"
    echo ""
    echo "Examples:"
    echo "  /orchestrator autonomous-coding start ./my_project"
    echo "  /orchestrator customer-support dev"
    echo "  /orchestrator financial-analyst dev --port 3002"
    exit 1
    ;;
esac
```

## Options

**For Autonomous Coding:**
- `--model MODEL` - Specify the Claude model to use (default: claude-sonnet-4-5-20250929)
- `--max-iterations N` - Limit the number of agent iterations (default: unlimited)

**For Customer Support & Financial Analyst:**
- `--port PORT` - Specify the port to run the development server (default: 3000)

## Examples

### Autonomous Coding Agent Examples

**Start a new autonomous coding project:**
```
/orchestrator autonomous-coding setup ./my_chat_app
# Edit ./my_chat_app/app_spec.txt with requirements
/orchestrator autonomous-coding start ./my_chat_app
```

**Resume an existing project:**
```
/orchestrator autonomous-coding status ./my_chat_app
/orchestrator autonomous-coding start ./my_chat_app
```

**Run with limited iterations for testing:**
```
/orchestrator autonomous-coding start ./my_chat_app --max-iterations 3
```

**Run the generated application:**
```
/orchestrator autonomous-coding run-init ./my_chat_app
```

### Customer Support Agent Examples

**Quick setup and start:**
```
/orchestrator customer-support setup-env
/orchestrator customer-support install
/orchestrator customer-support dev
```

**Build for production:**
```
/orchestrator customer-support build
```

**Run on a different port:**
```
/orchestrator customer-support dev --port 3001
```

### Financial Data Analyst Examples

**Quick setup and start:**
```
/orchestrator financial-analyst setup-env
/orchestrator financial-analyst install
/orchestrator financial-analyst dev
```

**Build for production:**
```
/orchestrator financial-analyst build
```

**Run on a different port:**
```
/orchestrator financial-analyst dev --port 3002
```

## Notes

### Autonomous Coding Agent
- Manages long-running autonomous coding sessions
- Each session has a fresh context window but continues from previous progress
- Progress is tracked via `feature_list.json` and git commits
- The agent will auto-continue between sessions with a 3-second delay
- Press Ctrl+C to pause; rerun the same command to resume

### Customer Support Agent
- Requires AWS Bedrock access and knowledge base setup
- Supports real-time thinking & debug information display
- Integrates with Amazon Bedrock Knowledge Bases for RAG
- Customizable UI with shadcn/ui components

### Financial Data Analyst
- Supports multiple file formats (text, PDF, images, CSV)
- Generates interactive charts based on context and data
- Uses Claude 3 Haiku & Claude 3.5 Sonnet models
- Built with Next.js 14 and Recharts for visualization

## Integration

This slash command provides a unified interface to manage all Claude quickstart agents:

**Prerequisites:**
- Claude Code CLI installed (`npm install -g @anthropic-ai/claude-code`)
- Python dependencies for autonomous-coding (`pip install -r requirements.txt`)
- Node.js 18+ for customer-support and financial-analyst
- ANTHROPIC_API_KEY environment variable set

## Troubleshooting

### Autonomous Coding Agent

**"Project directory does not exist"**
- Run `/orchestrator autonomous-coding setup [project-dir]` first to create the project structure

**"feature_list.json not found"**
- The initializer agent hasn't run yet. Use `/orchestrator autonomous-coding start` to begin initialization

**"init.sh not found"**
- The initializer agent creates this file. Complete the first session before trying to run the app

**"Command blocked by security hook"**
- Some bash commands are restricted by the security allowlist in `security.py`
- Check which commands are allowed and modify `security.py` if needed

### Customer Support Agent

**"Cannot find module" or dependency errors**
- Run `/orchestrator customer-support install` to install all dependencies

**"API key not set" errors**
- Ensure `.env.local` exists with valid ANTHROPIC_API_KEY
- Run `/orchestrator customer-support setup-env` for guidance

**AWS Bedrock connection issues**
- Verify AWS credentials (BAWS_ACCESS_KEY_ID, BAWS_SECRET_ACCESS_KEY)
- Ensure your AWS account has Bedrock access enabled
- Check that knowledge base IDs in the code match your Bedrock setup

**Port already in use**
- Use `--port` flag to specify a different port: `/orchestrator customer-support dev --port 3001`

### Financial Data Analyst

**"Cannot find module" or dependency errors**
- Run `/orchestrator financial-analyst install` to install all dependencies

**"API key not set" errors**
- Ensure `.env.local` exists with valid ANTHROPIC_API_KEY
- Run `/orchestrator financial-analyst setup-env` for guidance

**Chart not rendering**
- Ensure uploaded files contain valid data in supported formats
- Check browser console for JavaScript errors
- Try refreshing the page

**Port already in use**
- Use `--port` flag to specify a different port: `/orchestrator financial-analyst dev --port 3002`

## Quick Reference

**Autonomous Coding:**
```bash
/orchestrator autonomous-coding setup ./project
/orchestrator autonomous-coding start ./project
/orchestrator autonomous-coding status ./project
/orchestrator autonomous-coding verify ./project
/orchestrator autonomous-coding run-init ./project
```

**Customer Support:**
```bash
/orchestrator customer-support setup-env
/orchestrator customer-support install
/orchestrator customer-support dev
/orchestrator customer-support build
```

**Financial Analyst:**
```bash
/orchestrator financial-analyst setup-env
/orchestrator financial-analyst install
/orchestrator financial-analyst dev
/orchestrator financial-analyst build
```
