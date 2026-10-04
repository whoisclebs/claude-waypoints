# claude-waypoints

![status: design](https://img.shields.io/badge/status-design-orange)
![platform: Claude Code mod](https://img.shields.io/badge/platform-Claude%20Code%20mod-blue)

Plan work as a graph of waypoints. Let Claude Code walk the route.

`claude-waypoints` is a Claude Code mod that turns a large goal into a dependency graph (DAG) of small tasks, runs the independent ones in parallel with subagents, and only marks a task done after its checks pass.

> **Status: design stage.** There is nothing to install yet. This README describes what the mod is meant to do, so the scope is clear before the code lands. It is updated as each part ships.

## Why

A single Claude Code session decides on the fly what to parallelize and in what order. That works for short tasks. For a goal with many steps, it leaves three gaps:

- **Order is implicit.** Nothing guarantees that a task starts only after the work it depends on is verified.
- **Context grows.** Every step shares one conversation, so long runs get expensive and noisy.
- **"Finished" is a claim.** A subagent reporting success is not the same as the work being correct.

`claude-waypoints` moves those three concerns out of the model and into plain code.

## Concepts

| Term | Meaning |
| --- | --- |
| **Waypoint** | One task: a title, acceptance criteria, a scope, and how to verify it. |
| **Route** | The whole graph of waypoints for a goal. |
| **Leg** | One attempt to complete a waypoint, run by a subagent with a clean context. |
| **Gate** | A waypoint that waits for a human decision. |
| **Log** | An append-only event file that lets a route resume after the session ends. |

## How it is meant to work

```
 /waypoints plan | run | status          tools the model can call
            |                                      |
            v                                      v
   +--------------------------------------------------+
   |  route file  -->  validate graph                 |
   |  scheduler: ready? parallel limit? scope lock?   |
   +------------------------+-------------------------+
                            | one subagent per leg
                            v
              subagent with a bounded, clean context
                            |
                            v
   verify: scope respected -> files changed -> checks pass
        pass: accept, unlock dependents
        fail: retry with the findings, or raise a gate
                            |
   append-only log  +  live pane, status line and toasts
```

Planned behavior:

- **Deterministic scheduler.** A waypoint is ready when every dependency is accepted. Ready waypoints run in parallel up to a limit, and two waypoints with overlapping write scopes never run at the same time.
- **Bounded context.** Each leg receives only the goal, its own criteria, and the reports of the waypoints it depends on.
- **Scope enforced at the tool call.** Edits outside a waypoint's allowed paths are denied by a hook, rather than relying on the subagent to follow instructions.
- **Verification before acceptance.** A leg that exits cleanly is not accepted until its checks pass.
- **Resumable.** Progress lives in a log inside the repository. After a restart, legs that never finished are marked lost and retried.
- **Visible.** A live pane shows the graph, and anything that needs a person (approve, retry, reject) is a button.

## Roadmap

- [ ] Route file format and graph validation (ordering, cycles, scope overlap)
- [ ] Scheduler with parallel limit and write-scope locks
- [ ] Leg runner on Claude Code subagents
- [ ] Verification pipeline and retry with findings
- [ ] Event log and resume
- [ ] Live pane, status line and approval gates
- [ ] `/waypoints` commands and model-callable tools
- [ ] Reusable route templates

## Installation

Not available yet. Install instructions will be added with the first release.

## Contributing

The project is at the design stage, so the most useful contribution right now is feedback on the concepts above. Open an issue to discuss a change before sending a pull request.

## License

To be decided before the first release.
