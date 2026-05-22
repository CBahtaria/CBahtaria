# Charles Bahtaria

Solo engineer building autonomous aerial systems with DAL-A safety discipline.
Based in Mbabane, Eswatini. Audience: defence forces, royals, government.

> The interface IS the model. Operator interfaces compress 6 data signals into the
> space marketing sites use for one headline.

---

## What I'm building

### [agentic-uav-stack](https://github.com/CBahtaria/agentic-uav-stack) — private
Multi-scale autonomous drone platform. Three-tier architecture:

| Layer | Component | What it owns |
|---|---|---|
| **Reactive** (1 kHz, on Pixhawk) | PX4 + NuttX RTOS | Attitude control, motor mix, safety pilot override |
| **Executive** (10–50 Hz, on Jetson) | Safety governor + ROS 2 Jazzy + DDS | Final flight authority. No `eval`/`exec`/`getattr` in hot path. Returns `NO_GO` on any exception. |
| **Advisory** (cloud or local) | AI router → Claude / Ollama | Suggestions only. Must include `HUMAN_AUTHORIZATION_REQUIRED: true` on engagement recs. |

Currently SRL-2, HIL ramping for SRL-3. 768 tests passing. Kardashev 0.68.

**Subsystems shipped:** Brain daemon with key isolation (`brain/daemon.py` owns every secret;
subprocesses get `env={}`); Simplex formal shield (5 geometric invariants); deadman's switch
with NTP-attack detection; Octogent multi-agent vuln scanner (9 persona children + 1 synthesis);
N-version dispatch (`shared/security/divtab.py` — silent divergence is the tamper signal);
LZ77+Markov audit codec (UAVZMA, ~46% smaller than UAVZ on governor batches); EAV status
registry (`shared/status_registry/` — NATS KV + SQLite mirror, no schema migrations).

### [sentinel](https://github.com/CBahtaria/sentinel) — public
UEDF SENTINEL v5.0 — military command & control system for the Umbutfo Eswatini Defence
Force. PHP 8.x + MySQL. Real-time drone fleet management, threat detection, RBAC with 2FA,
audit logging, WebSocket telemetry. Sister project to agentic-uav-stack; subscribes to
the same NATS namespace (`uav.v1.*`).

### [second-brain](https://github.com/CBahtaria/second-brain) — private
Obsidian-style knowledge vault. 29 cross-linked wiki articles + 28 studio agents that
compound knowledge via per-session `## Evolution Log` entries. Karpathy-style synthesis
pipeline (`<scratchpad>` before article emission). Driven by the agentic-uav-stack brain
daemon — `AGENT_EVOLUTION` coworker auto-appends Evolution Log entries from claude-mem
observations daily at 23:30.

---

## Operating principles

1. **No AI in the flight command path.** AI is advisory; the formal shield + governor
   have final say. Engagement recommendations require human authorization.
2. **DAL-A invariants are CI-gated.** `scripts/check_dal_a_rules.py` rejects `eval`,
   `exec`, `getattr`, `__import__` in safety-critical files.
3. **Fail-safe means `NO_GO`.** Governor returns NO_GO on any exception; decision
   variables are pre-bound before the try block to prevent stale-setpoint propagation.
4. **API keys never leave the daemon process.** `brain/daemon.py` loads `.env`, pops
   secrets from `os.environ`, then `subprocess.Popen(..., env={})`. Audit via
   `make audit-keys` (greps `/proc/*/environ`).
5. **Defense in depth at every layer.** SROS2 enclaves enforce topic-level pub/sub
   authorization at the DDS layer — even if AI code bypasses app-level auth, only the
   `ai_agent` enclave may publish `/ai/suggested_action`.
6. **Multi-persona security review.** 9 concurrent personas (red-team, blue-team,
   supply-chain, compliance, insider-threat, architecture-integrity, trojan-horse,
   goldilocks, cat-burglar) run weekly with explicit failure containment per child.

---

## Tech

**Languages.** Python 3.12 (primary, CI matrix 3.11–3.13), TypeScript, PHP 8, C++, Bash.

**Runtime.** ROS 2 Jazzy, NuttX RTOS, Linux (Fedora dev, RHEL9 ground station, Jetson Orin
edge). Docker Compose dev, Kubernetes + MicroShift edge, Helm prod, Ansible fleet, systemd.

**Data plane.** NATS JetStream (14 streams under `uav.v1.*`), Qdrant, SQLite (WAL + FTS5),
DuckDB (telemetry buffer), PostgreSQL, MySQL.

**AI.** Claude Sonnet/Opus (cloud, $5/day cap), Ollama mistral:7b-q4 (local fallback),
hybrid router with circuit breaker (3 failures → force local 15 min). Custom hooks: SFT,
RLVR (verifiable rewards), distillation, GGUF quantization, off-policy SAC.

**Security.** SROS2 keystore + DDS Security, TPM 2.0 attestation, cosign image signing
(WIP), NTS-NTP, MAVLink 2 message signing (WIP), supply-chain guardian (Jia Tan / XZ
post-mortem applied).

**UI.** Flask + Socket.IO + React/JSX (no build step — Babel on CDN). BARTARIA TAC-OS v5
design tokens — Carbon/Armor/Phosphor/Sodium/Tracer/Target/Iris palette, Doto + Iceland +
DM Sans + JetBrains Mono. r3f + shadergradient + liquid-glass-js + MetalFlow for the
knowledge graph webapp.

---

## Audience

I write so the next operator can pick this up cold. National Libraries, government
officials, royals are the design audience for the compliance posture. The repos and
docs reflect that — read `agentic-uav-stack/WORKING-CONTEXT.md` for the current state
snapshot, `agentic-uav-stack/SECURITY.md` for the threat model, `agentic-uav-stack/RULES.md`
for the hard rules.

---

## What I am not

Not a content marketer. Not a vibe-coder. Not a wrapper around someone else's API. The
work targets compliance gates before features, not after; tamper-evident audit before
"observability"; deterministic safety before clever AI.

Contact through the issue tracker on any repo above.
