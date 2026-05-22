# Persistent AI Companion/Assistant Desktop System
A modular AI companion system built in Python, focused on memory persistence, contextual interaction, and local execution. 

## Overview
An experimental, Python-based AI companion designed for deep desktop integration and long-term conversational persistence. The project explores how to build a context-aware agent that interacts directly with the local operating system (macOS) while maintaining a consistent behavioral profile across sessions.

## Current Focus
Development is currently centered on refining the agent's cognitive architecture to operate efficiently under hardware constraints. Key areas include:
- **Hierarchical Memory Management:** Moving beyond simple vector RAG to a layered system (short-term buffer, rolling session summaries, and episodic storage).
- **Tool Orchestration:** Decoupling the LLM from system execution. The LLM acts as the narrator/synthesizer, while Python handles deterministic task routing.
- **Local OS Integration:** Bypassing heavy automation frameworks in favor of lightweight, native OS interactions (AppleScript, subprocess manipulation).
- **Cross-Process Visual Sync:** Managing a lightweight, stateless UDP connection between the Python backend and a Unity-besed 2D/3D frontend for low-latency avatar lip-sync and state changes. (firs attempts for 3D where made on Godot, than switched on Unity engine)

## Technical Challenges & Solutions

Building a persistent agent reveals severe limitations in standard LLM implementation patterns. Here are some core issues currently being addressed:

**1. The "Role Bleeding" Problem**
In long-running sessions, models often lose track of ownership (confusing user statements with AI statements).
*Approach:* Instead of relying on post-processing or database tags, the system uses strict prompt hierarchy and absolute ownership directives in the system prompt, enforcing identity boundaries before the context window is evaluated.

**2. Memory Fragmentation vs. Narrative Continuity**
Standard semantic search (RAG) often retrieves isolated log lines based on keyword similarity, stripping away conversational context and causing hallucinations.
*Approach:* Implementing a "Session Summary" (a background worker that periodically asks a fast model to synthesize the current mood, topics, and goals) and an "Episodic Memory" table. The retrieval system searches completed narrative episodes rather than raw chat logs.

**3. Tool Orchestration and "Instruction Fatigue"**
Relying on a large model to correctly format JSON tags for system commands (e.g., `[ACTION: OPEN_APP]`) leads to high latency and frequent parsing errors due to conflicting prompt instructions (tone vs. formatting).
*Approach:* Implementing "Python-first Intent Detection". Input is parsed locally via regex/logic gates. If a system command is detected (e.g., opening an app or typing text), Python executes the action deterministically in milliseconds, then passes an execution report to the LLM to generate a natural, conversational acknowledgment.

*Generic architecture example of the routing logic:*
```python
# The system intercepts operational intent before LLM evaluation
if action_detected(user_input):
    success, result_msg = os_bridge.dispatch_intent(user_input)
    if success:
        # LLM is only used to narrate the successful deterministic execution
        system_report = f"[SYS_EXECUTION_REPORT: {result_msg}]"
        return generate_llm_response(context, system_report)
```

**4. Hardware and Rendering Constraints**
Running complex UI frameworks alongside local NLP routing on older hardware (e.g., 2017 dual-core architectures) causes severe CPU bottlenecking.
*Approach:* Stripped down heavy Python GUI libraries in favor of native standard Tkinter for the control panel. The visual representation (avatar) is offloaded to a compiled Godot executable, communicating with the Python brain via a lightweight UDP socket at 30fps. 

## Tech Stack
- **Backend/Logic:** Python 3
- **Persistence:** SQLite (multi-layered schema for raw logs, session contexts, and episodic memory)
- **Inference:** Groq API (Llama-3/OSS models)
- **Audio/TTS:** Edge-TTS, Whisper, and native macOS audio drivers (`afplay`)
- **Frontend/Avatar:** Godot Engine 3.5 (UDP socket receiver)
- **System Control:** Native macOS AppleScript & Bash

<img width="1394" height="671" alt="tkinter ui with unity runtime" src="https://github.com/user-attachments/assets/9954fb2b-b3fb-4230-a0db-df074d1b2d3b" />


## Repository Scope
This repository is intended as architectural documentation. Because the system is deeply coupled to my specific local environment, custom hardware constraints, and ongoing private experiments, the complete source code is *at the moment* not public. 

I use this space to progressively document system logic, architecture notes, specific problem-solving strategies, and structural snippets as the project evolves.

## Development Philosophy
The approach here is strictly pragmatic: build to solve actual bottlenecks. I favor writing custom, lightweight logic over importing massive, black-box frameworks (like LangChain or AutoGen) that obscure the underlying mechanics. The goal is determinism where it matters (system execution) and LLM creativity only where it adds value (conversation and synthesis).

## Future Directions
- Refining the "Memory Gatekeeper": a deterministic Python module that decides *which* memory layer to inject into the prompt based on the user's specific phrasing.
- Expanding the background heartbeat: allowing the system to trigger autonomous interactions based on OS events (e.g., low disk space or inactivity timeouts).
