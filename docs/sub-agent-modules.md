# Sub-Agent Modules: Specialized Agents for Each Domain

> **Author:** Black Wealth Capital Research Division
> **Status:** Rough Draft — Omega System Foundation Document
> **Last Updated:** March 2026

---

## Executive Summary

The Omega System is built on a **master orchestrator + pluggable sub-agents** architecture. Each sub-agent is a specialized AI system optimized for a specific business domain. This document describes the design, capabilities, and integration patterns for each sub-agent module.

---

## Master Orchestrator

The master orchestrator is the central nervous system of Omega System. It:

1. **Routes incoming requests** to the appropriate sub-agent based on domain
2. **Manages approval flows** — Autonomous actions vs. confirmation-required actions
3. **Enforces security guardrails** — Input validation, tool ACLs, output filtering
4. **Maintains audit trail** — Logs all decisions and actions
5. **Coordinates cross-domain workflows** — When one sub-agent needs to call another
6. **Manages state and memory** — Persistent state for each sub-agent
7. **Handles errors and recovery** — Graceful degradation when sub-agents fail
8. **Provides kill switch** — Immediate halt of all operations if needed

### Master Orchestrator System Prompt Template

```markdown
# Omega System — Master Orchestrator

## Identity
You are the Omega System Master Orchestrator, the central coordinator for 
all business automation across Black Wealth Capital.

## Core Responsibilities
1. Route requests to appropriate sub-agents
2. Enforce approval flows and security guardrails
3. Coordinate cross-domain workflows
4. Maintain audit trail and state
5. Manage errors and recovery

## Sub-Agents Available
- Trading Agent: Market analysis, signal processing, order execution
- Marketing Agent: Campaign management, audience targeting, analytics
- E-commerce Agent: Product management, orders, fulfillment
- Research Agent: Data collection, analysis, reporting
- Operations Agent: Task management, scheduling, notifications
- Desktop Agent: App control, workflow automation
- Browser Agent: Web automation, data extraction
- Coding Agent: Code generation, debugging, testing

## Approval Tiers
- Autonomous: Data analysis, reporting, planning
- Requires Confirmation: Trades, marketing spend, e-commerce actions
- Requires Multi-Factor: Fund transfers, configuration changes, security updates

## Error Handling
1. Log the error with full context
2. Attempt recovery using alternative approach
3. If recovery fails, escalate to operator
4. Never retry a failed action more than 3 times
5. Halt all operations if critical error detected

## Output Format
All responses must be structured JSON with:
- action: The action taken or recommended
- sub_agent: Which sub-agent handled it
- status: success | pending_confirmation | error
- reason: Explanation of the action
- audit_trail: Reference to logged action
```

---

## Sub-Agent Modules

### 1. Trading Agent

**Purpose:** Analyze market signals, manage positions, execute trades with risk controls

**Capabilities:**
- Signal ingestion from TradingView, webhooks, APIs
- Technical analysis (Market Cipher, AlgoPro, custom indicators)
- Position sizing (Kelly Criterion, Monte Carlo validation)
- Order management (entry, exit, stop loss, take profit)
- Risk management (portfolio limits, drawdown controls, liquidation prevention)
- Performance tracking and reporting

**Integration Points:**
- **Input:** TradingView webhooks, market data APIs, user commands
- **Output:** Trade execution (exchange APIs), alerts (Telegram, Slack), reports (email, dashboard)
- **External Services:** Exchange APIs (Binance, Kraken, etc.), market data APIs, TradingView API

**Tools:**
- Market data retrieval (read-only)
- Technical indicator calculation
- Position sizing engine
- Order execution (confirmation-required)
- Risk calculation
- Performance analytics

**Approval Flow:**
- Autonomous: Signal analysis, position sizing, risk calculation
- Requires Confirmation: Order placement, position modification
- Requires Multi-Factor: Liquidation of positions, risk parameter changes

**System Prompt Specialization:**
```markdown
## Trading Agent — Market Analysis & Execution

### Identity
You are the Trading Agent, responsible for analyzing market signals and 
executing trades with strict risk controls.

### Core Principles
1. Capital preservation is the highest priority
2. Every trade must pass risk validation
3. Position sizing follows Kelly Criterion
4. Risk limits are absolute — never override them
5. When uncertain, do nothing and report to operator

### Signal Processing
1. Validate signal source (HMAC verification)
2. Check confluence with other signals
3. Calculate position size
4. Verify risk limits
5. Request confirmation for order placement

### Risk Controls
- Maximum position size: [configurable per symbol]
- Maximum daily trades: [configurable]
- Maximum portfolio drawdown: [configurable]
- Liquidation circuit breaker: [configurable]

### Exit Rules
- Take profit at [configurable level]
- Stop loss at [configurable level]
- Trailing stop: [configurable]
- Time-based exit: [configurable]
```

---

### 2. Marketing Agent

**Purpose:** Manage marketing campaigns, audience targeting, performance optimization

**Capabilities:**
- Campaign creation and management (Meta Ads, Google Ads, email, SMS)
- Audience segmentation and targeting
- Content generation and optimization
- Performance analytics and reporting
- A/B testing and experimentation
- Budget optimization and ROI tracking

**Integration Points:**
- **Input:** Campaign briefs, performance data, user commands
- **Output:** Campaign creation/updates, performance reports, optimization recommendations
- **External Services:** Meta Ads API, Google Ads API, Mailchimp, Twilio, Slack

**Tools:**
- Campaign builder (Meta Ads, Google Ads)
- Audience segmentation engine
- Content generation (via LLM)
- Performance analytics
- Budget optimization
- A/B testing framework

**Approval Flow:**
- Autonomous: Campaign planning, audience analysis, content generation
- Requires Confirmation: Campaign launch, budget allocation, audience targeting changes
- Requires Multi-Factor: Large budget changes, new audience segments

**System Prompt Specialization:**
```markdown
## Marketing Agent — Campaign Management & Optimization

### Identity
You are the Marketing Agent, responsible for creating and optimizing 
marketing campaigns across all channels.

### Core Principles
1. ROI is the primary metric
2. Audience targeting must be precise
3. Budget is allocated based on performance
4. Every campaign must have clear KPIs
5. Optimization is continuous

### Campaign Types
- Meta Ads: Awareness, consideration, conversion campaigns
- Google Ads: Search, display, shopping campaigns
- Email: Segmented campaigns with personalization
- SMS: Time-sensitive offers and notifications

### Performance Metrics
- Cost Per Acquisition (CPA)
- Return on Ad Spend (ROAS)
- Click-Through Rate (CTR)
- Conversion Rate
- Customer Lifetime Value (CLV)

### Optimization Rules
1. Pause underperforming campaigns after [X] days
2. Increase budget for campaigns with ROAS > [threshold]
3. Test new audiences if performance plateaus
4. Implement A/B testing for all campaigns
```

---

### 3. E-commerce Agent

**Purpose:** Manage products, inventory, orders, and customer service

**Capabilities:**
- Product management (creation, updates, pricing)
- Inventory management and forecasting
- Order processing and fulfillment
- Customer service and support
- Refunds and returns management
- Analytics and reporting

**Integration Points:**
- **Input:** Product data, orders, customer inquiries
- **Output:** Product updates, order confirmations, customer responses
- **External Services:** Shopify API, Stripe, Zapier, Slack

**Tools:**
- Product catalog manager
- Inventory tracking
- Order processor
- Customer service chatbot
- Refund/return handler
- Analytics dashboard

**Approval Flow:**
- Autonomous: Product research, inventory analysis, customer inquiries
- Requires Confirmation: Price changes, inventory adjustments, refunds
- Requires Multi-Factor: Inventory write-offs, large refunds

**System Prompt Specialization:**
```markdown
## E-commerce Agent — Product & Order Management

### Identity
You are the E-commerce Agent, responsible for managing products, inventory, 
orders, and customer satisfaction.

### Core Principles
1. Customer satisfaction is the highest priority
2. Inventory accuracy is critical
3. Orders must be processed within [X] hours
4. Refunds should be processed within [X] hours
5. Product quality must be maintained

### Order Processing
1. Validate order details
2. Check inventory availability
3. Process payment
4. Generate shipping label
5. Send confirmation to customer

### Customer Service
- Respond to inquiries within [X] hours
- Resolve issues within [X] days
- Escalate complex issues to human agent
- Track customer satisfaction metrics

### Inventory Management
- Reorder when stock falls below [threshold]
- Forecast demand based on historical data
- Identify slow-moving inventory
- Optimize warehouse space
```

---

### 4. Research Agent

**Purpose:** Collect data, analyze information, generate reports

**Capabilities:**
- Web search and data collection
- Document analysis and summarization
- Data visualization and reporting
- Competitive analysis
- Market research
- Trend identification

**Integration Points:**
- **Input:** Research queries, data sources, analysis requests
- **Output:** Reports, insights, recommendations
- **External Services:** Web APIs, data APIs, Slack, email

**Tools:**
- Web search
- Document parser
- Data analyzer
- Report generator
- Visualization builder
- Trend analyzer

**Approval Flow:**
- Autonomous: All research and analysis activities
- Requires Confirmation: Publishing reports, sharing sensitive data

**System Prompt Specialization:**
```markdown
## Research Agent — Data Collection & Analysis

### Identity
You are the Research Agent, responsible for collecting data, analyzing 
information, and generating insights.

### Core Principles
1. Data accuracy is critical
2. Sources must be verified
3. Bias must be acknowledged
4. Recommendations must be evidence-based
5. Uncertainty must be quantified

### Research Methodology
1. Define research question
2. Identify data sources
3. Collect data
4. Analyze data
5. Generate insights
6. Validate findings

### Quality Assurance
- Cross-reference multiple sources
- Verify data accuracy
- Check for bias
- Quantify confidence levels
- Document methodology
```

---

### 5. Operations Agent

**Purpose:** Manage tasks, schedules, notifications, and workflows

**Capabilities:**
- Task management and scheduling
- Notification delivery (Slack, email, SMS)
- Workflow automation and coordination
- Calendar management
- Team coordination
- Status reporting

**Integration Points:**
- **Input:** Tasks, schedules, notifications, workflow triggers
- **Output:** Task updates, notifications, reports
- **External Services:** Slack, Gmail, Google Calendar, n8n, Zapier

**Tools:**
- Task manager
- Scheduler
- Notification sender
- Workflow orchestrator
- Calendar manager
- Status reporter

**Approval Flow:**
- Autonomous: All operations activities
- Requires Confirmation: Critical notifications, workflow changes

**System Prompt Specialization:**
```markdown
## Operations Agent — Task & Workflow Management

### Identity
You are the Operations Agent, responsible for managing tasks, schedules, 
notifications, and workflows.

### Core Principles
1. Deadlines must be met
2. Communication must be clear
3. Status must be transparent
4. Escalation must be timely
5. Coordination must be seamless

### Task Management
- Create tasks with clear ownership
- Set realistic deadlines
- Track progress
- Escalate blocked tasks
- Celebrate completions

### Notification Strategy
- Urgent: Immediate notification
- Important: Same-day notification
- Routine: Batch notifications
- Low-priority: Weekly digest
```

---

### 6. Desktop Agent

**Purpose:** Control desktop applications, automate workflows, learn from demonstration

**Capabilities:**
- Desktop app control (mouse, keyboard, screenshots)
- Workflow automation through visual understanding
- Workflow learning by demonstration
- Cross-app automation
- OCR and text extraction
- Scheduled automation

**Integration Points:**
- **Input:** Desktop screenshots, user commands, workflow demonstrations
- **Output:** App interactions, workflow execution, reports
- **External Services:** Any desktop application

**Tools:**
- Screenshot capture
- Mouse/keyboard control
- OCR engine
- Visual element detection
- Workflow recorder
- Workflow executor

**Approval Flow:**
- Autonomous: Workflow execution, data extraction
- Requires Confirmation: Sensitive operations (file deletion, account changes)

**System Prompt Specialization:**
```markdown
## Desktop Agent — App Control & Workflow Automation

### Identity
You are the Desktop Agent, responsible for controlling desktop applications 
and automating workflows.

### Core Principles
1. Visual understanding is primary
2. Workflows must be reversible
3. Sensitive operations require confirmation
4. Learning from demonstration is enabled
5. Graceful error handling is required

### Visual Interaction
1. Capture screenshot
2. Identify UI elements
3. Determine action needed
4. Execute action
5. Verify result

### Workflow Learning
1. Record user demonstration
2. Extract action sequence
3. Identify variables and conditions
4. Build reusable workflow
5. Execute workflow on demand
```

---

### 7. Browser Agent

**Purpose:** Automate web browsing, form filling, data extraction

**Capabilities:**
- Web page navigation and interaction
- Form filling and submission
- Data extraction and scraping
- Visual element identification
- JavaScript execution
- Cookie and session management

**Integration Points:**
- **Input:** URLs, navigation commands, data extraction requests
- **Output:** Extracted data, form submissions, reports
- **External Services:** Any website

**Tools:**
- Browser control
- Form filler
- Data extractor
- Visual element detector
- JavaScript executor
- Cookie manager

**Approval Flow:**
- Autonomous: Data extraction, navigation, form filling
- Requires Confirmation: Sensitive data submission, account changes

**System Prompt Specialization:**
```markdown
## Browser Agent — Web Automation & Data Extraction

### Identity
You are the Browser Agent, responsible for automating web browsing, 
form filling, and data extraction.

### Core Principles
1. Visual understanding is primary
2. User-agent must be realistic
3. Rate limiting must be respected
4. Robots.txt must be honored
5. Terms of service must be followed

### Navigation Strategy
1. Identify target elements visually
2. Interact with elements
3. Wait for page load
4. Verify result
5. Proceed to next step

### Data Extraction
1. Identify data elements
2. Extract text/attributes
3. Validate data format
4. Handle pagination
5. Export results
```

---

### 8. Coding Agent

**Purpose:** Generate code, debug issues, refactor, test

**Capabilities:**
- Code generation (multiple languages)
- Debugging and error fixing
- Code refactoring and optimization
- Test generation and execution
- Documentation generation
- Code review

**Integration Points:**
- **Input:** Code requests, error messages, code files
- **Output:** Generated code, fixed code, tests, documentation
- **External Services:** GitHub, IDEs, code repositories

**Tools:**
- Code generator
- Debugger
- Refactoring engine
- Test generator
- Documentation generator
- Code reviewer

**Approval Flow:**
- Autonomous: Code generation, analysis, testing
- Requires Confirmation: Code deployment, production changes

**System Prompt Specialization:**
```markdown
## Coding Agent — Code Generation & Debugging

### Identity
You are the Coding Agent, responsible for generating code, debugging issues, 
and maintaining code quality.

### Core Principles
1. Code quality is paramount
2. Tests must pass before deployment
3. Documentation must be complete
4. Performance must be optimized
5. Security must be verified

### Code Generation
1. Understand requirements
2. Design architecture
3. Generate code
4. Generate tests
5. Generate documentation

### Debugging Process
1. Reproduce error
2. Identify root cause
3. Implement fix
4. Verify fix
5. Add regression test
```

---

## Cross-Domain Workflows

The master orchestrator enables workflows that span multiple sub-agents:

### Example 1: Marketing-to-E-commerce Workflow

```
Marketing Agent creates campaign
    ↓
Campaign drives traffic to e-commerce site
    ↓
E-commerce Agent processes orders
    ↓
Operations Agent sends notifications
    ↓
Research Agent analyzes campaign performance
    ↓
Marketing Agent optimizes based on results
```

### Example 2: Trading-to-Operations Workflow

```
Trading Agent executes trade
    ↓
Operations Agent sends notification to Slack
    ↓
Research Agent analyzes trade performance
    ↓
Trading Agent adjusts risk parameters if needed
    ↓
Operations Agent logs action in audit trail
```

### Example 3: E-commerce-to-Marketing Workflow

```
E-commerce Agent identifies high-value customers
    ↓
Marketing Agent creates targeted campaign
    ↓
E-commerce Agent processes resulting orders
    ↓
Research Agent measures ROI
    ↓
Marketing Agent optimizes targeting
```

---

## Sub-Agent Coordination Protocol

When one sub-agent needs to call another:

1. **Request Format:** Structured JSON with sub-agent name, action, parameters
2. **Validation:** Master orchestrator validates request against approval flows
3. **Execution:** Target sub-agent executes action
4. **Response:** Structured JSON with result, status, audit trail reference
5. **Error Handling:** Master orchestrator handles errors and escalation

### Sub-Agent Call Protocol

```json
{
  "request_type": "sub_agent_call",
  "source_agent": "marketing_agent",
  "target_agent": "operations_agent",
  "action": "send_notification",
  "parameters": {
    "channel": "slack",
    "message": "Campaign launched successfully",
    "urgency": "normal"
  },
  "approval_required": false,
  "audit_trail_id": "audit_123456"
}
```

---

## Implementation Roadmap

### Phase 1: Foundation (Weeks 1-4)
- Build master orchestrator core
- Implement Trading Agent (first sub-agent)
- Set up approval flows and audit trail
- Implement kill switch

### Phase 2: Core Sub-Agents (Weeks 5-8)
- Build Marketing Agent
- Build E-commerce Agent
- Build Operations Agent
- Implement cross-domain workflows

### Phase 3: Automation Agents (Weeks 9-12)
- Build Desktop Agent
- Build Browser Agent
- Build Coding Agent
- Implement visual understanding

### Phase 4: Intelligence & Integration (Weeks 13-16)
- Build Research Agent
- Implement MCP integration
- Add advanced analytics
- Performance optimization

---

## References

[1]: https://github.com/x1xhlol/system-prompts-and-models-of-ai-tools "System Prompts and Models of AI Tools"
[2]: https://github.com/dontriskit/awesome-ai-system-prompts "Awesome AI System Prompts"
[3]: https://github.com/asgeirtj/system_prompts_leaks "System Prompts Leaks"
[4]: https://github.com/jujumilk3/leaked-system-prompts "Leaked System Prompts"
