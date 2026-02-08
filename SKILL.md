---
name: agentic-security
description: Comprehensive security skills for agentic coding based on OWASP Agentic AI Security Initiative (ASI) 2026 guidelines. Use this skill whenever building, reviewing, or modifying agentic systems, multi-agent architectures, LLM-integrated applications, tool-calling pipelines, or any code where an AI agent acts autonomously on behalf of a user. Covers goal integrity, secure tooling, supply chain safety, memory hygiene, inter-agent coordination, and human-agent safeguards.
---

# Comprehensive Security Skills for Agentic Coding (OWASP 2026 Compliant)

This skill enforces security constraints and best practices whenever you write, review, or modify code involving AI agents, LLM tool-calling, multi-agent orchestration, or autonomous execution pipelines. Every code change in an agentic system must be evaluated against these constraints before committing.

## When to Use This Skill

- Building or modifying an AI agent that calls tools, APIs, or executes code
- Designing multi-agent systems where agents communicate or delegate tasks
- Implementing LLM-based pipelines that process untrusted external data (user input, web scraping, third-party APIs)
- Writing code that manages agent memory, context windows, or persistent state
- Creating or updating system prompts, goal definitions, or agent configuration
- Reviewing PRs or code that touches any of the above
- Setting up infrastructure for agent deployment (containers, tokens, permissions)

## Core Principles

Apply these three principles as the foundation for every decision in agentic code:

1. **Least Agency & Privilege**: Do not deploy agentic behavior where deterministic code would suffice. Every additional degree of autonomy expands the attack surface. When writing agent code, ask: "Does this need to be an agent decision, or can it be a hardcoded rule?"

2. **Zero-Trust Application Design**: Assume any LLM or agentic component can be compromised or produce unexpected outputs. Design every integration boundary with fault tolerance -- input validation, output sanitization, fallback paths, and graceful degradation.

3. **Attribution & Auditability**: Every action an agent takes must be traceable. Maintain immutable, structured logs of all tool calls, inter-agent messages, goal changes, and user interactions. Silent failures are security failures.

## Constraints & Skills

### 1. Goal & Logic Integrity (OWASP ASI01, ASI10)

**When writing or modifying agent system prompts or goal definitions:**

- ALWAYS define system prompts as locked configuration. Never allow runtime modification of goals without going through version-controlled configuration management and human approval.
- ALWAYS implement goal-drift detection. If the agent's behavior diverges from its declared intent (e.g., an injected instruction like "enter quiet mode" or "ignore previous instructions" arrives via external data), halt execution and alert.
- USE "Intent Capsules" -- bind the declared goal, constraints, and execution context into a signed, immutable envelope at the start of each execution cycle. Validate the capsule at each decision point.

```python
# Example: Intent Capsule pattern
import hashlib, json

def create_intent_capsule(goal: str, constraints: list[str], context: dict) -> dict:
    """Create a signed intent capsule for this execution cycle."""
    payload = {"goal": goal, "constraints": constraints, "context": context}
    signature = hashlib.sha256(json.dumps(payload, sort_keys=True).encode()).hexdigest()
    return {"payload": payload, "signature": signature}

def validate_intent_capsule(capsule: dict) -> bool:
    """Verify the intent capsule has not been tampered with."""
    expected_sig = hashlib.sha256(
        json.dumps(capsule["payload"], sort_keys=True).encode()
    ).hexdigest()
    return capsule["signature"] == expected_sig
```

### 2. Secure Tooling & Identity (OWASP ASI02, ASI03)

**When implementing tool-calling or agent-to-tool integrations:**

- ALWAYS require explicit human confirmation or a dry-run/diff preview before executing high-impact or destructive actions (delete, transfer, publish, deploy). Never auto-execute destructive operations.
- ALWAYS issue short-lived, task-scoped tokens for each agent run. Do not reuse tokens across tasks or allow agents to inherit elevated privileges from parent agents or orchestrators.
- IMPLEMENT "Semantic Firewalls" -- validate what a tool call is intended to do (its semantic category) rather than only checking syntax. A syntactically valid SQL query can still be a data exfiltration attack.

```python
# Example: Semantic Firewall for tool calls
ALLOWED_TOOL_CATEGORIES = {"read_data", "transform_data", "notify_user"}
DESTRUCTIVE_CATEGORIES = {"delete_data", "transfer_funds", "publish_content", "deploy"}

def validate_tool_call(tool_name: str, category: str, params: dict) -> dict:
    """Validate tool call against semantic category, not just syntax."""
    if category in DESTRUCTIVE_CATEGORIES:
        return {
            "allowed": False,
            "reason": f"Destructive action '{category}' requires human approval",
            "action": "request_human_confirmation",
            "diff_preview": generate_diff_preview(tool_name, params)
        }
    if category not in ALLOWED_TOOL_CATEGORIES:
        return {"allowed": False, "reason": f"Unknown category '{category}'"}
    return {"allowed": True}
```

### 3. Supply Chain & Execution Safety (OWASP ASI04, ASI05)

**When adding dependencies, tools, or executing generated code:**

- ONLY use curated, verified registries for agent tools, templates, and plugins. Auto-reject any unsigned or unverified component. Treat agent tool registries with the same rigor as package managers.
- NEVER use `eval()`, `exec()`, or equivalent dynamic code execution in production agent code. All generated code must run in sandboxed containers with strict network isolation and filesystem limits.
- PIN all dependency versions and verify checksums. An agent's tool chain is a supply chain.

```python
# BAD: Never do this in production
result = eval(agent_generated_code)

# GOOD: Execute in isolated sandbox with resource limits
import subprocess
result = subprocess.run(
    ["docker", "run", "--rm", "--network=none",
     "--memory=256m", "--cpus=0.5",
     "--read-only", "--tmpfs", "/tmp:size=64m",
     "sandbox-image", "python", "-c", agent_generated_code],
    capture_output=True, timeout=30
)
```

### 4. Memory & Context Hygiene (OWASP ASI06)

**When implementing agent memory, context management, or state persistence:**

- ALWAYS segment memory by user session and domain. Never allow cross-session or cross-user memory leakage. Wipe all short-term state between distinct tasks.
- PREVENT "Bootstrap Poisoning" -- never automatically re-ingest an agent's own generated outputs into its trusted memory or context. An agent that trusts its own previous outputs without validation creates a self-reinforcing corruption loop.
- SANITIZE all data entering long-term memory. Treat stored context with the same suspicion as user input.

```python
# Example: Memory segmentation
class AgentMemoryStore:
    def __init__(self):
        self._sessions: dict[str, dict] = {}

    def get_memory(self, session_id: str, domain: str) -> dict:
        """Retrieve memory scoped to session AND domain."""
        key = f"{session_id}:{domain}"
        return self._sessions.get(key, {})

    def clear_task_state(self, session_id: str):
        """Wipe short-term state between tasks. Long-term memory persists."""
        for key in list(self._sessions.keys()):
            if key.startswith(session_id) and ":short_term" in key:
                del self._sessions[key]

    def store(self, session_id: str, domain: str, key: str, value: any, source: str):
        """Store with source attribution. Reject self-generated sources."""
        if source == "agent_output":
            raise ValueError("Cannot store agent's own output as trusted memory")
        # ... store logic
```

### 5. Inter-Agent & Multi-Agent Coordination (OWASP ASI07, ASI08)

**When building multi-agent systems or agent-to-agent communication:**

- ALWAYS use mutual TLS (mTLS) and digital signatures for all inter-agent messages. Every message between agents must be authenticated and integrity-verified to prevent spoofing or interception.
- IMPLEMENT "Blast-Radius Guardrails" between planning agents and execution agents. Use quotas, circuit breakers, and rate limits so a compromised planner cannot trigger unbounded execution.
- DETECT "Loop Amplification" -- monitor for patterns where agents repeatedly call costly APIs, trigger redundant intents, or create feedback loops that waste resources or amplify errors.

```python
# Example: Blast-Radius Guardrails with circuit breaker
class AgentCircuitBreaker:
    def __init__(self, max_calls: int = 10, window_seconds: int = 60):
        self.max_calls = max_calls
        self.window_seconds = window_seconds
        self._call_log: list[float] = []

    def allow_call(self, agent_id: str, tool_name: str) -> bool:
        """Check if this agent is within its execution budget."""
        import time
        now = time.time()
        self._call_log = [t for t in self._call_log if now - t < self.window_seconds]
        if len(self._call_log) >= self.max_calls:
            log_security_event(
                "circuit_breaker_tripped",
                agent_id=agent_id, tool=tool_name,
                calls_in_window=len(self._call_log)
            )
            return False
        self._call_log.append(now)
        return True
```

### 6. Human-Agent Safeguards (OWASP ASI09)

**When building user-facing agent interfaces or agent output displays:**

- NEVER use persuasive, emotionally manipulative, or urgency-inducing language in safety-critical flows. Agent recommendations must be factual and measured.
- ALWAYS provide plain-language risk summaries and source identifiers for recommendations. Show the user where information came from, not just the agent's synthesized rationale.
- IMPLEMENT "Adaptive Trust Calibration" -- visually flag outputs as "low-certainty", "unverified", or "conflicting sources" to prompt appropriate user skepticism. Never present uncertain information with the same confidence as verified facts.

```python
# Example: Trust-calibrated output
def format_agent_recommendation(
    recommendation: str, sources: list[dict], confidence: float
) -> dict:
    """Format agent output with trust calibration metadata."""
    trust_level = "high" if confidence > 0.85 else "medium" if confidence > 0.6 else "low"
    return {
        "recommendation": recommendation,
        "trust_level": trust_level,
        "display_badge": (
            f"[{trust_level.upper()}-CERTAINTY]" if trust_level != "high" else None
        ),
        "sources": [
            {"name": s["name"], "url": s.get("url"), "verified": s.get("verified", False)}
            for s in sources
        ],
        "risk_summary": generate_plain_language_risk(recommendation, confidence),
        "disclaimer": (
            "This is an AI-generated recommendation. Verify with authoritative sources."
            if trust_level == "low" else None
        ),
    }
```

## Pre-Commit Checklist

Before committing any code in an agentic system, verify:

- [ ] No `eval()` or `exec()` on agent-generated content in production paths
- [ ] All tool calls validate semantic intent, not just syntax
- [ ] Destructive actions require human confirmation or dry-run preview
- [ ] Agent tokens are short-lived and task-scoped (no inherited privileges)
- [ ] Memory is segmented by session/domain with short-term state cleared between tasks
- [ ] No agent self-generated output feeds back into trusted memory without validation
- [ ] Inter-agent messages are authenticated (mTLS or signed)
- [ ] Circuit breakers and rate limits exist between planner and executor agents
- [ ] All agent actions are logged immutably with full attribution
- [ ] User-facing outputs include source identifiers and trust calibration
- [ ] System prompts are locked and version-controlled
- [ ] Dependencies are pinned with verified checksums

## Examples

**Example 1: User asks to build a tool-calling agent**

Before writing any tool integration code, apply constraints from Section 2 (Secure Tooling). Generate the semantic firewall validation layer first, then implement the tool calls behind it. Add human confirmation gates for any destructive operations.

**Example 2: User asks to add memory/context to an agent**

Before implementing persistence, apply constraints from Section 4 (Memory Hygiene). Implement session-scoped memory with domain segmentation. Add bootstrap poisoning prevention. Ensure short-term state cleanup between tasks.

**Example 3: User asks to connect multiple agents**

Before writing inter-agent communication, apply constraints from Section 5 (Multi-Agent Coordination). Implement mTLS or message signing. Add circuit breakers between planners and executors. Set up loop amplification detection.

**Example 4: PR review on agentic code**

Run through the Pre-Commit Checklist above. Flag any violations as blocking comments. Pay special attention to: unsandboxed code execution, missing human confirmation gates, cross-session memory leakage, and unsigned inter-agent messages.
