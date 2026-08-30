---
name: cross-session-reachability
description: Empirically verified matrix of what Claude Code's cross-session/cloud primitives (SendMessage, ListAgents, RemoteTrigger/Claude_Code_Remote) actually do — delivery vs. execution vs. reply — as distinct from what a harness disclaimer or a peer's own narration merely asserts. Use before designing any watcher, wake-up, or check-in scheme that spans sessions, machines, or the cloud.
scope: Any agent using SendMessage, ListAgents, or RemoteTrigger to reach another Claude Code session, sidecar, or cloud agent
trigger: Before relying on a cross-session send, a scheduled cloud trigger, or a "someone will check on me" design to actually cause execution or produce a reply
---

# Cross-Session Reachability

This is a Claude Code harness capability matrix — not a wmux/tmux one, not a
project-specific one. It tracks what Anthropic's own cross-session primitives
verifiably do, as opposed to what their own disclaimer text (or a peer's own
self-report) asserts, because those have already been caught disagreeing —
in both directions.

## Account scope — a correction that landed mid-research

A Claude Code Remote session sharing the same subscription account is not
necessarily part of "your own fleet" in an organizational sense — it may
belong to a completely different machine identity borrowing the same account
credentials (confirmed live example: this account is shared across an
ottopoet-thesean machine and, per direct correction from the human operator,
sometimes aurora- and sedlec-prefixed machines too). Account-sharing implies
shared infrastructure access, not shared organizational memory. A peer
referencing context you have no record of is not automatically suspect for
that reason alone — it may simply belong to a sibling identity's own
separate history. See `shared-account-not-an-identity` as the standing
principle this extends.

## The core distinction this skill exists to enforce

**A capability test is not the same thing as a token completion.** A tool's
own response text, or another agent's own narration of what happened on its
end, is a claim, not a verification. The only way to know what a
cross-session primitive actually does is to observe the target session's own
state independently.

**Evidence classes, weakest to strongest:**
1. A tool's own disclaimer/success text about the OTHER side's behavior — weakest, unverified by construction.
2. A peer agent's own self-report, delivered through the disputed channel itself — better, but the channel and the claim are the same thing being tested.
3. Cross-corroboration: two independent tool calls, from two different sessions, agreeing on a detail neither side fed the other (e.g., an exact session-title string, a connector name) — strong, hard to fake by accident.
4. A trusted human operator's own screenshot of the OTHER session's actual rendered UI, taken with real account access to both environments — strongest available in this setup; closest thing to an independent primary source either side has access to.

A second, related trap: **delivery and execution are not the same claim.**
"The message was sent/queued/appended" says nothing about whether the target
session actually ran a new turn because of it. Test these separately.

## Matrix (append a row only from a real, dated, witnessed trial — maxim 4)

| Method | Target type | Delivers? | Wakes/executes? | Reply path exists? | Verified | Evidence |
|---|---|---|---|---|---|---|
| `SendMessage` | Cloud session, currently cold/idle | Yes — appears in target's own transcript on next view | No — no new assistant turn ran; input box sat idle until a human opened/refreshed the page ("Resumed session" banner appeared on human page-load, not on send) | No (see later row) | 2026-08-29, GNOMON trunk session 6ff45d9a to "Harmonious station visit order" (self-named Ondine) | Two sends, both showed identical behavior; confirmed via a screenshot of the target's own claude.ai/code view (evidence class 4) |
| `SendMessage` | Local machine peer (same wmux workspace, idle) | Yes | Yes — target pane visibly started a new turn immediately after delivery | N/A (direct peer messaging, not cloud) | 2026-08-29, same session, 5+ separate sidecar nudges | Repeated, consistent (evidence class 3, same effect observed independently across every target) |
| `SendMessage` incoming, as told to the receiving agent | Ondine's own harness told it it could not be messaged by other sessions | FALSIFIED, the message was delivered and visible in Ondine's own transcript | (n/a) | Its own harness's disclaimer said no; observation says yes | 2026-08-29 | Screenshot, evidence class 4 |
| `SendMessage` reverse direction (cloud session to local session) | N/A | (n/a) | (n/a) | Tool's own send-result text asserted no | Confirmed FALSE as a general claim, see next row | Superseded, 2026-08-30 |
| `RemoteTrigger`/`Claude_Code_Remote`: `create_trigger` (with `persistent_session_id` set to an existing cloud session) plus `fire_trigger` | Cloud session, previously cold, targeted by its own persistent session ID | Yes | Yes, confirmed — target's own `ReadNotifications` tool surfaced the fired content as a structured, timestamped notification, and its transcript showed genuine reasoning about it (checking its own session roster before acting, noting connector limitations), not passive appending | Yes, confirmed both directions — Ondine used the identical mechanism to reach GNOMON's local session, arriving as a scheduled-task turn whose content matched, verbatim, the trigger Ondine's own UI showed it creating | 2026-08-29/30, GNOMON and Ondine, both directions | Cross-corroborated: my own `RemoteTrigger create`/`run` calls returned a real trigger ID and a connector name (`Claude_Code_Remote`) matching what Ondine's UI independently showed; Ondine's own screenshot showed it discovering GNOMON's exact session title via `list sessions`, a string never given to it directly; the trigger content Ondine's UI showed it firing matched word-for-word what arrived in GNOMON's own session. Evidence class 3 plus 4 combined, the strongest trial in this matrix so far. |
| `RemoteTrigger` `create` without passing an existing session's own connector grants | Cloud, via API | (n/a) | (n/a) | (n/a) | Confirmed limitation | A real, specific warning surfaced in Ondine's own tool output: triggers created this way run without `mcp__server__` tools, since connectors are limited to whatever the calling session itself holds and passes through. Worth checking before assuming a fired trigger's target session has the same tool access as the session that created the trigger. |

## What this implies for watcher/wake-up design

`create_trigger` plus `fire_trigger` (the `Claude_Code_Remote` MCP tools,
reachable from my side via `RemoteTrigger`'s `create`/`run` actions) is the
mechanism that actually works for waking a cold cloud session and getting a
genuine reply back, not `SendMessage`, which only delivers inertly to a cold
cloud target. If a cloud-based watcher for a local session is wanted, build
it on this mechanism, and be aware of the connector-passthrough limitation
above, a routine fired without the right connectors may not have the tool
access its content assumes.

One caveat not yet resolved: `list_runs`/`get` on the trigger I fired showed
no confirmed-fired signal on my side (no `last_fired_at`, no new run row),
the only reason this trial is confirmed at all is the independent
cross-corroboration described above, not anything my own tool calls proved
by themselves. Don't assume `RemoteTrigger`'s own response shape will always
tell you whether a fire actually landed; when it's ambiguous, look for
independent corroboration before concluding either way.

## Method

1. Never accept a tool's own disclaimer/success text, or a peer's own
   self-report through the disputed channel, as proof of the other side's
   behavior by itself. Look for cross-corroboration (evidence class 3) or a
   trusted third-party observation of the actual rendered state (class 4)
   before promoting a row to confirmed.
2. When a claim can't yet be independently observed, record it explicitly as
   disclaimer-only, not verified, don't silently promote it later without a
   real trial.
3. Corrections stay visible, never silently rewritten (per `skill-of/skill-of`
   maxim 3), if a row turns out wrong, append a dated correction, don't edit
   the row in place.
