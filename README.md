# Platanus Build Night ft. Anthropic

## 2026 - Buenos Aires - Tiendanube Office

---

This is the code repository for salmundani at Platanus Build Night 26, in Buenos Aires.

* Full name: Daniel Salmun
* Github username: salmundani

Remember you should push the code before the deadline and make sure its deployed.

Good luck 🍌🚀

---

## Running Locally

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed
- [Codex CLI](https://github.com/openai/codex) installed and configured
- A git repository with a remote

### Usage

```bash
claude --plugin-dir /path/to/ralphex
```

Then invoke the workflow with:

```
/ralphex <task description>
```

The plugin will plan, implement, commit, and request a Codex review in a loop until the review passes.
