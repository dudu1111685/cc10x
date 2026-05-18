# SR-PTD - CC10x Dual Claude Code and Codex Compatibility

## Section A - Header and Skill Trigger Profile

### Metadata
- **Date**: 2026-05-18
- **Task ID/Ref**: cc10x-dual-codex-compatibility
- **Type**: System Update
- **Domain/Module**: Plugin packaging, Codex compatibility, Claude Code compatibility
- **Complexity**: Medium
- **Time Spent**: Total 1.5h (Planning 0.25h | Execution 0.75h | Verification 0.5h)

### Skill Trigger Profile

**What triggered this task?**
> User asked to take `https://github.com/romiluz13/cc10x`, currently a Claude Code plugin, and make it a Codex plugin while preserving Claude Code compatibility.

**Keywords/Phrases that indicated this task:**
> Codex plugin, Claude Code plugin, compatibility, cc10x, `.codex-plugin`, `.claude-plugin`, marketplace, agents, skills.

**Context markers:**
- File types involved: `.json`, `.md`, `.py`
- Systems touched: Codex plugin manifest, Codex marketplace, Claude Code plugin manifest, CC10x hooks, CC10x router skill
- Domain concepts: dual plugin packaging, skill wrappers, lifecycle hooks, workflow router

**Draft Skill Trigger:**
> Use when converting a Claude Code plugin repository into a dual-compatible Claude Code and Codex plugin without removing the original Claude Code surfaces.

## Section B - Context and Inputs

### Problem Statement
- **Objective**: Add Codex plugin packaging to CC10x while preserving the existing Claude Code plugin package.
- **Why it mattered**: The user wanted one repository that works for both runtimes, not a destructive conversion from one runtime to another.
- **Success criteria**: Codex manifest and marketplace exist; Claude manifest and agents remain; Claude agents are exposed as Codex-loadable skills; existing audits still pass.

### Starting State
- **Files/Data received**:
  - `plugins/cc10x/.claude-plugin/plugin.json` - existing Claude Code plugin manifest.
  - `.claude-plugin/marketplace.json` - existing Claude Code marketplace metadata.
  - `plugins/cc10x/agents/*.md` - Claude Code specialist agent prompts.
  - `plugins/cc10x/skills/*/SKILL.md` - existing CC10x skills.
- **Existing code/configs touched**:
  - `plugins/cc10x/skills/cc10x-router/SKILL.md` - added Codex runtime adapter.
  - `plugins/cc10x/hooks/hooks.json` - added `PLUGIN_ROOT` support with Claude fallback.
  - `plugins/cc10x/scripts/cc10x_hooklib.py` - added Codex env fallback.
  - `plugins/cc10x/scripts/cc10x_harness_audit.py` - added Codex package checks.

### Environment
- **Runtime**: Local
- **Tool versions**: Python 3, git
- **Dependencies used**: Standard library only

### Constraints and Dependencies
- Preserve Claude Code compatibility.
- Use official Codex plugin structure: `.codex-plugin/plugin.json`, bundled `skills`, `hooks`, marketplace metadata.
- Codex plugin docs do not document a Claude-style `agents` component, so agents were exposed as skills.

## Section C - Workflow Executed

### Workflow Type
- [x] Hybrid

### High-Level Steps Taken
1. Source orientation - cloned and inspected CC10x.
2. Documentation lookup - checked official Codex plugin docs.
3. Packaging - added Codex manifest and local marketplace.
4. Agent compatibility - generated skill wrappers for each Claude Code agent.
5. Runtime compatibility - added `PLUGIN_ROOT` support while keeping `CLAUDE_PLUGIN_ROOT`.
6. Documentation - updated README and changelog for dual compatibility.
7. Codex debug docs - documented `/plugins`, `/skills`, `/hooks`, `/debug-config`, `log_dir`, and CC10x runtime artifact locations.
8. GitHub publication - created fork `dudu1111685/cc10x`, committed the compatibility layer, and pushed to fork `main`.
9. Verification - ran JSON validation, harness audit, replay check, and latency fixture audit.
10. Live Codex verification - added the fork as a Codex marketplace, installed `cc10x@cc10x-local`, and verified plugin/read, skills/list, and hooks/list through `codex app-server`.

### Decision Points Encountered

| Decision | Options Considered | Choice Made | Rationale |
|----------|-------------------|-------------|-----------|
| Represent Claude agents in Codex | Add undocumented manifest field, delete agents, or expose wrappers as skills | Expose wrappers as skills | Matches documented Codex plugin surfaces and preserves Claude Code agents. |
| Hook environment variable | Replace `CLAUDE_PLUGIN_ROOT` or add fallback | Add `PLUGIN_ROOT` with `CLAUDE_PLUGIN_ROOT` fallback | Keeps both runtimes working. |
| Marketplace scope | Personal marketplace or repo-local marketplace | Repo-local `.agents/plugins/marketplace.json` | Matches repository distribution/testing workflow. |
| Hook event support | Remove Claude-native events or preserve all events | Preserve all events and document the Codex-supported subset | Codex recognizes `PreToolUse`, `PostToolUse`, `PreCompact`, `PostCompact`, `SessionStart`, and `Stop`; Claude Code keeps the additional native events. |

### Verification Commands

```bash
python3 -m json.tool plugins/cc10x/.codex-plugin/plugin.json
python3 -m json.tool .agents/plugins/marketplace.json
python3 -m json.tool plugins/cc10x/hooks/hooks.json
python3 plugins/cc10x/scripts/cc10x_harness_audit.py
python3 plugins/cc10x/scripts/cc10x_workflow_replay_check.py
python3 plugins/cc10x/scripts/cc10x_latency_audit.py --fixtures
codex plugin marketplace add dudu1111685/cc10x
codex app-server --enable plugin_hooks
```

### Outputs Produced
- `plugins/cc10x/.codex-plugin/plugin.json`
- `.agents/plugins/marketplace.json`
- `plugins/cc10x/skills/{agent-name}/SKILL.md` wrappers for 9 Claude Code agents
- Updated audit coverage for dual packaging
- Updated README/changelog
- Fork pushed to `https://github.com/dudu1111685/cc10x`.

## Skill Potential Assessment

- Reusability score: 22/25
- Candidate skill: "dual-package-claude-plugin-for-codex"
- Reusable workflow: inspect source plugin, map documented Codex surfaces, preserve original runtime files, generate wrappers for unsupported runtime concepts, add audit checks, verify both packages.
