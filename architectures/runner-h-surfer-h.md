# Runner H & Surfer H Architecture: Deep Dive

> **Author:** Black Wealth Capital Research Division
> **Status:** Rough Draft — ØMEGA AI Foundation Document
> **Last Updated:** March 2026

---

## Executive Summary

Runner H and Surfer H are browser-use AI agents developed by **H Company** (formerly Holistic AI), a Paris-based startup founded by former DeepMind researchers Charles Kantor and Karl Cobbe [1]. The company has raised over $220 million and operates what they describe as the world's most advanced browser-use agent — an AI system that can navigate websites, fill forms, extract data, and complete multi-step workflows entirely through browser interaction [2].

What makes H Company's approach architecturally distinctive is their development of **Hollow One**, a family of proprietary Vision-Language Models (VLMs) specifically trained for browser interaction [3]. Rather than using general-purpose models like GPT-5 or Claude and bolting on browser tools, H Company built models from the ground up that understand web page layouts, interactive elements, and navigation patterns at a fundamental level.

This document provides an architectural analysis of Runner H and Surfer H based on publicly available documentation, technical blog posts, benchmark results, and architectural descriptions from H Company [1] [2] [3] [4].

---

## 1. Product Architecture: Runner H vs. Surfer H

H Company offers two distinct products that share the same underlying model architecture but serve different deployment contexts [1]:

| Dimension | Runner H | Surfer H |
|-----------|----------|----------|
| **Deployment** | Cloud-hosted API | Chrome extension (local browser) |
| **Execution** | Remote headless browser | User's own browser session |
| **Authentication** | Credentials passed via API | Uses user's existing login sessions |
| **Use Case** | Backend automation, batch processing | Interactive assistance, real-time help |
| **Visibility** | User sees results, not execution | User watches agent work in real-time |
| **Pricing** | Per-task API pricing | Subscription model |
| **Security Model** | Isolated cloud environment | Runs in user's browser context |

### 1.1 Runner H (Cloud API)

Runner H operates as a cloud-hosted service. The user sends a task description via API, and Runner H executes it in a remote headless browser environment. The user receives results (extracted data, screenshots, confirmation of actions) without seeing the intermediate browser interactions [1].

This is architecturally similar to Manus's sandbox model — the agent operates in an isolated environment, and the user interacts through an API layer. The key difference is that Runner H is specialized for browser-based tasks, while Manus is a general-purpose agent with browser capabilities as one of many tool categories.

### 1.2 Surfer H (Chrome Extension)

Surfer H runs as a Chrome extension in the user's own browser. It can see the user's screen, understand the current page context, and take actions (clicking, typing, navigating) within the user's authenticated sessions [1].

This is architecturally similar to Vy by Vercept — a local agent that operates within the user's existing environment. The security implications are significant: Surfer H has access to everything the user's browser can access, including authenticated sessions, cookies, and local storage.

---

## 2. Hollow One: Purpose-Built Vision-Language Models

The most architecturally significant aspect of H Company's approach is **Hollow One**, their family of proprietary VLMs [3]. Rather than using general-purpose models and adding browser tools, H Company trained models specifically for browser understanding.

### 2.1 Why Purpose-Built Models Matter

General-purpose LLMs like Claude or GPT-5 understand web pages through text extraction — they receive HTML or Markdown representations of page content. This loses critical information:

| Information Type | Text Extraction | Visual Understanding |
|-----------------|-----------------|---------------------|
| **Layout** | Lost — DOM order ≠ visual order | Preserved — model sees spatial relationships |
| **Visual hierarchy** | Partially preserved via headings | Fully preserved — size, color, position |
| **Interactive elements** | Identified by tag type | Identified by visual appearance (buttons look like buttons) |
| **State indicators** | Requires parsing CSS classes | Visually obvious (checked boxes, active tabs, loading spinners) |
| **Error messages** | May be in dynamic elements not captured | Visually prominent regardless of implementation |
| **Overlays/modals** | Often missed by text extraction | Visually dominant and correctly prioritized |

Hollow One processes **screenshots** of web pages as its primary input, supplemented by structured element data. This means it understands web pages the way humans do — by looking at them [3].

### 2.2 Model Architecture

While H Company has not published full architectural details, their technical descriptions and benchmark results reveal a multi-component system [3] [4]:

**Visual Encoder:** Processes screenshots into visual embeddings that capture layout, hierarchy, and interactive element positions. This is likely based on a Vision Transformer (ViT) architecture, potentially fine-tuned on millions of web page screenshots.

**Element Localizer:** Maps visual regions to specific interactive elements. When the model decides to "click the blue Submit button," the localizer translates this into precise coordinates or element identifiers. This is the component that enables pixel-accurate interaction [3].

**Action Predictor:** Given the current page state (visual + structural), the task description, and the action history, predicts the next action. Actions include click, type, scroll, navigate, wait, and extract [3].

**Validator:** Evaluates whether an action achieved its intended effect by comparing the page state before and after execution. If the validator detects that an action failed (e.g., a form submission returned an error), it triggers a retry or alternative approach [3].

### 2.3 The Policy → Localizer → Validator Pipeline

H Company describes their execution pipeline as a three-stage process [3]:

```
Task Description + Page Screenshot
        ↓
    [Policy Model]
    "What should I do next?"
    Output: High-level action (e.g., "Click the login button")
        ↓
    [Localizer Model]
    "Where exactly is that element?"
    Output: Precise coordinates or element reference
        ↓
    [Action Execution]
    Browser performs the action
        ↓
    [Validator Model]
    "Did the action succeed?"
    Output: Success/Failure + reasoning
        ↓
    If success → next action
    If failure → retry or alternative approach
```

This pipeline is a significant architectural innovation. Most browser-use agents combine all these functions into a single model call, which leads to errors when the model correctly identifies what to do but incorrectly localizes the target element, or when it successfully executes an action but fails to verify the result [3].

**Refinement Suggestion for ØMEGA AI:** The Policy → Localizer → Validator pipeline maps directly to trading:
- **Policy Model:** "What trading action should I take?" (analyze signal, place order, adjust stop)
- **Localizer:** "Which specific instrument, exchange, and order parameters?" (BTC/USDT on Binance, limit order at $X)
- **Validator:** "Did the order execute correctly?" (check order status, verify fill price, confirm position)

### 2.4 Memory and Context Management

H Company's agents maintain a **structured memory** of their interaction history [3]:

| Memory Component | Content | Retention |
|-----------------|---------|-----------|
| **Task context** | Original task description and constraints | Full session |
| **Action history** | Sequence of actions taken with outcomes | Full session, summarized for long tasks |
| **Page state cache** | Recent page screenshots and element maps | Rolling window (last N pages) |
| **Error log** | Failed actions with error descriptions | Full session |
| **Extracted data** | Information gathered during task execution | Persisted to output |

The action history serves as a form of **episodic memory** — the agent can refer back to what it has already tried, what worked, and what failed. This prevents repetitive failures and enables learning within a single task execution [3].

---

## 3. Benchmark Performance

H Company has published benchmark results that demonstrate the effectiveness of their purpose-built approach [4]:

### 3.1 WebVoyager Benchmark

WebVoyager tests agents on real-world web tasks across diverse websites. H Company reports that Runner H achieves state-of-the-art performance, outperforming general-purpose agents that use GPT-5 or Claude with browser tools [4].

### 3.2 OSWorld Benchmark

OSWorld tests agents on operating system-level tasks (not just browser). While Runner H is specialized for browser tasks, H Company has demonstrated competitive performance on OS-level benchmarks through Surfer H's Chrome extension capabilities [4].

### 3.3 Why Specialization Wins

The benchmark results illustrate a broader architectural lesson: **specialized models outperform general-purpose models on domain-specific tasks**. A VLM trained specifically on web page interaction understands browser UI patterns that general-purpose models must learn from context or tool descriptions [3].

**Refinement Suggestion:** The ØMEGA AI should consider training or fine-tuning specialized models for chart analysis. A model trained specifically on TradingView chart screenshots would outperform a general-purpose VLM at identifying patterns, support/resistance levels, and indicator signals.

---

## 4. Security Considerations

### 4.1 Runner H (Cloud) Security Model

Runner H's cloud deployment provides natural isolation — tasks execute in remote browser environments that are destroyed after completion. However, several security concerns remain:

| Concern | Description | Mitigation |
|---------|-------------|------------|
| **Credential handling** | Users must provide login credentials for authenticated tasks | Credentials should be encrypted in transit and at rest; never logged |
| **Data exfiltration** | Agent has access to page content during execution | Output filtering to prevent sensitive data leakage |
| **Task injection** | Malicious websites could inject instructions into the agent | Input validation and prompt injection detection |
| **Session isolation** | Multiple users' tasks must be fully isolated | Separate browser instances per task |

### 4.2 Surfer H (Extension) Security Model

Surfer H's local deployment introduces more significant security concerns:

| Concern | Description | Mitigation |
|---------|-------------|------------|
| **Full browser access** | Extension can see all tabs, cookies, local storage | Strict permission scoping; user consent for each action |
| **Authenticated sessions** | Agent operates within user's login sessions | Action confirmation for sensitive operations |
| **Cross-site data** | Agent could transfer data between sites | Output monitoring and data flow controls |
| **Extension vulnerabilities** | Chrome extension attack surface | Regular security audits; minimal permissions |

---

## 5. Architectural Lessons for the ØMEGA AI

1. **Purpose-built models outperform general-purpose models** on domain-specific tasks. Consider fine-tuning models specifically for chart analysis and trading signal interpretation.

2. **The Policy → Localizer → Validator pipeline** separates decision-making from execution from verification. This three-stage pattern should be applied to every trading action.

3. **Visual understanding** captures information that text extraction misses. Chart analysis should process screenshots, not just numerical data, to capture patterns that humans see visually.

4. **Structured memory** (task context + action history + error log) enables within-session learning. The ØMEGA AI should maintain similar structured memory for each trading session.

5. **Specialization at the model level** is more effective than tool-level adaptation. If possible, fine-tune models on trading-specific data rather than relying entirely on prompt engineering.

6. **Cloud vs. local deployment** is a fundamental architectural choice with security implications. For trading, cloud deployment (like Runner H) provides better isolation, while local deployment (like Surfer H) provides lower latency for time-sensitive operations.

---

## References

[1]: https://www.hcompany.ai/ "H Company — Official Website"
[2]: https://medium.com/@kram254/runner-h-surfer-h-a-masterclass-in-modern-browser-use-agents-fb68cb666b29 "Runner H & Surfer H: A Masterclass in Modern Browser-Use Agents"
[3]: https://docs.hcompany.ai/architecture "H Company Technical Documentation"
[4]: https://www.hcompany.ai/benchmarks "H Company Benchmark Results"
