# Veracode Compensating Controls — Agent Skill

An [Agent Skills](https://agentskills.io) skill that helps developers respond to Veracode SAST findings. Given a CWE ID, it walks through applicable compensating controls and generates a ready-to-paste **Mitigated by Design** statement for the Veracode portal.

These are designed to be expanded and changed on a per security team basis. For example, some teams dictate that data coming from a configuration file is acceptable risk. Others don't, depending on the risk profile. This is a way of helping guide the team to an appropriate outcome.

## What it does

1. You provide a CWE ID from a Veracode finding
2. The agent loads the relevant reference file (`CWE-<id>.md`) and presents the controls that apply
3. For each control it explains what it is and how to confirm it is in place
4. Once you confirm which controls exist in your application, the agent drafts the mitigation text

If you give it more context, it can provide an opinion on what you have, depending on the LLM.

### Key distinction

| Term | Meaning |
| --- | --- |
| **Fixed** | Root cause eliminated. SAST would no longer report it. Risk is zero. |
| **Mitigated** | Flaw remains in the code and SAST still reports it, but compensating controls reduce the risk to an acceptable level. **This skill is for mitigation.** |

## Supported CWEs

| CWE | Title |
| --- | --- |
| [CWE-80](./CWE-80.md) | Basic XSS — Improper Neutralization of Script-Related HTML Tags |
| [CWE-89](./CWE-89.md) | SQL Injection — Improper Neutralization of Special Elements in SQL |

More CWEs can be added by creating a `CWE-<id>.md` file following the existing format.

---

## Installation

This skill follows the [Agent Skills](https://agentskills.io) open standard and works with any compatible client.

### Option A — Cross-client (works everywhere)

Clone or copy this repository into your user-level skills directory:

```bash
# Standard cross-client path
git clone https://github.com/dipsylala/veracode-compensating-controls ~/.agents/skills/veracode-compensating-controls
```

Or for a project-specific install:

```bash
git clone https://github.com/dipsylala/veracode-compensating-controls .agents/skills/veracode-compensating-controls
```

### Option B — Client-specific paths

| Client | User-level path |
| --- | --- |
| Claude Code / Claude desktop | `~/.claude/skills/veracode-compensating-controls` |
| VS Code (GitHub Copilot) | `~/.vscode/skills/veracode-compensating-controls` |
| Cursor | `~/.cursor/skills/veracode-compensating-controls` |
| Gemini CLI | `~/.gemini/skills/veracode-compensating-controls` |
| Roo Code | `~/.roo-cline/skills/veracode-compensating-controls` |

Example for Claude Code:

```bash
git clone https://github.com/dipsylala/veracode-compensating-controls ~/.claude/skills/veracode-compensating-controls
```

### Keeping it up to date

```bash
cd ~/.agents/skills/veracode-compensating-controls
git pull
```

---

## Usage

Once installed, the skill activates automatically when your agent detects a Veracode-related task. You can also trigger it explicitly:

> "I have a Veracode finding for CWE-89 on line 42 of UserRepository.java. What compensating controls can I apply?"

The agent will load the skill, present the relevant controls, and draft a mitigation statement once you confirm which are in place.

---

## Adding more CWEs

Create a new file named `CWE-<id>.md` in this directory. Follow the structure of an existing file:

- **Overview** — what the CWE is
- **Note on scope** — clarify when to mitigate vs. when the fix would eliminate the finding entirely
- **Compensating Controls** — one section per control, each with a plain-English explanation and a "How to confirm" guide
- **Mitigation Text Templates** — ready-to-paste text for each control or combination

Focus controls on cases where a legitimate protection exists but the static analyser cannot trace it — not on controls that would fix the flaw outright.
