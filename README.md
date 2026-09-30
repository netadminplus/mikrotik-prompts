<div align="right">

[فارسی 🇮🇷](README.fa.md)

</div>

# MikroTik Prompts

![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)
![RouterOS](https://img.shields.io/badge/RouterOS-v7-red.svg)
![Tested](https://img.shields.io/badge/tested-real%20hardware-success)

Tested AI prompts for configuring MikroTik routers across common, real-world scenarios.

Each prompt is written to be **agent-agnostic** — you can hand it to Claude, Codex, Qwen, or
another capable AI agent that has SSH access to your router, and it will guide the setup,
ask what it needs, and apply the configuration carefully.

> [!IMPORTANT]
> These prompts are a **facilitator for people who already know MikroTik and basic
> networking** — they speed up and de-risk work you could do yourself. They are **not** a
> zero-to-hero tool and will not teach you networking. Always review the agent's plan before
> approving it, and read the per-prompt README and cautions first.

## Prompts

| # | Prompt | What it does | Docs |
|---|--------|--------------|------|
| 01 | [Tunnel Bypass](prompts/01-tunnel-bypass/) | Builds redundant tunnels from a home MikroTik (behind CGNAT, no public IP) to a MikroTik server abroad, with automatic failover and policy routing, so selected clients reach the internet through the server. | [EN](prompts/01-tunnel-bypass/README.md) · [FA](prompts/01-tunnel-bypass/README.fa.md) |

## How to use

1. Open the prompt's folder and read its README.

2. Copy the whole `prompt.md` and paste it to your AI agent.

3. Fill the `CONFIG` block at the bottom of the prompt with what you know (router
   addresses, credentials, preferences). Anything you leave blank, the agent will ask.

4. Let the agent run its pre-flight checks (it backs up first) and confirm its plan.

## Responsibility & disclaimer

> [!CAUTION]
> These prompts are tested, but **you are responsible for what you run on your own
> equipment**, including legal compliance in your jurisdiction. Networks differ, and —
> especially in Iran — connectivity and filtering change constantly. Nothing here is
> guaranteed to work or to be safe in any particular situation. Always keep the backups the
> prompt makes, and be ready to roll back.

Please read the full **[Disclaimer](DISCLAIMER.md)** before use.

> [!CAUTION]
> **Security matters.** Running a prompt means giving an AI agent sensitive details — router
> IP, username, password, Wi-Fi password, keys, and your server IP. Use an agent you trust,
> never paste these publicly, and **change the passwords and keys you shared after setup.**

## License

Released under the [MIT License](LICENSE). No warranty of any kind.

---

Maintained by **NetAdminPlus** (Ramtin Rahmani Nejad). Contributions and field reports are welcome.
