# AI-Agents-Threat-Model
This Skills file is built based on OWASP Top 10 For Agentic Applications 2026 Report (https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/)

Use the following instructions on how to use it across different AI coding Tools

# Claude Code
Claude Code natively supports the Agent Skills standard.

**Setup:** Place the SKILL.md in .claude/skills/AI-Agent-Threat-Model/SKILL.md.

**Run:** Type /AI-Agents-Threat-Model in your session.

**Pro-Tip:** Use the ultrathink keyword in your prompt (e.g., /AI-Agent-Threat-Model ultrathink "audit our RAG connector") to trigger a deeper architectural analysis.

# Google Antigravity
Google Antigravity uses a similar "Skills" architecture to extend its reasoning capabilities.

**Setup:** Use the antigravity skills add command or point it to your local directory.

**Constraint:** Ensure your system_instruction explicitly mentions: "Strictly adhere to the 'First Principles' and 'Least Agency' guidelines defined in the Agentic Threat Modeler skill".

# GitHub Copilot
While Copilot doesn't support the full /command execution of Claude Code Skills, you can enforce these rules using project instructions.

**Setup:** Copy the content of the SKILL.md (excluding frontmatter) into a file named .github/copilot-instructions.md.

**Usage:** Copilot will now automatically use the OWASP ASI terminology when you ask it for "Security Review" or to "Rewrite this tool for better isolation."
