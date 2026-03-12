# Vy by Vercept Architecture: Deep Dive

> **Author:** Black Wealth Capital Research Division
> **Status:** Rough Draft — Omega System Foundation Document
> **Last Updated:** March 2026

---

## Executive Summary

Vy is a computer-use AI agent developed by **Vercept**, a startup that was acquired by Anthropic in 2026 [1]. Vy operates as a native desktop application (originally macOS, later cross-platform) that uses **visual-first architecture** — it sees the user's screen, understands UI elements through computer vision, and executes tasks by controlling the mouse and keyboard, exactly as a human would [2] [3].

Vy represents a fundamentally different approach from both OpenClaw (which operates through APIs and shell commands) and Runner H (which operates through a dedicated browser). Vy interacts with **any application** on the user's computer by understanding the visual interface, making it application-agnostic and API-independent [2].

The acquisition by Anthropic signals that visual-first computer use is considered a strategically important capability for the next generation of AI agents. This document analyzes Vy's architecture and extracts patterns applicable to the Omega System.

---

## 1. Visual-First Architecture

### 1.1 The Core Principle

Vy's defining architectural decision is to treat the computer screen as the primary input modality [2]. Rather than interfacing with applications through APIs, command-line tools, or browser automation frameworks, Vy:

1. **Captures screenshots** of the user's screen at regular intervals
2. **Analyzes the visual content** using computer vision and VLMs to understand what is displayed
3. **Identifies interactive elements** (buttons, text fields, menus, links) by their visual appearance
4. **Executes actions** by controlling the mouse cursor and keyboard

This approach has a profound architectural implication: **Vy works with any application that has a visual interface**, regardless of whether that application exposes an API, supports automation frameworks, or was designed for programmatic access [2].

| Approach | Vy (Visual-First) | API-Based (OpenClaw) | Browser-Based (Runner H) |
|----------|-------------------|---------------------|-------------------------|
| **Application coverage** | Any GUI application | Only apps with APIs/CLIs | Only web applications |
| **Setup required** | None — works immediately | API keys, configurations | Website-specific adapters |
| **Robustness to UI changes** | Adapts visually | Breaks on API changes | Breaks on DOM changes |
| **Speed** | Slower (screenshot + vision) | Fast (direct API calls) | Medium (DOM manipulation) |
| **Accuracy** | Depends on vision model quality | High (structured data) | Medium (DOM parsing) |
| **Access to hidden state** | Limited to visible content | Full programmatic access | Full DOM access |

### 1.2 Screen Understanding Pipeline

Vy's screen understanding pipeline processes each screenshot through multiple stages [2] [3]:

**Stage 1: Screen Capture**
Vy captures the current screen state as a high-resolution screenshot. The capture frequency is adaptive — higher during active task execution, lower during idle monitoring.

**Stage 2: UI Element Detection**
A specialized computer vision model identifies all interactive elements on the screen: buttons, text fields, dropdown menus, checkboxes, links, tabs, and other controls. Each element is classified by type and assigned a bounding box [2].

**Stage 3: Text Recognition (OCR)**
Optical character recognition extracts all visible text, including labels, headings, body text, error messages, and status indicators. This text is associated with its spatial position on the screen [2].

**Stage 4: Semantic Understanding**
A VLM processes the screenshot along with the detected elements and extracted text to build a semantic understanding of the current application state. This includes:
- What application is active
- What the user is currently doing
- What options are available
- What the current state of any ongoing process is

**Stage 5: Action Planning**
Given the task description and the current screen state, the model plans the next action. Actions include mouse clicks, keyboard input, scrolling, drag-and-drop, and keyboard shortcuts [2].

### 1.3 Workflow Learning

One of Vy's most distinctive capabilities is **workflow learning** — the ability to observe a user performing a task once and then automate that task going forward [2]:

> "Vy can observe a user's workflow once and then automate it, mimicking human interaction with on-screen elements." [2]

This is architecturally significant because it means Vy can learn new capabilities without any programming or configuration. The user simply demonstrates the workflow, and Vy records the sequence of visual states and actions, creating a replayable automation [2].

**Refinement Suggestion for Omega System:** Workflow learning could be applied to chart analysis. A trader could demonstrate their analysis process on a TradingView chart — identifying support/resistance, checking indicators, marking entry/exit points — and the agent could learn to replicate this process on new charts.

---

## 2. Privacy-Centric Design

Vy's privacy architecture is a key differentiator [2] [3]:

### 2.1 Local Processing

Vy prioritizes processing on the user's local machine, minimizing data sent to external servers. Screen captures are processed locally whenever possible, with cloud processing used only for complex reasoning tasks that exceed local model capabilities [3].

### 2.2 Minimal Data Retention

Vy does not retain user data unless explicitly instructed. Screen captures are processed and discarded — they are not stored, logged, or used for training [3].

### 2.3 No Credential Access

Vy does not request or store login credentials for any application. It operates within the user's existing authenticated sessions, using the visual interface to interact with applications the user is already signed into [2].

| Privacy Feature | Implementation | Benefit |
|----------------|---------------|---------|
| **Local processing** | On-device vision models | Screen content never leaves the machine |
| **No data retention** | Process-and-discard pipeline | No persistent record of user activity |
| **No credential access** | Uses existing sessions | No password storage or management |
| **Explicit consent** | User initiates every task | No autonomous surveillance |
| **Transparent execution** | User watches agent work | Full visibility into agent actions |

**Refinement Suggestion:** The Omega System should adopt Vy's privacy principles for sensitive financial data. Trading account credentials should never be stored in agent memory. Instead, agents should operate within pre-authenticated API sessions with scoped permissions.

---

## 3. Cross-Application Integration

### 3.1 The "@" Mention System

Vy implements a cross-application reference system using the "@" symbol [2]:

> "Typing '@' allows direct interaction with browser tabs or documents, eliminating the need for manual navigation or switching between apps." [2]

This enables workflows that span multiple applications. For example, a user could say: "Take the data from @spreadsheet and create a presentation in @slides with charts showing the quarterly trends."

### 3.2 Application-Agnostic Automation

Because Vy interacts through the visual interface, it can automate workflows across applications that have no API integration with each other. A workflow might involve:

1. Reading data from a PDF viewer
2. Entering that data into a web form
3. Taking a screenshot of the result
4. Pasting it into an email

No API integration is needed between any of these applications — Vy bridges them through visual interaction [2].

---

## 4. Scheduled Automation

Vy supports scheduled task execution, enabling workflows that run automatically at predefined times [2]:

| Schedule Type | Example | Use Case |
|--------------|---------|----------|
| **Time-based** | "Every morning at 9am" | Generate daily reports |
| **Event-based** | "When a new email arrives from X" | Auto-process specific emails |
| **Interval-based** | "Every 2 hours" | Check and update dashboards |
| **Conditional** | "When the spreadsheet is updated" | Trigger downstream workflows |

This is functionally similar to OpenClaw's heartbeat system but with a key difference: Vy's scheduled tasks operate through the visual interface, meaning they can automate applications that don't support cron-style automation [2].

**Refinement Suggestion:** The Omega System should implement visual-based monitoring for platforms that lack API access. For example, if a trading platform only provides a web interface without API, a Vy-style visual agent could monitor the platform's UI for position changes, alert conditions, or execution confirmations.

---

## 5. Anthropic Acquisition and Strategic Implications

Vercept's acquisition by Anthropic in 2026 [1] signals several strategic directions:

1. **Computer use is a priority** — Anthropic is investing in agents that can interact with desktop applications, not just text-based interfaces.

2. **Visual-first is the future** — The acquisition validates the approach of using computer vision as the primary interface modality, rather than APIs or DOM manipulation.

3. **Privacy-centric design matters** — Vercept's emphasis on local processing and minimal data retention aligns with Anthropic's stated commitment to AI safety.

4. **Cross-platform capability** — Vy's expansion from macOS to Windows before the acquisition suggests that cross-platform computer use is a key capability.

---

## 6. Architectural Comparison with Other Agents

| Dimension | Vy (Vercept) | OpenClaw | Manus | Runner H |
|-----------|-------------|----------|-------|----------|
| **Primary interface** | Visual (screenshots) | API/CLI/MCP | Sandbox tools | Browser DOM + vision |
| **Application scope** | Any GUI app | API-accessible apps | Sandbox environment | Web applications |
| **Execution location** | User's machine | User's machine | Cloud sandbox | Cloud (Runner H) / Local (Surfer H) |
| **Learning method** | Workflow observation | Skill files (SKILL.md) | System prompt + modules | Pre-trained VLM |
| **Privacy model** | Local-first, no retention | File-based, local storage | Cloud-hosted, sandboxed | Cloud-processed |
| **Proactive behavior** | Scheduled automation | Heartbeat system | Task-triggered | Task-triggered |
| **Model type** | Proprietary VLM | General-purpose LLM | General-purpose LLM | Hollow One VLM |

---

## 7. Key Architectural Lessons for the Omega System

1. **Visual-first interaction** enables automation of applications without APIs. This is valuable for monitoring trading platforms, chart analysis tools, and market data dashboards that may not offer programmatic access.

2. **Workflow learning** allows non-technical users to teach the agent new capabilities by demonstration. Traders could demonstrate their analysis process, and the agent could learn to replicate it.

3. **Privacy-centric design** is essential for financial applications. Screen captures of trading accounts should never be stored or transmitted unnecessarily.

4. **Cross-application workflows** enable complex multi-tool analysis. A trading workflow might span TradingView (chart analysis), CoinGecko (fundamental data), Binance (order execution), and a spreadsheet (portfolio tracking).

5. **Scheduled automation** enables proactive monitoring without requiring always-on human attention. Market monitoring, position health checks, and signal aggregation can all be scheduled.

6. **The acquisition signal** — Anthropic's acquisition of Vercept validates visual-first computer use as a strategically important capability. The Omega System should invest in visual chart analysis capabilities.

---

## References

[1]: https://vercept.com/ "Vercept is joining Anthropic — Official Announcement"
[2]: https://aiadoptionagency.com/vercept-and-vy-redefining-human-computer-interaction-through-ai/ "Vercept and Vy: Redefining Human-Computer Interaction Through AI"
[3]: https://medium.com/@sandeepsd245/ditch-the-clicks-talk-to-your-mac-is-vy-by-vercept-the-future-of-how-we-use-computers-2a86cc86b62d "Ditch the Clicks, Talk to Your Mac: Is Vy by Vercept the Future?"
[4]: https://www.fastcompany.com/91447623/ai-control-pc-vercept-vy-browser "I let AI control my entire PC. Here's what happened. — Fast Company"
[5]: https://www.superbcrew.com/vy-by-vercept-uses-advanced-ui-understanding-to-complete-tasks-on-your-mac-just-like-you-would/ "Vy by Vercept Uses Advanced UI Understanding — SuperbCrew"
