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

- When agent_speaking = True

  -  Soft acknowledgements → ignored

  -  Interrupt commands or mixed sentences → interrupt

- When agent_speaking = False
  -  All inputs are accepted as valid user input

This ensures identical words behave differently depending on the agent’s state.

## Semantic Interruption Detection

The handler performs semantic analysis on the STT transcript.

**Rules:**

- If the transcript contains any interrupt keyword, the agent is interrupted.

- If the transcript contains only soft acknowledgements, it is ignored.

- If the transcript contains a mix of soft acknowledgements and other words, it is treated as an interruption.

### Examples:

"yeah"                  → IGNORE (while speaking)

"yeah wait a second"    → INTERRUPT

"stop"                  → INTERRUPT


## No VAD Kernel Modification

The low-level VAD remains unchanged.

Instead:

The handler receives VAD events via on_vad(vad_id)

- A short delay is introduced to wait for STT

- The handler decides whether to interrupt before stopping playback

- This keeps the solution modular and compliant with the assignment.

## Decision Flow

## Decision Flow

- **VAD Triggered**
- **Wait briefly for STT (~100 ms)**
- **Check agent speaking state**
- **Analyze STT semantics**
- **Final Decision**
  - IGNORE
  - ACCEPT
  - INTERRUPT


If STT does not arrive within the timeout:

- While speaking → IGNORE (safe default)

- While silent → no action


## Code Structure

**Core Class: InterruptHandler**

### Responsibilities:

- Track agent speaking state

- Handle VAD events

- Handle STT results

- Apply interruption decision rules

## Tokenization Strategy

- All text is converted to lowercase

- Split on non-alphanumeric characters

- Tokens are matched against:

  - Soft acknowledgement set

  - Interrupt keyword set

- This avoids errors due to punctuation or casing.

## Conclusion

This implementation correctly solves the problem of intelligent interruption handling by combining:

- Agent state awareness

- Semantic understanding of user speech

- Minimal and safe integration with existing VAD systems

The solution is modular, configurable, and fully compliant with the assignment requirements.
