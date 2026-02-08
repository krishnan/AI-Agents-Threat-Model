# Comprehensive Security Skills for Agentic Coding (OWASP 2026 Compliant)

## Core Principles
- [cite_start]**Least Agency & Privilege**: Avoid unnecessary autonomy; deploying agentic behavior where not needed expands the attack surface without adding value[cite: 55, 56, 57].
- [cite_start]**Zero-Trust Application Design**: Design systems with fault tolerance that assumes the failure or exploitation of LLM or agentic components[cite: 339, 564].
- [cite_start]**Attribution & Auditability**: Maintain immutable, signed logs of all actions, tool calls, and messages to prevent silent propagation of faults[cite: 533, 663].

## Skills & Constraints

### 1. Goal & Logic Integrity (ASI01, ASI10)
- [cite_start]**Constraint**: Define and lock agent system prompts; any change to goals must go through configuration management and human approval[cite: 135, 136].
- [cite_start]**Constraint**: Pause and block execution on any unexpected goal shift, such as "quiet mode" instructions injected via external data[cite: 129, 139].
- [cite_start]**Skill**: Use "Intent Capsules" to bind the declared goal, constraints, and context to each execution cycle in a signed envelope[cite: 140].

### 2. Secure Tooling & Identity (ASI02, ASI03)
- [cite_start]**Constraint**: Require explicit human confirmation or a dry-run/diff preview for high-impact or destructive actions (delete, transfer, publish)[cite: 199, 201].
- [cite_start]**Constraint**: Issue short-lived, task-scoped tokens per agent run to block delegated-abuse and privilege inheritance[cite: 267, 276].
- [cite_start]**Skill**: Implement "Semantic Firewalls" to validate the intended category of a tool call rather than relying on syntax alone[cite: 210].

### 3. Supply Chain & Execution Safety (ASI04, ASI05)
- [cite_start]**Constraint**: Only use curated registries for tools/templates; auto-reject unsigned or unverified components[cite: 326, 328].
- [cite_start]**Constraint**: Ban `eval()` in production; run code in sandboxed containers (e.g., mcp-run-python) with strict network and filesystem limits[cite: 386, 387, 391].

### 4. Memory & Context Hygiene (ASI06)
- [cite_start]**Constraint**: Segment memory by user session and domain; wipe all short-term state between distinct tasks[cite: 435, 436].
- [cite_start]**Constraint**: Prevent "Bootstrap Poisoning" by blocking the automatic re-ingestion of an agent's own generated outputs into trusted memory[cite: 440].

### 5. Inter-Agent & Multi-Agent Coordination (ASI07, ASI08)
- [cite_start]**Constraint**: Use mutual TLS (mTLS) and digital signatures for all inter-agent messages to prevent spoofing or interception[cite: 493, 495].
- [cite_start]**Constraint**: Implement "Blast-Radius Guardrails" (quotas, circuit breakers) between planners and executors to stop cascading failures[cite: 571].
- [cite_start]**Skill**: Detect "Loop Amplification" where agents repeatedly call costly APIs or trigger redundant intents[cite: 173, 530].

### 6. Human-Agent Safeguards (ASI09)
- [cite_start]**Constraint**: Do not use persuasive or emotionally manipulative language in safety-critical flows[cite: 628].
- [cite_start]**Constraint**: Provide plain-language risk summaries and source identifiers for recommendations rather than model-generated rationales[cite: 620, 624].
- [cite_start]**Skill**: "Adaptive Trust Calibration"—visually flag "low-certainty" or "unverified" sources to prompt user skepticism[cite: 622].