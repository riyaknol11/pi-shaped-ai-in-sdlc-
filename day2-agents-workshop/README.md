# Meeting Scheduler AI Agent


## Use Case Definition

### Problem Statement

Scheduling meetings via email is a time-consuming process that typically requires 5-8 email exchanges to find a mutually agreeable time. Professionals waste 15-30 minutes per meeting on coordination, manually checking calendars and composing confirmation emails.

### Solution

An AI-powered meeting scheduler agent that automates the scheduling workflow by:
- Checking calendar availability against existing commitments
- Suggesting optimal time slots based on working hours and preferences
- Drafting professional confirmation emails
- Sending automated email notifications via SMTP

### Target Users

- Busy executives and managers receiving multiple meeting requests
- Executive assistants managing calendars for multiple people
- Sales professionals scheduling client meetings
- Remote teams coordinating across time zones

### Example User Interactions

1. "When am I free next week for a 1-hour meeting?"
2. "I need to schedule a call with John about Q2 planning on Wednesday"
3. "Book Tuesday at 10am and send confirmation to sarah@example.com"
4. "What meetings did we just schedule?"
5. "Find me three available slots tomorrow afternoon"

---

## System Architecture

The workflow implements a multi-stage pipeline architecture:

```
User Input (Chat)
    ↓
AI Agent (Google Gemini 1.5 Flash)
    ├─→ Calendar Checker Tool (Code)
    ├─→ Calculator Tool (Duration calculations)
    └─→ Memory (Window Buffer - 10 messages)
    ↓
Response Processing
    ↓
Email Data Parser (Code Node)
    ↓
SMTP Email Sender (Gmail)
```

This architecture separates concerns:
- **AI Agent**: Reasoning and decision-making
- **Tools**: External data access and computation
- **Parser**: Data transformation
- **SMTP**: Email delivery

---

## LLM Selection

**Selected Model:** Google Gemini 1.5 Flash

### Rationale

- **Cost-effective**: Free tier provides 1,500 requests per day, suitable for development and testing
- **Fast response times**: Average 2-3 second latency for scheduling queries
- **Strong tool integration**: Native support for function calling and tool use
- **Sufficient reasoning capability**: Handles multi-step scheduling logic effectively
- **No billing setup required**: Immediate availability with API key

### Alternatives Considered

- **GPT-4o-mini**: Excellent quality but requires billing setup and credit card
- **Claude Sonnet**: Superior reasoning but higher cost per request
- **Decision**: Gemini Flash provides optimal balance of capability, speed, and accessibility for this use case

---

## Workflow Components

### 1. Chat Trigger
- **Type:** n8n Chat Trigger node
- **Purpose:** Initiates conversation and captures user input
- **Configuration:** Webhook-based trigger for real-time interaction

### 2. AI Agent
- **Type:** n8n AI Agent node with Google Gemini
- **Function:** Orchestrates tool use and generates responses
- **System Prompt:** Defines agent behavior, tool usage patterns, and output formatting

### 3. Google Gemini Chat Model
- **Model:** gemini-1.5-flash
- **Temperature:** 0.5 (balanced creativity and consistency)
- **Max Tokens:** 2048

### 4. Calendar Checker Tool
- **Type:** Code Tool (JavaScript)
- **Function:** Simulates calendar database with existing meetings
- **Returns:** Available time slots formatted as human-readable strings
- **Simulated Schedule:**
  - Monday 10am-11am: Team Standup
  - Tuesday 2pm-3pm: Client Call
  - Wednesday 11am-12pm: 1:1 Manager
  - Thursday 9am-10am: Planning Session
  - Lunch block: 12pm-1pm (all days)

### 5. Calculator Tool
- **Type:** n8n Calculator Tool
- **Function:** Performs time duration calculations and arithmetic operations

### 6. Window Buffer Memory
- **Type:** n8n Memory node
- **Capacity:** 10 messages
- **Function:** Maintains conversation context for follow-up queries

### 7. Email Data Parser
- **Type:** Code node
- **Function:** Extracts structured email data (to, subject, body) from agent response
- **Output:** JSON object for SMTP node consumption

### 8. SMTP Email Sender
- **Type:** n8n Send Email node
- **Configuration:** Gmail SMTP (smtp.gmail.com:465)
- **Authentication:** App Password (16-character Google-generated password)

![alt text](<screenshots/Screenshot from 2026-04-23 14-22-56.png>)

![alt text](<screenshots/Screenshot from 2026-04-23 14-21-21.png>)

![alt text](<screenshots/Screenshot from 2026-04-23 14-04-07.png>)
---

## Setup Instructions

### Prerequisites

- n8n instance (local or cloud deployment)
- Google Gemini API key
- Gmail account with 2-Step Verification enabled
- Gmail App Password

### Step 1: Obtain Google Gemini API Key

1. Navigate to: https://aistudio.google.com/app/apikey
2. Sign in with Google account
3. Click "Create API Key"
4. Copy and securely store the key

### Step 2: Generate Gmail App Password

1. Enable 2-Step Verification: https://myaccount.google.com/security
2. Navigate to: https://myaccount.google.com/apppasswords
3. Select app: "Mail", device: "Other (n8n)"
4. Click "Generate"
5. Copy the 16-character password

### Step 3: Import Workflow

1. Download `meeting-scheduler-workflow.json`
2. In n8n: Navigate to Workflows → Import from File
3. Select the downloaded JSON file

### Step 4: Configure Credentials

**Google Gemini:**
1. Click on "Google Gemini Chat Model" node
2. Add credential with API key from Step 1

**Gmail SMTP:**
1. Click on "Send Email" node
2. Create new SMTP credential:
   - Host: smtp.gmail.com
   - Port: 465
   - Security: SSL/TLS
   - User: your-email@gmail.com
   - Password: App Password from Step 2

### Step 5: Activate and Test

1. Save the workflow
2. Activate using the toggle switch
3. Open chat interface
4. Test with: "When am I free tomorrow?"

---

## Testing Scenarios

### Test 1: Calendar Availability Query

**Input:** "When am I free next week for a 1-hour meeting?"

**Expected Behavior:**
- Agent invokes calendar_checker tool
- Returns 3-5 available time slots
- Formats times in 12-hour AM/PM format
- Avoids existing meetings and lunch hour

**Actual Result:** Successfully retrieved and formatted available slots

### Test 2: Meeting Booking with Memory

**Input (Turn 1):** "I need to schedule a call with John about Q2 planning"

**Input (Turn 2):** "Book the 10am slot on Tuesday"

**Expected Behavior:**
- Agent remembers context from Turn 1 (who: John, topic: Q2 planning)
- Uses memory to reference "the 10am slot" without re-asking
- Confirms booking with complete details

**Actual Result:** Memory successfully retained context across turns

### Test 3: Email Confirmation Sending

**Input:** "Send confirmation to sarah@example.com"

**Expected Behavior:**
- Agent drafts professional email with meeting details
- Parser extracts TO, SUBJECT, BODY fields
- IF node routes to SMTP sender
- Email delivered to recipient

**Actual Result:** Email successfully sent and received in inbox

### Test 4: Multi-turn Conversation

**Input (Turn 1):** "When am I free tomorrow?"  
**Input (Turn 2):** "What about Wednesday instead?"  
**Input (Turn 3):** "Book the 2pm slot"

**Expected Behavior:**
- Context maintained across 3 turns
- Agent understands "Wednesday instead" refers to meeting discussion
- "The 2pm slot" resolved without ambiguity

**Actual Result:** All contextual references correctly understood

---

## Evaluation and Reflection

### What does your agent do well?

1. **Tool Orchestration:** Successfully coordinates multiple tools (calendar checker, calculator, email sender) without manual intervention

2. **Context Retention:** Window buffer memory enables natural multi-turn conversations where users can reference previous exchanges without repeating information

3. **Professional Output:** Generates well-formatted, business-appropriate email confirmations with proper structure and tone

4. **Availability Checking:** Accurately filters available time slots against simulated calendar conflicts and business hours constraints

5. **Architectural Separation:** Clean separation between AI reasoning (agent), data transformation (parser), and email delivery (SMTP) mirrors production system design

### Current Limitations

1. **Simulated Calendar:** Uses hardcoded meeting schedule instead of integrating with real calendar APIs (Google Calendar, Outlook)

2. **Single User Scope:** Cannot check availability across multiple participants or find mutual free time

3. **Limited Memory Window:** 10-message buffer loses context in extended conversations, requiring users to repeat information

4. **No Time Zone Handling:** Assumes all times are in user's local timezone, cannot handle international scheduling

5. **Static Working Hours:** Hardcoded 9am-5pm schedule does not adapt to user preferences or role requirements

6. **No Rescheduling Logic:** Cannot handle "move the Tuesday meeting to Wednesday" type requests

7. **Basic Conflict Detection:** Only checks exact hour blocks, does not account for meeting buffer time or travel time

### What would you improve or extend with more time?

**Near-term Improvements (1-2 weeks):**
1. Integrate Google Calendar API for real-time availability checking
2. Implement time zone detection and conversion for international meetings
3. Add meeting duration flexibility (15-min, 30-min, 2-hour slots)
4. Expand memory window to 20 messages or implement conversation summarization

**Medium-term Extensions (1-2 months):**
5. Multi-participant scheduling with availability aggregation
6. Rescheduling and cancellation workflows
7. Meeting preferences learning (preferred times, buffer requirements)
8. Calendar conflict warnings with override options

**Long-term Vision (3-6 months):**
9. Integration with video conferencing platforms (Zoom, Meet link generation)
10. Automatic meeting preparation (agenda creation, document gathering)
11. Post-meeting follow-up automation (notes, action items)
12. Analytics dashboard (meetings scheduled, time saved, patterns)

### How does memory improve the agent's usefulness in your use case?

Memory is critical for meeting scheduling because:

1. **Reduces Cognitive Load:** Users can say "book the 10am slot" instead of "book a meeting for Tuesday, April 29 at 10:00 AM with John about Q2 planning"

2. **Enables Natural Follow-ups:** Questions like "what about Wednesday instead?" or "send confirmation to the attendees" are understood in context

3. **Maintains Meeting Details:** Agent remembers who the meeting is with, the topic, and the proposed times throughout the conversation

4. **Supports Iterative Refinement:** Users can progressively narrow down preferences ("I need a slot next week" → "Wednesday afternoon" → "the 2pm option") without starting over

5. **Prevents Redundancy:** Eliminates need to re-specify meeting purpose or participants when asking follow-up questions

**Without memory:**
- User: "Book the 2pm slot"
- Agent: "Which day and with whom?"

**With memory:**
- User: "Book the 2pm slot"
- Agent: "Confirmed: Meeting with John about Q2 planning on Tuesday at 2:00 PM"

### Did the tool behave as expected? Were there any edge cases?

**Expected Behaviors:**

- **Calendar Checker:** Consistently returned available slots, correctly avoided conflicts
- **Calculator:** Accurate time duration calculations
- **Email Sender:** Successfully delivered formatted emails via SMTP
- **Memory:** Retained context across conversation turns as designed

**Edge Cases Identified:**

1. **Ambiguous Time References**
   - Input: "Schedule a meeting next Monday"
   - Issue: "Next Monday" could mean different dates depending on current day of week
   - Handling: Agent should confirm exact date before booking

2. **Missing Email Recipient**
   - Input: "Send confirmation email" (no recipient specified)
   - Issue: Parser fails to extract 'TO' field
   - Current behavior: IF node routes to false path, no email sent
   - Improvement: Agent should ask for recipient email

3. **Multiple Requests in Single Message**
   - Input: "Check availability tomorrow and Wednesday, then send confirmation to both sarah@example.com and john@example.com"
   - Issue: Single email parser cannot handle multiple recipients
   - Current behavior: Only first email processed
   - Improvement: Support comma-separated recipient lists

4. **Calendar Checker Returns Empty**
   - Scenario: Request availability for date with no free slots
   - Current behavior: Returns empty array
   - Improvement: Agent should proactively suggest next available day

5. **Memory Overflow**
   - Scenario: Conversation exceeds 10 messages
   - Issue: Earliest context lost (e.g., meeting topic mentioned in message 1)
   - Mitigation: Implement sliding window with summary of key details

6. **Timezone in Email Body**
   - Input: User in India scheduling with US-based participant
   - Issue: Email confirmation shows only single timezone
   - Improvement: Include both timezones in confirmation

---

## Core Concept Questions

### 1. What is the difference between a simple LLM call and an LLM-powered agent?

**Simple LLM Call:**
- Single request-response cycle with no external data access
- Limited to knowledge within training data (cutoff date)
- Stateless - no memory between interactions
- Cannot perform actions or access real-time information
- Example: "What is the capital of France?" → "Paris"

**LLM-Powered Agent:**
- Can invoke tools to access external systems (calendars, databases, APIs)
- Maintains conversation memory across multiple turns
- Executes multi-step reasoning workflows (check calendar → suggest times → send email)
- Can take actions in real world (create calendar events, send emails)
- Example: "Schedule a meeting next week" → checks calendar → suggests 3 times → books confirmed slot → sends email

**In this workflow:**
- Simple LLM would only describe how meeting scheduling works
- Agent actually checks calendar, suggests real times, and sends confirmation emails

The key difference is **agency** - the ability to perceive, reason, and act autonomously using available tools.

### 2. What role does the system prompt play in shaping agent behaviour?

The system prompt serves as the agent's **operational manual**, defining:

**1. Identity and Capabilities**
- Establishes agent role: "You are an intelligent meeting scheduling assistant"
- Lists available tools: calendar_checker, calculator, email_sender
- Sets behavioral constraints: working hours, lunch blocks, email format

**2. Tool Usage Patterns**
- Specifies when to invoke each tool ("use calendar_checker when user asks about availability")
- Defines expected input/output formats for tools
- Establishes tool calling precedence

**3. Response Formatting**
- Dictates email structure (EMAIL_DATA: TO/SUBJECT/BODY format)
- Specifies time format (12-hour AM/PM, not 24-hour)
- Defines confirmation message templates

**4. Decision Logic**
- Guides multi-step workflows (check → suggest → confirm → send)
- Sets conflict resolution rules (avoid lunch, respect working hours)
- Defines edge case handling

**Impact on this workflow:**
- Without "use EMAIL_DATA format" instruction, parser would fail to extract email fields
- Without "suggest 3 time slots" guideline, agent might suggest only 1 or too many
- Without "12-hour AM/PM" specification, times might be formatted inconsistently

The system prompt is the **primary control mechanism** for shaping agent behavior without modifying code.

### 3. How does tool use extend what an LLM can do on its own?

**LLM Alone (Limitations):**
- Cannot access information after training cutoff date
- Cannot retrieve user-specific data (personal calendar, emails)
- Cannot perform precise calculations reliably (e.g., "2 weeks from Tuesday" date arithmetic)
- Cannot take actions in external systems (send emails, create events)
- Cannot verify current state (is this time slot actually free?)

**LLM + Tools (Extended Capabilities):**

**1. Calendar Checker Tool:**
- Problem: LLM cannot know user's actual schedule
- Solution: Tool queries simulated calendar database
- Result: Agent suggests times that avoid real conflicts

**2. Calculator Tool:**
- Problem: LLMs approximate math, may make errors
- Solution: Tool executes precise arithmetic
- Result: Accurate meeting duration calculations

**3. Email Sender (SMTP):**
- Problem: LLM cannot send actual emails
- Solution: Tool connects to SMTP server
- Result: Confirmation emails delivered to recipients

**Transformation in this workflow:**
- **Without tools:** Agent could only describe meeting scheduling process theoretically
- **With tools:** Agent performs actual scheduling - checks availability, books slots, sends confirmations

Tools transform the LLM from a **knowledge base** into an **action-taking assistant**.

### 4. What are the trade-offs between short-term (window buffer) and long-term (database) memory in agents?

| Aspect | Window Buffer Memory | Database Memory (Postgres/Redis) |
|--------|---------------------|----------------------------------|
| **Persistence** | Session-only (lost on restart) | Survives application restarts |
| **Capacity** | Last N messages (10 in this workflow) | Unlimited historical conversations |
| **Setup Complexity** | Zero configuration required | Requires database installation and schema design |
| **Performance** | Fastest (in-memory storage) | Slower (disk I/O and network latency) |
| **Cost** | Free (RAM allocation) | Database hosting and storage costs |
| **Privacy** | Auto-deleted after session | Requires data retention policies and GDPR compliance |
| **Use Cases** | Single scheduling conversation | Long-term executive assistant tracking patterns |

**Trade-offs for this workflow:**

**Current Choice (Window Buffer):**
- **Advantages:**
  - Sufficient for typical scheduling conversation (3-7 messages to book meeting)
  - No infrastructure overhead
  - Automatic privacy compliance (no persistent storage)
  
- **Disadvantages:**
  - Cannot answer "What meetings did I schedule last week?"
  - Lost context if user returns hours later
  - Cannot learn user's scheduling preferences over time

**Database Memory Would Enable:**
- "Schedule my weekly 1:1 with Sarah" (remembers recurring pattern)
- "What's on my calendar this month?" (historical query)
- "Find a time that works like last time" (pattern matching)

**Decision Rationale:**
Window buffer chosen because:
1. Most scheduling conversations complete within 10 messages
2. Simplifies deployment (no database dependency)
3. Appropriate for demonstration and testing
4. Can upgrade to database memory in production if needed

### 5. What is the purpose of the AI Agent node in n8n compared to a simple LLM Chain node?

**LLM Chain Node:**
```
User Input → LLM → Response
```
- Linear, single-pass processing
- No tool integration
- No decision-making about actions
- Suitable for: text generation, summarization, translation

**AI Agent Node:**
```
User Input → Agent [Analyze → Select Tool → Execute → Observe → Repeat] → Response
```
- Autonomous tool selection and invocation
- Multi-step reasoning loops
- Can retry or use different strategies
- Suitable for: task automation, data retrieval, multi-step workflows

**Concrete Comparison with Meeting Scheduling:**

**LLM Chain Approach:**
```
User: "Schedule a meeting with John next week"
LLM Chain: "I recommend scheduling the meeting on Tuesday or Wednesday afternoon, 
            as those are typically good times for professional meetings."
```
Result: Generic advice based on training data, no actual scheduling

**AI Agent Approach:**
```
User: "Schedule a meeting with John next week"
Agent Process:
1. Invokes calendar_checker tool → retrieves actual availability
2. Identifies 3 free slots: Tue 10am, Wed 2pm, Thu 11am
3. Responds: "Based on your calendar, you're free: [3 specific times]"
4. User: "Book Tuesday 10am and email John"
5. Invokes email_sender → sends actual confirmation
```
Result: Actual scheduling with real data and actions

**Purpose of AI Agent Node:**
- Enables **tool use** without hardcoding workflow paths
- Provides **autonomous decision-making** (which tool to use when)
- Supports **iterative reasoning** (if first approach fails, try another)
- Manages **state** across multi-step operations

In this workflow, the Agent node orchestrates calendar checking, email drafting, and memory management dynamically based on conversation flow.

### 6. What risks or failure modes did you observe when testing your agent?

**Observed Failure Modes:**

**1. Tool Invocation Errors**
- **Symptom:** Agent attempts to call calendar_checker with malformed input
- **Cause:** User query too ambiguous ("find me a time")
- **Impact:** Tool returns error, agent unable to suggest times
- **Mitigation:** Improved system prompt with input validation guidance

**2. Email Parser Failures**
- **Symptom:** Parser unable to extract TO field from agent response
- **Cause:** Agent generated email in free-form text instead of structured EMAIL_DATA format
- **Impact:** IF node routes to false path, email not sent
- **Observed when:** Agent given vague instruction like "notify them"
- **Mitigation:** Explicit system prompt instruction to use EMAIL_DATA format

**3. Memory Context Loss**
- **Symptom:** After 10+ messages, agent forgets meeting topic mentioned early in conversation
- **Cause:** Window buffer overflow (only retains last 10 messages)
- **Impact:** User must repeat information
- **Example:** Message 1: "Schedule Q2 planning meeting", Message 12: "What was the topic?" → Agent doesn't remember
- **Mitigation:** Increase buffer size or implement summarization

**4. SMTP Authentication Failures**
- **Symptom:** Email send fails with "Invalid credentials" error
- **Cause:** Gmail App Password entered with spaces (16 characters must be continuous)
- **Impact:** Workflow succeeds until SMTP node, then fails silently
- **Mitigation:** Input validation on credential setup, clear error messages

**5. Ambiguous Time Reference Resolution**
- **Symptom:** "Next Monday" scheduled for wrong date
- **Cause:** Calendar checker interprets relative dates based on execution time
- **Impact:** Meeting booked for unintended date
- **Observed:** Testing on Friday evening - "next Monday" ambiguous (in 3 days vs 10 days)
- **Mitigation:** Agent should confirm absolute date before booking

**6. Concurrent Booking Race Condition**
- **Symptom:** Two users book same time slot simultaneously
- **Cause:** Calendar checker reads state, then SMTP sends, no locking mechanism
- **Impact:** Double-booking in production scenario
- **Mitigation:** Implement optimistic locking or calendar API integration with conflict detection

**7. Tool Timeout**
- **Symptom:** Calendar checker occasionally times out on complex queries
- **Cause:** JavaScript execution limit in Code node
- **Impact:** Agent unable to provide availability, workflow fails
- **Mitigation:** Optimize tool code, implement retry logic

**Most Critical Risk:**
Email sent with incorrect details (wrong time, wrong recipient) due to parsing error. Mitigation requires validation layer before SMTP node.

### 7. How would you evaluate whether your agent is performing well in a real production setting?

**Evaluation Framework:**

**1. Functional Accuracy Metrics**

**Meeting Scheduling Accuracy:**
- Metric: Percentage of suggested times that are actually conflict-free
- Target: 100% (zero false positives)
- Measurement: Compare suggested slots against actual calendar state
- Current workflow: 100% accuracy with simulated calendar

**Email Delivery Success Rate:**
- Metric: Percentage of intended emails successfully delivered
- Target: >99.5%
- Measurement: SMTP delivery confirmations vs. send attempts
- Tool: Email delivery logs, bounce tracking

**Context Retention Accuracy:**
- Metric: Percentage of follow-up queries correctly resolved using memory
- Target: >90% within 10-message window
- Measurement: Manual review of multi-turn conversations
- Test cases: "Book the 2pm slot", "Send to the same person", "What did we schedule?"

**2. User Experience Metrics**

**Task Completion Rate:**
- Metric: Percentage of conversations resulting in successful booking
- Target: >80%
- Measurement: Track from initial request to confirmed meeting
- Failure reasons: User abandons, no suitable time found, email delivery fails

**Time to Schedule:**
- Metric: Average time from first message to confirmed booking
- Baseline: Manual scheduling takes 15-30 minutes (5-8 email exchanges)
- Target: <3 minutes with agent
- Measurement: Timestamp analysis (first user message → booking confirmation)

**User Satisfaction Score:**
- Metric: Post-interaction rating (1-5 stars)
- Target: >4.2 average
- Collection method: "How helpful was the scheduling assistant?" prompt after booking

**3. Technical Performance Metrics**

**Response Latency:**
- Metric: Time from user message to agent response
- Target: <5 seconds (P95), <3 seconds (P50)
- Measurement: Webhook receive timestamp to response sent timestamp
- Breakdown: LLM inference time, tool execution time, SMTP send time

**Tool Invocation Success Rate:**
- Calendar Checker: >99% success
- Email Sender: >99% success
- Calculator: >99.9% success
- Measurement: Tool execution logs with error tracking

**Memory Efficiency:**
- Metric: Percentage of conversations completing within 10-message buffer
- Target: >90%
- If exceeded: Consider increasing buffer or implementing summarization

**4. Business Impact Metrics**

**Time Saved:**
- Calculation: (Meetings scheduled × 20 min saved per meeting) = Total time saved
- Example: 100 meetings/month × 20 min = 2,000 min = 33 hours saved
- Value: 33 hours × $50/hour = $1,650/month productivity gain

**Adoption Rate:**
- Metric: Percentage of team using agent vs. manual scheduling
- Target: >70% adoption within 3 months
- Measurement: Unique users per week, meetings scheduled via agent vs. manual

**Error Cost:**
- Metric: Number of double-bookings or scheduling errors
- Target: <1% error rate
- Impact: Each error costs meeting rescheduling + participant frustration
- Measurement: User-reported issues, calendar conflict detection

**5. Evaluation Methods**

**A. Automated Testing Suite**
```
Daily regression tests:
- 50 standard scheduling scenarios
- 10 edge cases (no availability, ambiguous dates, multiple recipients)
- 5 failure scenarios (tool timeout, SMTP failure, memory overflow)
- Assertion: Expected output matches actual output
```

**B. A/B Testing**
```
Experiment: System prompt variations
- Control (A): Current prompt with 3 time slot suggestions
- Variant (B): Modified prompt with 5 time slot suggestions
- Metrics: Task completion rate, user satisfaction
- Duration: 2 weeks
- Analysis: Statistical significance testing (p<0.05)
```

**C. Production Monitoring Dashboard**
```
Real-time metrics:
- Requests per hour
- Average response time
- Tool success rates
- Active conversations
- Error rate trend

Alerts:
- Response time >10 seconds
- Error rate >2%
- SMTP failures >5 in 1 hour
```

**D. User Feedback Collection**
```
Post-interaction survey:
1. "Did the agent successfully schedule your meeting?" (Yes/No)
2. "How would you rate the experience?" (1-5 stars)
3. "What could be improved?" (Free text)

Analysis: Weekly review of feedback, categorize improvement requests
```

**6. Success Criteria for This Workflow**

Based on current capabilities:

| Metric | Target | Measurement Method |
|--------|--------|-------------------|
| Scheduling Accuracy | 100% | Calendar conflict detection |
| Email Delivery Rate | >99% | SMTP logs |
| Response Time (P95) | <5s | Timestamp analysis |
| Task Completion Rate | >75% | Conversation tracking |
| User Satisfaction | >4.0/5 | Post-interaction survey |
| Context Retention | >85% | Multi-turn test cases |

**7. Continuous Improvement Process**

```
Weekly:
- Review error logs
- Analyze failed conversations
- Identify top 3 user pain points

Monthly:
- A/B test system prompt variations
- Review user feedback themes
- Update tool logic based on edge cases

Quarterly:
- Comprehensive performance review
- Feature prioritization based on data
- Infrastructure optimization (if response time degrading)
```

**Production-Ready Evaluation:**
The key is combining quantitative metrics (response time, accuracy) with qualitative feedback (user satisfaction) and continuously iterating based on data. The workflow is production-ready when it meets all success criteria consistently over 30-day period.

