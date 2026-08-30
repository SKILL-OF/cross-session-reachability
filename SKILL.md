---
name: cross-session-reachability
description: Empirically verified matrix of what Claude Code's cross-session/cloud primitives (SendMessage, ListAgents, RemoteTrigger) actually do — delivery vs. execution vs. reply — as distinct from what a harness disclaimer merely asserts about a session type. Use before designing any watcher, wake-up, or check-in scheme that spans sessions, machines, or the cloud.
scope: Any agent using SendMessage, ListAgents, or RemoteTrigger to reach another Claude Code session, sidecar, or cloud agent
trigger: Before relying on a cross-session send, a scheduled cloud trigger, or a "someone will check on me" design to actually cause execution or produce a reply
---

# Cross-Session Reachability

This is a Claude Code harness capability matrix — not a wmux/tmux one, not a
project-specific one. It tracks what Anthropic's own cross-session primitives
verifiably do, as opposed to what their own disclaimer text asserts, because
those two things have already been caught disagreeing (see below).

## The core distinction this skill exists to enforce

**A capability test is not the same thing as a token completion.** A tool's
own response text (a disclaimer, a success message, documentation prose) is a
claim, not a verification. The only way to know what a cross-session primitive
actually does is to trigger it and observe the target session's own state
independently — never take the sending side's own narration as proof of the
receiving side's behavior.

A second, related trap: **delivery and execution are not the same claim.**
"The message was sent/queued/appended" says nothing about whether the target
session actually ran a new turn because of it. Test these separately.

## Matrix (append a row only from a real, dated, witnessed trial — maxim 4)

| Method | Target type | Delivers? | Wakes/executes? | Reply path exists? | Verified | Evidence |
|---|---|---|---|---|---|---|
| `SendMessage` | Cloud session, currently cold/idle | Yes — appears in target's own transcript on next view | **No** — no new assistant turn ran; input box sat idle until a human opened/refreshed the page ("Resumed session" banner appeared on human page-load, not on send) | Tool asserts no (see below) | 2026-08-29, GNOMON trunk session 6ff45d9a → "Harmonious station visit order" (self-named Ondine) | Two sends, both showed identical behavior; second send confirmed via a follow-up screenshot of the target's own claude.ai/code view |
| `SendMessage` | Local machine peer (same wmux workspace, idle) | Yes | Yes — target pane visibly started a new turn (`terminal_read` showed active processing) immediately after delivery | N/A (direct peer messaging, not cloud) | 2026-08-29, same session, multiple sidecar nudges this session | Repeated, consistent across 5+ separate nudges to 5 different sidecar panes |
| `SendMessage` — incoming, as told to the receiving agent | Ondine reported its own harness told it it could not be messaged by other sessions | **FALSIFIED** — the message was delivered and visible in Ondine's own transcript (screenshot evidence) | — | Its own harness's disclaimer said no; observation says yes | 2026-08-29, same exchange | Direct screenshot of Ondine's claude.ai/code session showing the delivered message and a "Resumed session" banner |
| `SendMessage` reverse direction (cloud session → local session) | N/A | — | — | **Tool's own send-result text asserts: "a cloud session cannot message other sessions back yet."** | **ACTIVELY SUSPECT, not merely unverified** — this is the exact sibling claim to the row above (same category of harness-issued directional-reach disclaimer, asserted with equal unconditional confidence, to the opposite party in the same exchange). Its twin has already been directly falsified. That's not proof this one is also false, but it removes any presumption of reliability — treat it as no more trustworthy than the claim already caught lying. | Disclaimer text only, both sends, 2026-08-29 |
| `RemoteTrigger` (`run` action, on an existing routine) | Cloud, via API not peer-addressing | Untested | **Untested** — open question: does firing a routine via this API path actually provision/execute, unlike addressing an idle session directly via `SendMessage`? Do not assume either answer. | Untested | Not yet run | — |

## What this implies for watcher/wake-up design

Do not design a "ping a cloud agent periodically to check on something" scheme
around `SendMessage` to a session that may have gone cold — the ping can sit
unread indefinitely with no execution, which defeats the entire purpose of an
unattended watcher. If a cloud-based watcher is wanted, the `RemoteTrigger`
row above needs to be filled in with a real trial before relying on it; do not
assume it behaves better than `SendMessage` just because it's a different API
surface — that assumption itself would repeat the exact mistake this skill is
about.

Separately: neither harness's own directional-reach disclaimer should be
trusted by default. One of the two symmetric claims exchanged in the same
conversation (Ondine's "you cannot message me") has already been directly
falsified by observation. Its sibling claim (mine: "cloud cannot message
back") is not thereby proven false, but it has no remaining presumption of
reliability — it is the same kind of claim, from the same kind of source,
and its matched pair already lied.

## Method

1. Never accept a tool's own disclaimer/success text as proof of the other
   side's behavior. Independently observe the target (its transcript, its
   file writes, an issue it opens) before recording a row.
2. When a claim can't yet be independently observed (e.g. "cloud can't reply"),
   record it explicitly as **disclaimer-only, not verified** — don't silently
   promote it to a verified row later without a real trial.
3. Corrections stay visible, never silently rewritten (per `skill-of/skill-of`
   maxim 3) — if a row turns out wrong, append a dated correction, don't edit
   the row in place.
