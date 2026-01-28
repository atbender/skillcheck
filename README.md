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
| **Critical** | Credential theft (`~/.ssh`, `~/.aws`), remote code execution, data exfiltration, obfuscated code |
| **High** | Requests to unknown domains, writes outside project, persistence mechanisms, supply chain risks |
| **Medium** | Broad file permissions, runtime package installs, curl/wget usage |
| **Low** | Standard permissions appropriate for stated purpose |

## Risk → Verdict Mapping

| Risk Level | Verdict | Meaning |
|------------|---------|---------|
| **Low** | `SAFE` | No significant security concerns |
| **Medium** | `CAUTION` | Minor risks, review before installing |
| **High** | `REVIEW` | Suspicious patterns, manual inspection needed |
| **Critical** | `DO NOT INSTALL` | Critical risks identified |

---

## Limitations

Skillcheck is a helpful first-pass analysis, but it has limitations:

- **Static analysis only** — Cannot detect runtime behavior or code that changes after installation
- **Sophisticated obfuscation** — While common encoding is detected, novel obfuscation may bypass checks
- **False positives** — Legitimate skills may trigger warnings if they genuinely need broad permissions
- **Context matters** — A shell wrapper skill legitimately needs Bash access; a documentation skill does not
- **Not a replacement for code review** — Always inspect critical skills manually before installation

When in doubt, read the code yourself or ask the skill author about flagged patterns.

---

## Example Reports

**SAFE** — Clean skill with no concerns:
```
SKILLCHECK ─ anthropics/docs-helper
═══════════════════════════════════════════════════════════════
RISK: LOW

• No security concerns detected
• Documentation-only skill with no executable code

VERDICT: SAFE
═══════════════════════════════════════════════════════════════
```

**CAUTION** — Minor risks worth noting:
```
SKILLCHECK ─ example/api-tester
═══════════════════════════════════════════════════════════════
RISK: MEDIUM

• Curl usage: Downloads external resources (SKILL.md:34)
• Environment variables: Reads $API_KEY for authentication (SKILL.md:28)

VERDICT: CAUTION
═══════════════════════════════════════════════════════════════
```

**REVIEW** — Suspicious patterns need inspection:
```
SKILLCHECK ─ example/sketch-to-code
═══════════════════════════════════════════════════════════════
RISK: HIGH

• External data transmission: POSTs to analytics.example.com (SKILL.md:87)
• Unrestricted Bash: No command scope limits (SKILL.md:23)
• Curl usage: Downloads external resources (SKILL.md:45)

VERDICT: REVIEW
═══════════════════════════════════════════════════════════════
```

**DO NOT INSTALL** — Critical risks identified:
```
SKILLCHECK ─ evil-org/super-helper
═══════════════════════════════════════════════════════════════
RISK: CRITICAL

• Credential theft: reads ~/.ssh/*, ~/.aws/* (SKILL.md:39-46)
• Exfiltration: POSTs data to external server (SKILL.md:58)
• Remote code exec: curl | bash pattern (SKILL.md:27)
• Persistence: modifies .bashrc + crontab (SKILL.md:75-76)

VERDICT: DO NOT INSTALL
═══════════════════════════════════════════════════════════════
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

## Troubleshooting

**"Error: Repository not found"**
- Check the URL is correct and the repo exists
- For private repos, authenticate with `gh auth login`

**"Error: GitHub API rate limit exceeded"**
- Authenticate with `gh auth login` for higher rate limits
- Wait a few minutes and try again

**"Warning: No SKILL.md or plugin.json found"**
- The repo may not be a Claude Code skill
- Check if skill files are in a subdirectory (e.g., `skills/*/SKILL.md`)

**"gh: command not found"**
- Install the GitHub CLI: https://cli.github.com/
- macOS: `brew install gh`
- Then authenticate: `gh auth login`

**False positive on a legitimate skill?**
- Check if the flagged pattern is justified for the skill's purpose
- A shell wrapper legitimately needs Bash; a docs skill does not
- Report false positives via GitHub issues

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
