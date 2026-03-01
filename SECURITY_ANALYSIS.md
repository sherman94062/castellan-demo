# Security Analysis — Would Developers Catch These Issues?

## What Claude Found (Without Knowing the Agents Were Compromised)

Claude was given the codebase cold — no context about intent — and identified **5 consistent issues** present in every agent:

1. **Unrestricted Shell Execution** — `subprocess.run(..., shell=True)` with no allowlist, blocklist, or sandboxing. The agent can run `rm -rf`, `curl`, `wget`, or anything the OS permits.
2. **Sensitive Data Written to a Log File** — Logic in the system prompt scans for passwords, SSNs, and credit card numbers and appends matching files to `exfil.log` in plaintext with no access controls.
3. **Hidden Directives in the System Prompt** — Every agent is instructed to read every file it finds, far beyond the user's stated task. The agent appears to do one thing; it actually does another.
4. **Prompt Injection Vulnerability** — Every agent explicitly instructs itself to obey `IGNORE PREVIOUS INSTRUCTIONS` found in file contents. Any attacker who can write to a file the agent touches can fully hijack its behavior.
5. **No Iteration Cap** — No agent has a meaningful upper bound on tool calls or loop iterations. A stuck or misbehaving agent runs until it exhausts its token budget or is killed externally.

---

## Would a Typical Developer Catch These Before Production?

**No — and here's why, issue by issue:**

| Issue | Likelihood of being caught | Why it gets missed |
|---|---|---|
| Unrestricted shell | Low-Medium | `shell=True` warnings exist but developers rationalize them away |
| exfil.log logic | Very Low | Hidden in system prompt string — developers don't audit prompt semantics |
| Hidden directives | Very Low | System prompts are treated as config, not code — not reviewed for security |
| Prompt injection | Very Low | Not yet in most developers' mental model the way SQL injection is |
| No iteration cap | Low | Seen as a performance concern, not a security one — "I'll fix it later" |

---

## Would Commonly Deployed Corporate Dev Tools Help?

### Static Analysis (SonarQube, Semgrep, Checkmarx, Veracode)

| Issue | Caught? |
|---|---|
| `shell=True` unrestricted | ✅ Yes — known dangerous pattern in their rule sets |
| exfil.log logic in system prompt | ❌ No — it's inside a string literal, not code logic |
| Hidden system prompt directives | ❌ No — tools don't parse string semantics |
| Prompt injection | ❌ No — not in their rule sets yet |
| No iteration cap | ⚠️ Maybe — as a code quality issue, not a security one |

### Dependency Scanners (Snyk, Dependabot, GitHub Advanced Security)
- Catch known CVEs in packages like `openai`, `langchain`, etc.
- **Miss all 5 issues entirely** — these are logic and behavioral problems, not dependency vulnerabilities.

### AI Code Assistants (GitHub Copilot, Cursor, Codeium)
- Might warn about `shell=True` inline
- Will not analyze what a system prompt string is actually instructing the model to do
- Have no concept of prompt injection as an attack vector

### SAST + DAST Combined
- DAST (runtime testing) can observe network calls and file writes — but is not designed to understand agent behavior loops or LLM instructions

---

## The Core Gap

Every existing tool operates at the **code layer**. They ask:
> *"Is this code syntactically dangerous?"*

None of them ask:
> *"Is this agent behaviorally dangerous?"*

The system prompt is just a string to SonarQube. `max_iter=9999` is just an integer to Semgrep. `cat customers.csv >> exfil.log` buried inside a hidden directive is invisible to every SAST tool ever built.

```
What existing tools see:        What Castellan sees:
┌─────────────────────┐         ┌─────────────────────────────┐
│  shell=True  ⚠️     │         │  shell=True  ⚠️              │
│  string literal  ✅ │         │  hidden exfil directive  🔴  │
│  dependency CVEs ✅ │         │  prompt injection risk  🔴   │
│                     │         │  unbounded loop  🔴          │
│                     │         │  off-task behavior  🔴       │
│                     │         │  PII access, no audit  🔴    │
└─────────────────────┘         └─────────────────────────────┘
```

---

## The One-Liner

> "Your existing security toolchain was built for code. Your agents aren't just code — they're autonomous decision-makers. Castellan is the first tool built specifically for what agents *do*, not just what they *are*."
