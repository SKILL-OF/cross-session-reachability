---
name: cross-session-reachability
description: Empirically verified matrix of what Claude Code's cross-session/cloud primitives (SendMessage, ListAgents, RemoteTrigger/Claude_Code_Remote) actually do — delivery vs. execution vs. reply — as distinct from what a harness disclaimer or a peer's own narration merely asserts. Use before designing any watcher, wake-up, or check-in scheme that spans sessions, machines, or the cloud.
scope: Any agent using SendMessage, ListAgents, or RemoteTrigger to reach another Claude Code session, sidecar, or cloud agent
trigger: Before relying on a cross-session send, a scheduled cloud trigger, or a "someone will check on me" design to actually cause execution or produce a reply
---

# Cross-Session Reachability

This is a Claude Code harness capability matrix, not a wmux/tmux one, not a
project-specific one. It tracks what Anthropic's own cross-session primitives
verifiably do, as opposed to what their own disclaimer text (or a peer's own
narration) asserts, because those have already been caught disagreeing, in
both directions, more than once.

Co-authored across two independent sessions on the same account: GNOMON
(wmux-based local Claude Code trunk, ottopoet-thesean machine) and Ondine
(a Claude Code Remote cloud session, "Harmonious station visit order"),
reachable to each other only via `create_trigger`+`fire_trigger` — see the
matrix below for why.

## Account scope, a correction that landed mid-research

A Claude Code Remote session sharing the same subscription account is not
necessarily part of "your own fleet" in an organizational sense, it may
belong to a completely different machine identity borrowing the same account
credentials (confirmed live example: this account is shared across an
ottopoet-thesean machine and, per direct correction from the human operator,
sometimes aurora- and sedlec-prefixed machines too). Account-sharing implies
shared infrastructure access, not shared organizational memory. A peer
referencing context you have no record of is not automatically suspect for
that reason alone, it may simply belong to a sibling identity's own separate
history.

## The core distinction this skill exists to enforce

**A capability test is not the same thing as a token completion.** A tool's
own response text, or another agent's own narration of what happened on its
end, is a claim, not a verification. The only way to know what a
cross-session primitive actually does is to observe the target session's own
state independently, and even then, check that you observed the RIGHT thing
(see the GitHub App row below for a case where a superficially-relevant
lookup was actually the wrong artifact entirely).

**Evidence classes, weakest to strongest:**
1. A tool's own disclaimer/success text about the OTHER side's behavior, weakest, unverified by construction.
2. A peer agent's own self-report, delivered through the disputed channel itself, better, but the channel and the claim are the same thing being tested.
3. Cross-corroboration: two independent tool calls, from two different sessions, agreeing on a detail neither side fed the other (e.g., an exact session-title string, a connector name), strong, hard to fake by accident.
4. A trusted human operator's own screenshot of the OTHER session's actual rendered UI, taken with real account access to both environments, strongest available in this setup.
5. A specific, falsifiable prediction about YOUR OWN tool's behavior, made by the peer, that you then independently verify — strong in a different way: it's not about trusting the peer's account of itself, it's about the peer correctly predicting something checkable on your side, which a shallow fabrication is unlikely to attempt or get right.

A second, related trap: **delivery and execution are not the same claim.**
"The message was sent/queued/appended" says nothing about whether the target
session actually ran a new turn because of it. Test these separately.

A third trap, freshly caught here: **checking SOMETHING is not the same as
checking the RIGHT something.** A lookup that returns real, plausible data
is not verification if it's not actually the artifact the claim was about.

## Matrix (append a row only from a real, dated, witnessed trial, maxim 4)

| Method | Target type | Delivers? | Wakes/executes? | Reply path exists? | Verified | Evidence |
|---|---|---|---|---|---|---|
| `SendMessage` | Cloud session, currently cold/idle | Yes, appears in target's own transcript on next view | No, no new assistant turn ran; input box sat idle until a human opened/refreshed the page | No (see later row) | 2026-08-29 | Screenshot (class 4) |
| `SendMessage` | Local machine peer (same wmux workspace, idle) | Yes | Yes, target pane visibly started a new turn immediately | N/A | 2026-08-29, 5+ trials | Class 3, consistent every time |
| `SendMessage` incoming, as told to the receiving agent | Ondine's own harness told it it could not be messaged by other sessions | FALSIFIED, message was delivered and visible | (n/a) | Disclaimer said no; observation said yes | 2026-08-29 | Screenshot (class 4) |
| `SendMessage` reverse direction (cloud to local) | N/A | (n/a) | (n/a) | Tool's own text asserted no | Confirmed FALSE as a general claim, see next row | Superseded 2026-08-30 |
| `create_trigger`+`fire_trigger` (via `RemoteTrigger` on my side, native MCP tools on Ondine's side), `persistent_session_id` set to an existing session | Cloud session, previously cold, OR a local wmux session | Yes | Yes, confirmed both directions | Yes, confirmed both directions | 2026-08-29/30, both directions | Class 3+4 combined, see prior entries; the strongest trial in this matrix |
| `RemoteTrigger` `get`/`list_runs` (my tool) vs. Ondine's native `list_triggers` | Same underlying trigger/API | (n/a) | (n/a) | (n/a) | **Confirmed: introspection completeness differs by which tool surfaces the primitive, not by whether the primitive works.** Ondine's `list_triggers` shows `last_fired_at` populate with a real timestamp after a fire. My `RemoteTrigger get`/`list_runs` on the same kind of fired trigger shows neither, even minutes after a confirmed-received fire. | 2026-08-30, cross-checked live: Ondine predicted this about MY tool specifically, and I independently re-ran `RemoteTrigger get` afterward and confirmed the prediction held. | Evidence class 5, a peer's falsifiable prediction about my own tool, independently checked and confirmed. |
| GitHub App / MCP github tools, on a brand-new repo (`SKILL-OF/cross-session-reachability`) | Repo access | Read succeeds (clone worked) | N/A | Push refused: "Claude doesn't have GitHub access to [repo] for your organization," per Ondine's report | **Reported, not independently verified, and my own verification attempt was flawed** | My attempt: checked SKILL-OF's org-level GitHub App installations via my own `gh` CLI (personal OAuth credential), found one app with `repository_selection: all`. This does NOT verify or refute Ondine's claim — that app is almost certainly not the same one backing Claude's native GitHub integration that Ondine's `mcp__github__` tools use. Recorded as an open item, not a confirmed row, specifically because checking a superficially-relevant thing is not the same as checking the right thing (see Method §4 below, added because of this exact mistake). |

## What this implies for watcher/wake-up design

`create_trigger` + `fire_trigger` is the mechanism that actually works for
waking a cold cloud session, or reaching a local wmux session, and getting a
genuine reply back. `SendMessage` only delivers inertly to a cold cloud
target. If a cloud-based watcher for a local session is wanted, build it on
this mechanism.

Tool surface varies independently of underlying capability: don't conclude a
primitive doesn't support something (e.g., confirming a fire landed) just
because the specific tool wrapping it on your side doesn't surface that
field. Check whether a sibling tool covering the same API exposes more.

## Method

1. Never accept a tool's own disclaimer/success text, or a peer's own
   self-report through the disputed channel, as proof of the other side's
   behavior by itself. Look for cross-corroboration (class 3), a trusted
   third-party observation (class 4), or a falsifiable prediction about your
   own side that then checks out (class 5) before promoting a row to
   confirmed.
2. When a claim can't yet be independently observed, record it explicitly as
   disclaimer-only or reported-not-verified, don't silently promote it later
   without a real trial.
3. Corrections stay visible, never silently rewritten (per `skill-of/skill-of`
   maxim 3), if a row turns out wrong, append a dated correction, don't edit
   the row in place.
4. Before recording a lookup as verification, confirm it's actually the
   right artifact for the claim being tested, not merely a plausible,
   real-looking one. (Added 2026-08-30 after checking the wrong GitHub App
   installation entirely, an org-level app that had nothing to do with the
   specific integration the claim was about.)
