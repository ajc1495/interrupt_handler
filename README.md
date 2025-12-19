# Interrupt Handler – Agent Speech Control Logic

## Overview

This project implements a **logic-level interruption handling system** for a voice-based agent.  
The goal is to correctly differentiate between:

- **Soft acknowledgements** (e.g., “yeah”, “ok”) that should *not* interrupt the agent while it is speaking.
- **Explicit interruption commands** (e.g., “stop”, “wait”) that must immediately interrupt the agent.
- The **same user input behaving differently** depending on whether the agent is speaking or silent.

This implementation strictly follows the assignment constraints:
- No modification to the low-level VAD kernel.
- All logic implemented as a decision layer inside the agent’s event loop.
- Decisions made using agent state + semantic content of STT.

---

## Assignment Requirements Covered

The implementation satisfies all required features:

1. **Configurable Ignore List**
2. **State-Based Filtering**
3. **Semantic Interruption Detection**
4. **No VAD Kernel Modification**

Each requirement is explained in detail below.

---

## Configurable Ignore List (Soft Acknowledgements)

A configurable list of words is defined to represent *soft acknowledgements* that should be ignored while the agent is speaking.

Default ignore list:
```text
["yeah", "ok", "okay", "hmm", "uh-huh", "right", "yep", "mm", "mhm"]
```
These words are:

Ignored only when the agent is speaking

Treated as valid user input when the agent is silent

The list can be modified easily when initializing the handler.

## State-Based Filtering
The handler maintains a boolean state:

agent_speaking = True / False

### Behavior:

* When agent_speaking = True

Soft acknowledgements → ignored

Interrupt commands or mixed sentences → interrupt

* When agent_speaking = False

All inputs are accepted as valid user input

This ensures identical words behave differently depending on the agent’s state.
