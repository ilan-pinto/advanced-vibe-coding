# Advanced Vibe Coding - Innovation Lab

A demonstration repository showcasing parallel multi-agent development approaches for building modern web applications using Claude AI.

## Overview

This repository demonstrates different approaches to AI-assisted collaborative development through two hands-on labs. Both labs solve the same problem - building a web dashboard for an e-commerce platform - but use different development methodologies to highlight the contrast between manual and automated multi-agent workflows.

**Educational Goals:**
- Learn parallel development patterns with AI agents
- Compare manual vs automated workflow orchestration
- Explore git worktrees for isolated development branches
- Master tmux for terminal multiplexing
- Understand microservices architecture patterns

## Project Requirements

### User Story

Build a web dashboard for an e-commerce platform that displays data from an API gateway. The dashboard provides administrators with real-time visibility into three core business areas:

- **Products**: Grid view showing inventory and prices
- **Users**: List view showing customer accounts
- **Orders**: Table view showing transaction statuses

This is a read-only dashboard focused on data visualization and monitoring, not data manipulation.

## Prerequisites

Before starting either lab, ensure you have the following installed:

### Required for Both Labs
- **Claude CLI**: [Install Claude Code](https://docs.claude.com/claude-code)
- **Git**: Version control system

### Required for Lab 2
- **tmux**: Terminal multiplexer
- **Podman**: Container engine
- **podman-compose**: Container orchestration tool

### Optional (to run the applications)
- **Node.js**: For JavaScript/TypeScript services
- **Python**: For Python services
- **Go**: For Go services
- **Rust**: For Rust services

## Labs

### Lab 1: Manual Multi-Agent Development

**Approach**: Manual parallel multi-agent development with Claude

**Directory**: `./lab1/`

**Tech Stack**:
- Node.js/Express (Product & Order Services, API Gateway)
- Python/FastAPI (User Service)
- React with Tailwind CSS (Frontend)
- Docker (Containerization)

**Instructions**:

1. Open 6 separate terminal windows
2. Start Claude CLI in each terminal
3. Open `lab1/demo1-prompts.md` to view the prompts
4. Copy and paste prompts into each terminal in sequence:
   - **Terminal 1**: Product Service (Node.js/Express on port 8001)
   - **Terminal 2**: User Service (Python/FastAPI on port 8002)
   - **Terminal 3**: Order Service (Node.js/Express on port 8003)
   - **Terminal 4**: API Gateway (Node.js/Express on port 8000)
   - **Terminal 5**: Frontend UI (React with Tailwind CSS on port 3001)
   - **Terminal 6**: Docker containerization setup
5. Monitor each terminal as Claude builds the respective component

**Learning Outcome**: Understanding manual coordination of parallel development tasks and multi-terminal workflows.

---

### Lab 2: Automated Multi-Agent Development

**Approach**: Automated workflow orchestration using the spawn command with tmux and git worktrees

**Directory**: `./lab2/`

**Tech Stack**:
- Python/FastAPI (Product Service)
- Go/Gin (User Service)
- Rust/Actix (Order Service)
- Node.js/Express (API Gateway)
- TypeScript/React with PatternFly 5 (Frontend)
- Podman (Containerization)

**Instructions**:

1. **Setup the spawn command**:
   ```bash
   # Copy the spawn command to lab2's .claude/commands directory
   cp custom-commands/spawn.md lab2/.claude/commands/spawn.md
   ```

2. **Navigate to lab2 and start Claude**:
   ```bash
   cd lab2
   claude
   ```

3. **Review the specifications** (optional but recommended):
   - `spec.md`: Feature specification with user stories
   - `plan.md`: Implementation plan with 7-agent workflow
   - `tasks.md`: Task breakdown with dependencies
   - `contracts/api-gateway.yaml`: OpenAPI contract

4. **Execute the spawn command**:
   ```
   /spawn
   ```

5. **Watch the automation**:
   - The spawn command will automatically:
     - Create git worktrees for each agent
     - Launch Claude agents in separate tmux sessions
     - Execute tasks sequentially with dependency management
     - Track progress in `completed tasks.md`

**Learning Outcome**: Automated multi-agent orchestration, git worktree management, and tmux session handling.

---

## Basic Tmux Commands

These commands are essential for Lab 2 to monitor and interact with the automated agents:

| Command | Description |
|---------|-------------|
| `tmux ls` | List all tmux sessions |
| `tmux attach -t <session>` | Attach to a specific session |
| `Ctrl+b d` | Detach from current session |
| `Ctrl+b c` | Create new window in session |
| `Ctrl+b n` | Navigate to next window |
| `Ctrl+b p` | Navigate to previous window |
| `Ctrl+b w` | List all windows |
| `Ctrl+b ,` | Rename current window |
| `tmux kill-session -t <session>` | Kill a specific session |
| `tmux kill-server` | Kill all tmux sessions |

**Pro Tip**: Use `tmux attach` to watch each agent's progress in real-time.

## Getting Started

```bash
# Clone the repository
git clone <repository-url>
cd advanced-vibe-coding

# For Lab 1
cd lab1
# Open 6 terminals and follow Lab 1 instructions above

# For Lab 2
cd lab2
cp ../custom-commands/spawn.md .claude/commands/spawn.md
claude
# Then run: /spawn
```

## Comparison

This innovation lab allows you to compare and contrast different AI-assisted development approaches:

| Aspect | Lab 1 (Manual) | Lab 2 (Automated) |
|--------|----------------|-------------------|
| **Orchestration** | Manual copy/paste | Automated via spawn command |
| **Terminals** | 6 separate terminals | 1 terminal + tmux sessions |
| **Agents** | 6 parallel agents | 7 sequential agents |
| **Tech Stack** | Node.js-heavy, simple | Polyglot (Python/Go/Rust/Node) |
| **Git Workflow** | Single branch | Git worktrees per agent |
| **Complexity** | Beginner-friendly | Advanced |
| **UI Framework** | Tailwind CSS | PatternFly 5 |
| **Containers** | Docker | Podman |
| **Task Tracking** | Manual | Automated with completed tasks.md |
| **Dependencies** | Ad-hoc | Declared in tasks.md |

**Key Insights**:
- **Speed**: Lab 2's automation reduces setup time significantly
- **Scalability**: Lab 2's approach scales better for larger projects
- **Learning Curve**: Lab 1 is easier to understand initially
- **Real-world Application**: Lab 2 demonstrates enterprise-grade patterns
