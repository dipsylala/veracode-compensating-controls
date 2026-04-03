---
name: veracode-compensating-controls
description: Look up compensating controls and draft Veracode "Mitigated by Design" mitigation text for a given CWE ID. Use when a developer has a Veracode scan flaw, wants to document that risk has been mitigated, mentions compensating controls, or provides a CWE ID and asks about mitigations or Veracode approval.
---

# Veracode Compensating Controls

## Key concepts

| Term | Meaning |
|---|---|
| **Fixed / Remediated** | The root cause has been eliminated. A correctly fixed flaw would no longer be reported by SAST. Risk is reduced to zero. |
| **Mitigated** | The flaw still exists in the code and SAST still reports it, but one or more compensating controls reduce the risk to an acceptable level. This skill is for mitigation only. |
| **False Positive** | The flagged value is not user-controlled so the flaw is not exploitable. Still submitted as a **Mitigated by Design** mitigation — the text explains the data-flow proving attacker influence is not possible. |

> If a developer says they have "fixed" the issue but Veracode still flags it, they may have applied a control the scanner cannot trace. Walk them through the controls below — the fix may actually be a mitigation.

## Quick start

Given a CWE ID from a Veracode scan:

1. Resolve `<skill-dir>/CWE-<id>.md` and read it
2. Present the available compensating controls with brief explanations
3. Ask which controls are already in place in their application
4. Generate a ready-to-paste **Mitigated by Design** statement

---

## Workflow

### Step 1 — Identify the CWE

Extract the CWE ID from the Veracode finding. If the developer provides a description instead of an ID, infer the most likely CWE and confirm before proceeding.

### Step 2 — Load the reference file

Load `<skill-dir>/CWE-<id>.md`. If no file exists for that CWE, tell the developer and offer to draft one using general secure-coding guidance.

### Step 3 — Present controls

Show all compensating controls from the reference file. For each, include:
- What it is (one sentence)
- How the developer can confirm it is in place

Then ask: **"Which of these controls are already in place in your application?"**

### Step 4 — Generate mitigation text

Use the confirmed controls to produce a **Mitigated by Design** statement ready to paste into the Veracode portal.

Follow these rules:
- Only include controls the developer has confirmed
- Be specific — include method names, framework names, or other details the developer supplies
- Do not claim risk is zero — mitigation lowers risk to an acceptable level, it does not eliminate it
- If no controls can be confirmed, advise the developer to remediate the root cause; a mitigation cannot be justified

**Mitigation text structure:**
```
The risk from this [CWE-ID] finding is mitigated by design.

[For each confirmed control: 2-3 sentences describing what is implemented and why it reduces exploitability.]

The flaw remains present in the code but the controls described above reduce the residual risk of successful exploitation to an acceptable level.
```

---

## Notes

- Mitigation type target: **Mitigated by Design**
- A mitigation does not fix the flaw — it demonstrates that risk has been reduced to an acceptable level despite the flaw remaining
- If the flagged value is not user-controlled, this is still submitted as a **Mitigated by Design** mitigation; the mitigation text should explain the full data-flow proving the value cannot be influenced by an attacker
- Multiple layered controls produce a stronger case — include all that the developer can confirm
- Avoid vague statements; Veracode reviewers will reject unsupported claims
