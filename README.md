# dev

Development workflow skills for Claude Code.

## Skills

| Skill | Description |
|-------|-------------|
| `auto-dev` | PRD-driven autonomous dev loop with zero human intervention |
| `chat-dev` | Interactive dev loop — discuss requirements, then execute |
| `task-dev` | TickTick-driven dev loop — read tasks and process one by one |
| `prd-decompose` | Break PRD into work units with DAG and file-overlap analysis |
| `auto-dev-loop` | Resume `auto-dev` across sessions via PROGRESS.md |
| `task-dev-loop` | Resume `task-dev` across sessions via TASK_PROGRESS.md |

## Install

```bash
claude plugin marketplace add TungYuChiang/dev
```

Then in Claude Code:

```
/plugin install dev@TungYuChiang-dev
```

## Usage

Skills are namespaced as `/dev:<skill-name>`:

```
/dev:chat-dev
/dev:auto-dev
/dev:task-dev
```
