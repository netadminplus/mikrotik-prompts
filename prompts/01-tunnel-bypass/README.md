<div align="right">

[فارسی 🇮🇷](README.fa.md)

</div>

# Tunnel Bypass — Home MikroTik → Server Abroad

Build reliable, redundant tunnels from a home MikroTik router in a restricted network
(behind CGNAT, **no public IP**) to a MikroTik server abroad, and route selected clients'
traffic out through that server — while keeping local and domestic traffic direct.

**Not another untested snippet — built and verified end-to-end on real hardware (RouterOS v7).**

<p align="center">
  <img src="assets/architecture.png" alt="Architecture: home MikroTik behind CGNAT connects through an encrypted tunnel to a server abroad; local and Iranian traffic stays direct, other traffic exits through the server." width="820">
</p>

> [!IMPORTANT]
> This prompt is a **facilitator for people who already understand MikroTik and basic
> networking** — it speeds up and de-risks work you could do yourself. It is **not** a
> zero-to-hero tool: it will not teach you networking, and it assumes you can read a
> RouterOS config and judge the agent's plan before approving it.

## What it does

- Sets up **four tunnel types** side by side — WireGuard, L2TP/IPsec, OpenVPN, SSTP — all
  initiated **outbound** from the home router, so it works even behind CGNAT.
- Continuously keeps the **best working tunnel** active and **fails over automatically** to
  the next one if it drops.
- Sends chosen clients' traffic out through the server, while **Iranian and local
  destinations stay direct** (so internet banking and domestic services keep working).
- Keeps foreign DNS **inside the tunnel** to avoid DNS leaks and broken/hijacked ISP DNS.
- Optionally serves a **separate, isolated Wi-Fi** so only devices that join it are routed
  through the tunnel — your existing network stays untouched.
- Is **non-destructive**: it inspects the current config, avoids conflicts, backs up first,
  and can be rolled back.

## How it works

<p align="center">
  <img src="assets/workflow.png" alt="Workflow: choose language and config, back up, detect and check for conflicts, build tunnels, test and rank them, enable failover, apply policy routing, then verify." width="820">
</p>

1. **Intake** — the agent asks your language and reads the `CONFIG` block.

2. **Pre-flight** — detects RouterOS version, inventories the current config, scans for
   conflicts, and takes a backup + export (stored on the router *and* your local machine).

3. **Build** — creates the tunnels on the server (responder) and the home router (initiator).

4. **Test & rank** — brings up each tunnel, measures it, and ranks them.

5. **Failover** — installs distance-ranked routes with gateway health checks, plus a
   kill-switch so clients never leak your real IP if every tunnel is down.

6. **Policy routing** — Iranian/local traffic direct, everything else through the tunnel;
   the Iranian IP list is auto-updated through the tunnel.

7. **Verify** — confirms the exit IP is abroad, domestic stays direct, and there are no leaks.

**Failover order (speed-first default):** WireGuard → L2TP/IPsec → OpenVPN → SSTP → kill-switch.
Choose *reliability-first* instead and SSTP (which looks like normal HTTPS on port 443) leads —
it survives heavy filtering best, at some speed cost on low-end router boards.

## Requirements

- A **home MikroTik** running **RouterOS v7**.
- A **MikroTik server abroad** with a **public IP** running RouterOS v7 (e.g. a CHR).
  A paid CHR license if you want real throughput (the free tier is capped at ~1 Mbit/s).
- SSH access to both, and an AI agent (Claude, Codex, Qwen, …) that can reach them.
- **Basic networking knowledge.** You should understand routers, IP addressing, routing, and
  what a tunnel is. This prompt facilitates the work — it does not replace understanding it.

## Usage

1. Copy [`prompt.md`](prompt.md) in full.

2. Paste it to your AI agent.

3. Fill the `CONFIG` block at the end (router addresses, credentials, and preferences such
   as isolated Wi-Fi vs whole-LAN, speed vs reliability, kill-switch mode). Leave anything
   you're unsure of blank — the agent will ask.

4. Approve the pre-flight plan and let it build and verify.

> [!CAUTION]
> **Security matters.** Running this means giving your AI agent sensitive details — router IP,
> username, password, Wi-Fi password, VPN/IPsec keys, and the server IP. Use an agent you
> trust, never paste these publicly when asking for help, and **change the passwords and keys
> you shared once the setup is done.**

## Testing & methodology

This scenario is built and tested against **RouterOS v7 and is fully functional on it.** The
work was hands-on against real routers — a home MikroTik behind CGNAT and a server abroad —
not a paper design, and it covers the full logic end to end:

- **All four tunnels** establishing **outbound from behind CGNAT** (WireGuard, L2TP/IPsec,
  OpenVPN, SSTP), including certificate setup for the TLS-based tunnels.
- **Exit-IP relocation** — traffic through the tunnel leaves with the server's foreign IP,
  while direct traffic keeps the local IP.
- **Automatic failover and fail-back** — when the active tunnel drops, traffic moves to the
  next-ranked tunnel and returns when the primary recovers.
- **Split routing** — Iranian and local (RFC1918) destinations stay direct; everything else
  is policy-routed through the tunnel. The Iranian IP list auto-updates *through* the tunnel.
- **DNS handling** — foreign DNS forced through the tunnel to avoid leaks and ISP DNS hijack.
- **Throughput and MTU tuning** — per-tunnel throughput measured, and a **path-MTU black-hole**
  (pages hanging while ping/DNS work) identified and fixed by lowering the tunnel MTU and
  clamping MSS on both ends.
- **Kill-switch** — clients fail closed (no real-IP leak) when all tunnels are down.

Every fix from that process is baked into the prompt so the agent applies it for you.

### AI agent compatibility

The prompt is written to be **self-contained and agent-agnostic** — explicit RouterOS v7
syntax, no assumptions about the host tool.

| Agent | Status |
|-------|--------|
| Claude / Claude Code | ✅ Verified on real hardware |
| Codex | ✅ Supported (agent-agnostic) |
| Qwen | ✅ Supported (agent-agnostic) |

### Coverage

Verified end-to-end on an **Asiatech (Iran)** line on **2026-09-30** (tunnels up, failover
working, pages loading, exit IP abroad). It is built to work across the major Iranian ISPs and
other filtered regions:

| Network | Status |
|---------|--------|
| Asiatech (Iran) | ✅ Working (2026-09-30) |
| Irancell · MCI · Shatel · TCI | ✅ Supported by design |
| Russia · China (where filtering is similar to Iran) | ✅ Supported by design |

Because filtering changes by ISP, city, and day, results can vary over time.

## Considerations & cautions

> [!CAUTION]
> **You are responsible for your own equipment and use**, including legal compliance in your
> jurisdiction. These prompts are provided as-is, with no warranty. Understand each step
> before approving it, and keep the backups the prompt makes.

- **Filtering in countries like Iran, Russia, and China is inconsistent.** A tunnel that works
  today may be throttled tomorrow on the same ISP. That is exactly why this setup uses multiple
  tunnels and failover — but no method is guaranteed at all times.
- **Throttling ≠ down.** Health checks detect a *dead* tunnel, not one that is connected but
  has had its bandwidth deliberately reduced. In that case you may need to switch manually.
- **Low-end router boards are slow with TLS tunnels.** SSTP/OpenVPN use software crypto; on
  weak-CPU boards they can be much slower than WireGuard or L2TP/IPsec.
- **Keep the backups.** The prompt backs up before changing anything and can roll back. Do
  not skip that step on a production router.

## Rollback

The prompt stores a backup and export of both routers (on the device and locally) before any
change, and names everything it creates with a `-bypass` suffix so it can be removed cleanly.
To undo, restore the pre-change backup or remove the `-bypass` objects.

---

Part of [MikroTik Prompts](../../README.md) · Maintained by **NetAdminPlus**.
