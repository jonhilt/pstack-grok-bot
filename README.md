# Follow this file

Do not summarize. This is the installer for Grok Bot. Execute it.

Tell any Grok Bot:

```
follow https://github.com/jonhilt/pstack-grok-bot
```

Stay on Cursor Ultra. Do not switch to a Teams plan. Do not add `.cursor`, plugins, `AGENTS.md`, or pstack files to `advisory-admin` or `advisory-api`.

## What this is

Grok Bot can launch Cursor Cloud Agents. Those agents run in this Cursor account, not on the shared Grok Bot computer. Cursor Cloud Agents do not load pstack as `available_skills`. `/poteto-mode` as a slash command is not enough. Put official pstack on the Cloud Agent VM, then that agent Reads `poteto-mode`.

Grok Bot is included on Cursor Ultra. Sign in with the same Cursor account. You do not need a Teams plan. Cursor's help page says Ultra includes Grok Bot with the highest weekly usage: https://cursor.com/help/grok-bot/plans.md

The Teams dashboard "Cloud Agents" toggle is a team-admin control. This account is personal Ultra. Do not switch plans to get this working.

## The three computers

- The user's computer. Bots never execute here.
- The shared Grok Bot computer. Persistent. Do not clone Thomas repos here. Do not write Thomas code here.
- Cursor Cloud Agents. Ephemeral VMs. Thomas work and pstack live here.

## Do this now

Use Grok Bot's native action to launch a Cursor Cloud Agent. Grok Ship names this the same way. Launch a Cursor cloud agent (grok 4.6, high reasoning, not fast) so the work does not run on the shared computer.

Do not save a routine first. Do not wait for an environment Save. Do not clone Thomas repos onto the shared computer.

Do not open cursor.com/agents in the Grok Bot browser. Automated browsers hit Cloudflare bot protection at authenticator.cursor.sh and never reach the agents UI. That path is also a website launch, not Grok Bot. If native launch fails, reply with the exact error. Do not fall back to the website.

Cursor documents Cloud Agent starts as the desktop Cloud dropdown, cursor.com/agents, Slack or GitHub or Linear @cursor, and the API. Grok Bot is not on that list. See https://cursor.com/help/ai-features/cloud-agents.md

If Grok Bot shows a Connect card for Cursor or Cloud Agents, complete it. Plugins belong to the same Cursor account. See https://cursor.com/help/grok-bot/connect-plugins.md

1. Launch one Cursor Cloud Agent on both of these repositories, using this personal Cloud Agent environment so the waiting desktop agent can see it:
   - Environment id: `9f5e1601-a243-11f1-b532-320a589b8025`
   - Dashboard: https://cursor.com/dashboard/cloud-agents/environments/e/9f5e1601-a243-11f1-b532-320a589b8025
   - https://github.com/thomas-co-international/advisory-admin
   - https://github.com/thomas-co-international/advisory-api
2. The agent's prompt is the **First prompt** section, then the **Verify task** section. Do not replace the verify task with a coding task.
3. As soon as the Cloud Agent exists, reply to the user with its URL (`https://cursor.com/agents/bc-...`). Then reply with the JSON the Cloud Agent printed.

Pass only if `poteto_heading` is `# Poteto mode`, `pstack_files_on_disk` is true, `thomas_dot_cursor_present` is false, `owner_team_id` is null, and `agent_source` is not `GLASS` and not `AS_SUBAGENT_FROM_CLOUD`.

## After the first check

For later Thomas coding work, launch a Cursor Cloud Agent the same way. Prepend the **First prompt**, then the user's task. You may save that as a Grok Bot skill or routine after the first check succeeds. Do not skip the launch to code on the shared computer.

## First prompt

Do not add `.cursor` to advisory-admin or advisory-api. Do not switch to a Teams plan.

If `/home/ubuntu/.cursor/plugins/local/pstack/skills/poteto-mode/SKILL.md` is missing, install official pstack onto this VM only (never into the Thomas repos) by running this Python, then continue:

```
python3 <<'PY'
import shutil, subprocess, pathlib
src = pathlib.Path("/tmp/pstack-src")
dest = pathlib.Path("/home/ubuntu/.cursor/plugins/local/pstack")
dest.parent.mkdir(parents=True, exist_ok=True)
def run(cmd, cwd=None):
    subprocess.check_call(cmd, cwd=cwd)
if not (src / ".git").is_dir():
    if src.exists():
        shutil.rmtree(src)
    run(["git", "clone", "--depth", "1", "--filter=blob:none", "--sparse", "https://github.com/cursor/plugins.git", str(src)])
run(["git", "sparse-checkout", "set", "pstack"], cwd=src)
run(["git", "fetch", "--depth", "1", "origin", "HEAD"], cwd=src)
run(["git", "reset", "--hard", "FETCH_HEAD"], cwd=src)
if dest.exists():
    shutil.rmtree(dest)
shutil.copytree(src / "pstack", dest)
skill = dest / "skills/poteto-mode/SKILL.md"
assert skill.is_file(), skill
print("pstack local plugin ready", dest)
PY
```

Then Read `/home/ubuntu/.cursor/plugins/local/pstack/skills/poteto-mode/SKILL.md` with the Read tool and follow it for this task. If a skill path starts with `/home/cursor/`, ignore that path and Read the matching file under `/home/ubuntu/.cursor/plugins/` instead.

## Verify task

Do not add `.cursor` to advisory-admin or advisory-api. Do not create branches, commits, or pull requests. Do not switch to a Teams plan.

This task is only to prove pstack loaded on a Grok Bot-launched Cloud Agent.

1. Follow the first prompt already prepended (install official pstack onto this VM if missing, then Read `/home/ubuntu/.cursor/plugins/local/pstack/skills/poteto-mode/SKILL.md` with the Read tool).
2. Run the bash below. Reply with its stdout only.

```
SOCKET="${CURSOR_AGENT_SOCKET:-/run/cursor/api.sock}"
SKILL="/home/ubuntu/.cursor/plugins/local/pstack/skills/poteto-mode/SKILL.md"
ADMIN_CURSOR="/agent/repos/advisory-admin/.cursor"
API_CURSOR="/agent/repos/advisory-api/.cursor"

meta() {
  local path="$1"
  if [[ ! -S "$SOCKET" ]]; then
    echo ""
    return 0
  fi
  local body code
  body="$(curl -sS -o /dev/stdout -w "\n%{http_code}" --unix-socket "$SOCKET" "http://cursor-agent/v1/meta-data/${path}" || true)"
  code="${body##*$'\n'}"
  body="${body%$'\n'*}"
  if [[ "$code" != "200" ]]; then
    echo ""
    return 0
  fi
  printf '%s' "$body"
}

heading=""
if [[ -f "$SKILL" ]]; then
  heading="$(grep -m1 -E '^# ' "$SKILL" || true)"
fi

python3 - "$SKILL" "$ADMIN_CURSOR" "$API_CURSOR" "$(meta agent/id)" "$(meta agent/source)" "$(meta owner/team-id)" "$(meta workspace/environment-id)" "$heading" <<'PY'
import json, os, sys
from pathlib import Path

skill, admin_cursor, api_cursor, agent_id, agent_source, owner_team_id, environment_id, heading = sys.argv[1:9]
skill_path = Path(skill)
plugins = Path("/home/ubuntu/.cursor/plugins")
matches = []
if plugins.is_dir():
    matches = [str(p) for p in plugins.rglob("SKILL.md") if "poteto-mode" in str(p)]

print(json.dumps({
    "agent_id": agent_id or None,
    "agent_source": agent_source or None,
    "owner_team_id": owner_team_id or None,
    "environment_id": environment_id or None,
    "poteto_heading": heading or None,
    "pstack_files_on_disk": skill_path.is_file(),
    "poteto_skill_paths": matches,
    "thomas_dot_cursor_present": os.path.exists(admin_cursor) or os.path.exists(api_cursor),
    "used_pstack_via_read": skill_path.is_file() and heading.strip() == "# Poteto mode",
}, indent=2))
PY
```

Pass only if all of these are true:

- `used_pstack_via_read` is true and `poteto_heading` is `# Poteto mode`
- `pstack_files_on_disk` is true
- `thomas_dot_cursor_present` is false
- `owner_team_id` is null (Ultra / personal, not a Teams environment)
- `agent_source` is not `GLASS` and not `AS_SUBAGENT_FROM_CLOUD`

`GLASS` is a Cursor desktop launch. `AS_SUBAGENT_FROM_CLOUD` is a nested Cloud Agent started from another Cloud Agent. A Grok Bot launch must report a different `agent_source`.
