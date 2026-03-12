# OpenClaw Security Vulnerabilities: Comprehensive Analysis

> **Author:** Black Wealth Capital Research Division
> **Status:** Rough Draft — Omega System Foundation Document
> **Last Updated:** March 2026

---

## Executive Summary

OpenClaw has been the subject of more independent security analyses than any other AI agent system, with reports from CrowdStrike [1], Trend Micro [2], Cisco [3], Snyk [4], NSFOCUS [5], Oasis Security [6], Microsoft [7], and Giskard [8]. This document consolidates all known vulnerabilities, attack vectors, and proof-of-concept exploits into a single reference, organized by severity and attack category.

---

## 1. Critical Vulnerabilities

### 1.1 Remote Code Execution (RCE) — NSFOCUS

NSFOCUS discovered two separate RCE vulnerabilities in OpenClaw that allow attackers to execute arbitrary code on the host machine [5]. These vulnerabilities exist because OpenClaw's shell tool provides unrestricted command execution capabilities, and prompt injection can be used to invoke this tool with attacker-controlled parameters.

**Attack Flow:**
1. Attacker sends a crafted message through a monitored channel (WhatsApp, Telegram, Discord)
2. Message contains hidden instructions that bypass the agent's safety guidelines
3. Agent executes a shell command specified by the attacker
4. Attacker achieves arbitrary code execution on the host machine

**Impact:** Complete compromise of the host system. The attacker can install malware, exfiltrate data, establish persistence, and use the compromised machine as a pivot point for further attacks.

**Mitigation:** Implement shell command allowlisting, sandboxed execution environments, and input sanitization for all shell tool parameters.

### 1.2 Website-to-Local Agent Takeover — Oasis Security

Oasis Security demonstrated that a malicious website can hijack the local OpenClaw agent when the agent browses the web [6]. The attack exploits the fact that web page content is ingested into the agent's context without sufficient sanitization.

**Attack Flow:**
1. Attacker creates a web page with hidden prompt injection content (invisible text, CSS-hidden elements)
2. User asks OpenClaw to browse a page that links to or embeds the malicious content
3. OpenClaw ingests the malicious instructions as part of the page content
4. Agent follows the injected instructions, potentially executing commands, sending data, or modifying its own configuration

**Impact:** Full agent takeover through a single web page visit. The attacker does not need direct access to any of the agent's communication channels.

### 1.3 Crypto Wallet Drain Attempt — CrowdStrike (Wild)

CrowdStrike documented a real-world attack observed on the Moltbook social network (OpenClaw's community platform) where an attacker attempted to drain cryptocurrency wallets [1]:

> "CrowdStrike Intelligence observed a prompt injection attack on the Moltbook social network that attempted to trick OpenClaw agents into executing cryptocurrency wallet drain operations." [1]

**Attack Flow:**
1. Attacker posts content on Moltbook containing hidden prompt injection
2. OpenClaw agents monitoring Moltbook ingest the malicious content
3. Injected instructions attempt to access cryptocurrency wallet credentials
4. If successful, the agent would execute wallet drain transactions

**Impact:** Direct financial loss through cryptocurrency theft. This is the first documented in-the-wild prompt injection attack targeting AI agents for financial gain.

---

## 2. High Severity Vulnerabilities

### 2.1 Credential Exfiltration — Cisco

Cisco researchers demonstrated that OpenClaw stores API keys and credentials in plaintext configuration files, and prompt injection can be used to read and exfiltrate these credentials [3]:

> "Personal AI agents like OpenClaw are a security nightmare. They store API keys in plaintext, have unrestricted shell access, and can be manipulated through prompt injection to exfiltrate any data they can access." [3]

**Affected Credentials:**
- LLM provider API keys (Anthropic, OpenAI, Google)
- Messaging platform tokens (WhatsApp, Telegram, Slack)
- Email credentials
- Calendar API keys
- GitHub tokens
- Any credentials stored in the agent's environment

### 2.2 Session Data Leakage — Giskard

Giskard's analysis found that sensitive data can leak across user sessions in multi-user OpenClaw deployments [8]. This occurs when conversation history from one session bleeds into another through context compaction or memory retrieval.

### 2.3 Skill Poisoning (ClawHub) — Snyk

Snyk's comprehensive audit of ClawHub found that 13.4% of all community skills contain critical security issues [4]. The breakdown:

| Category | Count | Percentage | Description |
|----------|-------|-----------|-------------|
| Prompt injection | 1,467 | 36.8% | Skills contain instructions that override agent behavior |
| Data exfiltration | 490 | 12.3% | Skills silently send data to external servers |
| Credential theft | 327 | 8.2% | Skills harvest API keys and passwords |
| Malware | 203 | 5.1% | Skills install persistent backdoors |
| **Total critical** | **534** | **13.4%** | Skills with at least one critical issue |

### 2.4 Memory Poisoning — Microsoft

Microsoft's security analysis identified that OpenClaw's file-based memory system is vulnerable to persistent prompt injection [7]. If an attacker can cause the agent to write malicious content to MEMORY.md or SOUL.md, that content persists across sessions and continues to influence agent behavior indefinitely.

---

## 3. Medium Severity Vulnerabilities

### 3.1 Identity Confusion — Microsoft

Microsoft found that OpenClaw's multi-channel architecture is vulnerable to identity confusion attacks [7]. An attacker can manipulate message prefixes to impersonate the agent owner, potentially gaining approval for actions that should require owner authorization.

### 3.2 Context Window Overflow

By sending extremely long messages or triggering extensive tool outputs, an attacker can force context compaction, potentially causing the agent to lose safety-critical instructions from its system prompt.

### 3.3 Rate Limiting Bypass

OpenClaw's default configuration does not implement rate limiting on incoming messages, allowing an attacker to flood the agent with requests and potentially trigger race conditions or resource exhaustion.

---

## 4. Mitigation Matrix

| Vulnerability | Mitigation | Implementation Complexity | Effectiveness |
|--------------|-----------|--------------------------|---------------|
| RCE | Shell command allowlisting + sandboxing | High | High |
| Website takeover | Content sanitization + injection detection | Medium | Medium |
| Credential exfiltration | Encrypted credential storage + output scanning | Medium | High |
| Session leakage | Strict session isolation + memory partitioning | Medium | High |
| Skill poisoning | Skill vetting pipeline + capability declarations | High | High |
| Memory poisoning | Memory integrity verification + write validation | Medium | Medium |
| Identity confusion | Cryptographic sender verification | Medium | High |
| Context overflow | Priority-based context management | Low | Medium |
| Rate limiting bypass | Per-channel rate limiting | Low | High |

---

## References

[1]: https://www.crowdstrike.com/en-us/blog/what-security-teams-need-to-know-about-openclaw-ai-super-agent/ "What Security Teams Need to Know About OpenClaw — CrowdStrike"
[2]: https://www.trendmicro.com/en_us/research/26/c/cisos-in-a-pinch-a-security-analysis-openclaw.html "CISOs in a Pinch: A Security Analysis of OpenClaw — Trend Micro"
[3]: https://blogs.cisco.com/ai/personal-ai-agents-like-openclaw-are-a-security-nightmare "Personal AI Agents like OpenClaw Are a Security Nightmare — Cisco"
[4]: https://snyk.io/blog/toxicskills-malicious-ai-agent-skills-clawhub/ "Snyk Finds Prompt Injection in 36%, 1467 Malicious Skills on ClawHub"
[5]: https://nsfocusglobal.com/openclaw-security-issues-add-a-security-guardrail-to-your-ai-application/ "OpenClaw Security Issues: Add a Security Guardrail — NSFOCUS"
[6]: https://www.oasis.security/blog/openclaw-vulnerability "OpenClaw Vulnerability: Website-to-Local Agent Takeover — Oasis Security"
[7]: https://www.microsoft.com/en-us/security/blog/2026/02/19/running-openclaw-safely-identity-isolation-runtime-risk/ "Running OpenClaw Safely — Microsoft Security Blog"
[8]: https://www.giskard.ai/knowledge/openclaw-security-vulnerabilities-include-data-leakage-and-prompt-injection-risks "OpenClaw Security Vulnerabilities — Giskard"
