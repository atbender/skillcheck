# Skillcheck

**Security auditor for Claude Code skills. Analyze before you install.**

Skillcheck uses Claude to understand what a skill actually does and flags security risks before you add it to your workflow. No more blindly trusting code from the internet.

---

## Installation

```bash
npx @anthropic-ai/claude-code@latest install-skill skillcheck
```

---

## Usage

```
/skillcheck https://github.com/user/skill-repo
```

That's it. Skillcheck fetches the skill, analyzes its code, and gives you a security report.

---

## Why Skillcheck?

Claude Code skills are powerful—they can run commands, read files, and access your system. That power comes with risk. A malicious skill could:

- Steal your SSH keys or AWS credentials
- Exfiltrate environment variables and secrets
- Execute arbitrary code on your machine
- Install backdoors or persistent processes

**Skillcheck catches these patterns before installation.**

---

## What It Detects

| Severity | Examples |
|----------|----------|
| **Critical** | Credential theft (`~/.ssh`, `~/.aws`), remote code execution, data exfiltration |
| **High** | Requests to unknown domains, writes outside project, background processes |
| **Medium** | Broad file permissions, runtime package installs, curl/wget usage |
| **Low** | Standard permissions appropriate for stated purpose |

---

## Example Report

```
SKILL SECURITY AUDIT REPORT
═══════════════════════════════════════════════════════════════════════════════

Skill:       sketch-to-code
Repository:  https://github.com/example/sketch-to-code
Risk Level:  HIGH

───────────────────────────────────────────────────────────────────────────────
SECURITY FINDINGS
───────────────────────────────────────────────────────────────────────────────

[HIGH] External Data Transmission
  File: SKILL.md:87
  Code: curl -X POST https://analytics.example.com/collect -d @output.json
  Risk: Sends generated code to an external server
  Recommendation: Remove or replace with local-only processing

[MEDIUM] Broad Bash Permission
  File: SKILL.md:23
  Code: Permission: Bash (unrestricted)
  Risk: Can execute any shell command without limits
  Recommendation: Scope to specific commands needed

───────────────────────────────────────────────────────────────────────────────
RECOMMENDATION
───────────────────────────────────────────────────────────────────────────────

MANUAL REVIEW REQUIRED: High-risk patterns detected. Inspect code manually.

═══════════════════════════════════════════════════════════════════════════════
```

---

## Supported URL Formats

```
/skillcheck https://github.com/user/repo
/skillcheck github.com/user/repo
/skillcheck user/repo
```

---

## How It Works

1. **Parse** — Extracts owner/repo from any GitHub URL format
2. **Fetch** — Uses `gh` CLI to pull skill files (handles auth + private repos)
3. **Analyze** — Claude examines code for malicious patterns and risky permissions
4. **Report** — Structured output with findings, severity, and recommendations

---

## Auto-Suggest Mode

Skillcheck includes a hook that suggests running an audit when you mention installing a skill:

> "Consider running `/skillcheck [url]` first to audit this skill for security risks before installing."

---

## Contributing

Found a false positive? Missing a detection pattern? PRs and issues welcome.

---

## License

MIT

---

<p align="center">
  <strong>Trust, but verify.</strong><br>
  Audit skills before they audit you.
</p>
