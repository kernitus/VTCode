# Full Automation

`--full-auto` lets VT Code run without pausing for human approval on explicitly allow-listed tools. Use it only when you fully trust the workspace configuration and have reviewed the safeguards below.

`--full-auto` is intentionally separate from normal session permissions:

- Normal sessions use primary agents plus granular permission rules.
- Agent specs use `permissions.default` plus `allow`, `ask`, `auto`, and `deny` rule buckets.
- The `permissions.auto` bucket sends matching calls to classifier-backed review instead of treating them as unrestricted.
- `--full-auto` uses the explicit `[automation.full_auto]` allow-list and skips normal approval prompts for allow-listed tools.

## Activation Checklist

1. **Update `vtcode.toml`**
    - Enable the feature: `automation.full_auto.enabled = true`.
    - Configure the tool allow-list to match your risk tolerance.
    - Keep `require_profile_ack = true` so a profile file is required.
2. **Create the acknowledgement profile**
    - Place the file referenced by `automation.full_auto.profile_path` in your workspace.
    - Document acceptable behaviour, escalation procedures, and any workspace-specific hazards.
3. **Review tool policies**
    - Full automation still honours existing tool policies; denied tools remain blocked.
    - Tools not included in the allow-list will be rejected automatically.
4. **Launch the agent**
    - Run `vtcode --full-auto` with any other CLI flags you need.
    - VT Code will also set `--skip-confirmations` internally.

## Runtime Behaviour

- VT Code displays the active allow-list at session start.
- Tool permission prompts are bypassed for allow-listed tools.
- Non allow-listed tools are rejected before execution, and their attempts are logged.
- Git diff confirmations and other safety prompts are skipped automatically.
- If the acknowledgement profile is missing while required, the CLI aborts before launching.

## Customising The Allow-List

```toml
[automation.full_auto]
enabled = true
require_profile_ack = true
profile_path = "automation/full_auto_profile.toml"
allowed_tools = [
    "read_file",
    "list_files",
    "grep_file",
    "run_pty_cmd",
]
```

Tips:

- Use the constants listed in `vtcode_core::config::constants::tools` to avoid typos.
- Include `"*"` only when the workspace is fully isolated.
- Combine with granular agent permissions if you need per-tool constraints in normal interactive sessions.

## Orchestrated Harness

For longer unattended builds, prefer enabling the planner/evaluator harness instead of relying on a single uninterrupted build loop:

```toml
[agent.harness]
orchestration_mode = "plan_build_evaluate"
max_revision_rounds = 2
```

When enabled, `vtcode exec --full-auto` writes a small set of working artefacts under `.vtcode/tasks/`:

- `current_spec.md`: high-level execution spec
- `current_contract.md`: observable done criteria and verification contract
- `current_task.md`: tracker state
- `current_evaluation.md`: evaluator output after a completion attempt

This keeps long-running work resumable and makes evaluator-driven revision rounds explicit instead of relying on the generator to judge itself.

## Profile File Recommendations

The profile file is a simple acknowledgement document. Suggested content:

- Operator name and timestamp approving unattended execution.
- Workspace-specific limitations, such as directories that must not be modified.
- Contact or escalation details if automation encounters unexpected failures.
- Rollback procedures or monitoring steps to follow afterwards.

Keeping this file under version control provides a clear audit trail for when full automation was used and under which guardrails.
