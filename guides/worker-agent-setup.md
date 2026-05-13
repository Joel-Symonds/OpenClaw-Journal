# Worker Agent Setup: Multi-Agent Architecture

## Overview
OpenClaw supports multiple agents with different roles, tool access, and security boundaries. This guide documents our three-worker setup for different task types.

## Architecture

### Main Agent (Kaida)
- **Role**: Primary assistant, coordinator
- **Tools**: Full access (exec, read/write/edit, gateway, sessions)
- **Sandbox**: Off (runs directly on host)
- **Purpose**: You talk to me, I decide which worker to spawn

### Worker 1: Heavy Research
- **Role**: Docker-sandboxed research agent
- **Tools**: Read/write/edit, web search, browser, image generation
- **Sandbox**: Docker (full isolation)
- **Purpose**: Web scraping, research, browser automation, heavy lifting

### Worker 2: Personal Tasks
- **Role**: File management, notes, personal tasks
- **Tools**: Read/write/edit, memory only
- **Sandbox**: Off (direct filesystem access needed)
- **Purpose**: Organize files, create notes, manage personal workspace

### Worker 3: System Monitoring
- **Role**: Observes system health, security checks
- **Tools**: Process status, memory read only
- **Sandbox**: Docker (read-only root, no network)
- **Purpose**: Check GPU temps, verify services running, flag issues

## Security Model
- Each worker has **minimal tools** for its role
- No worker can execute commands on the host without explicit approval
- Sandboxed workers run in Docker with restricted access
- Worker 3 (system monitoring) is read-only — can observe but never modify
- Main agent coordinates and decides when to spawn workers

## How It Works
1. **You ask me something**
2. **I decide** if it needs a worker (research, file task, system check)
3. **I spawn** the appropriate worker with its limited tools
4. **Worker completes** the task and reports back
5. **I give you** the result

No worker acts independently. They exist only when I spawn them for a specific task.

## Setup Process
1. Define worker roles in `openclaw.json`
2. Create workspace directories for each worker
3. Set tool permissions (minimal profile for most)
4. Configure sandboxing (Docker where needed)
5. Write SOUL.md files defining behavior and boundaries
6. Test each worker to verify permissions work

## Why Workers?
- **Security**: Limited tools mean limited damage if something goes wrong
- **Focus**: Each worker does one thing well
- **Isolation**: Sandboxed workers can't affect the host system
- **Coordination**: I manage when and how workers run

## Coming Soon
- Actual worker tasks and examples
- Security hardening notes
- Performance tuning for sandboxed agents
