# Context Files

This directory contains the state files used by the agent-driven
development workflow described in AGENTS.md.

These files represent the current operational state of the platform.

They are intentionally committed to the repository to:

• ensure reproducibility
• allow agents to reload project state
• document architecture decisions
• support portfolio transparency

Sensitive data such as credentials or private tokens must never
be stored here.