# 🎫 Support Ticket Classifier

An AI-powered customer support ticket classifier built with **LangChain**, **OpenAI**, and **Streamlit**.  
Automatically classifies tickets by category, priority, and sentiment — with built-in PII redaction and prompt injection protection.

---

## 📌 Table of Contents

- [Overview](#overview)
- [Pipeline Architecture](#pipeline-architecture)
- [Features](#features)
- [Project Structure](#project-structure)
- [Installation](#installation)
- [How to Run](#how-to-run)
- [How to Use](#how-to-use)
- [Sample Tickets](#sample-tickets)
- [Output Fields](#output-fields)
- [Security](#security)
- [Cost Tracking](#cost-tracking)
- [Tech Stack](#tech-stack)
- [Known Bugs Fixed](#known-bugs-fixed)
- [Future Improvements](#future-improvements)
- [Author](#author)

---

## Overview

This project is a **non-agentic, sequential AI pipeline** that processes customer support tickets through 5 steps:

1. Removes sensitive personal data (PII)
2. Checks for prompt injection attacks
3. Classifies the ticket using an LLM
4. Validates the LLM output as proper JSON
5. Calculates the API token cost

Built as a **portfolio project** to demonstrate real-world GenAI engineering skills including LangChain pipelines, LLM Guard patterns, structured output validation, and Streamlit deployment.

---

## Pipeline Architecture

```
Customer Ticket (raw text)
         │
         ▼
┌─────────────────────┐
│   1. PII Redaction  │  ← Removes emails, phones, credit cards, order IDs
│     (Regex based)   │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│   2. LLM Guard      │  ← Detects prompt injection attacks
│  (Regex + LLM dual  │     Layer 1: Fast regex check (free)
│      check)         │     Layer 2: LLM security check (if regex passes)
└────────┬────────────┘
         │
    Injection? ──── YES ──→ BLOCKED (return immediately)
         │
        NO
         │
         ▼
┌─────────────────────┐
│  3. Classification  │  ← LLM classifies category, priority, sentiment
│      (LLM)          │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  4. JSON Validation │  ← Validates LLM output structure and values
│    + Fallback       │     If fails → Fallback prompt → validate again
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  5. Cost Calculator │  ← Counts tokens, calculates USD cost
└────────┬────────────┘
         │
         ▼
    TicketResult object
    (all fields filled)
```

---

## Features

- **PII Redaction** — automatically detects and removes:
  - Email addresses
  - Phone numbers
  - Credit card numbers
  - Order IDs
  - ZIP codes
  - IP addresses

- **LLM Guard** — two-layer injection protection:
  - Layer 1: Fast regex pattern matching (10 known attack patterns)
  - Layer 2: LLM-based semantic detection for subtle attacks

- **Ticket Classification** — LLM returns structured JSON with:
  - Category, Priority, Sentiment, Summary, Suggested Action, Confidence

- **JSON Validation** — strict schema check with automatic fallback

- **Cost Tracking** — tracks input/output tokens and estimates USD cost per ticket

- **Streamlit UI** — clean web interface with sample tickets, sidebar settings, and live results

---

## Project Structure

```
Support-Ticket-Classifier/
│
├── classifier.py        # Core pipeline (PII, Guard, Classify, Validate, Cost)
├── app.py               # Streamlit web interface
├── requirements.txt     # Python dependencies
├── .gitignore           # Files excluded from Git
└── README.md            # This file
```

---

## Installation

### Step 1 — Clone the repository

```bash
git clone https://github.com/your-username/support-ticket-classifier.git
cd support-ticket-classifier
```

### Step 2 — Create a virtual environment (recommended)

```bash
# Windows
python -m venv venv
venv\Scripts\activate

# Mac / Linux
python -m venv venv
source venv/bin/activate
```

### Step 3 — Install dependencies

```bash
pip install -r requirements.txt
```

### Step 4 — Set your OpenAI API key

```bash
# Windows (Command Prompt)
set OPENAI_API_KEY=sk-your-key-here

# Windows (PowerShell)
$env:OPENAI_API_KEY="sk-your-key-here"

# Mac / Linux
export OPENAI_API_KEY="sk-your-key-here"
```

> Get your API key from: https://platform.openai.com/api-keys

---

## How to Run

### Streamlit Web App (recommended)

```bash
streamlit run app.py
```

Browser opens automatically at: `http://localhost:8501`

### Command Line (optional — if you have main.py)

```bash
# Run default sample ticket
python main.py

# Run your own ticket
python main.py --ticket "My order hasn't arrived after 10 days"

# Run all 5 sample tickets
python main.py --demo

# Output as JSON
python main.py --demo --json
```

---

## How to Use

### Using the Streamlit App

```
1. Open the app in your browser (http://localhost:8501)
2. Enter your OpenAI API key in the sidebar
3. Choose a model (gpt-4o-mini recommended — cheapest)
4. Pick a sample ticket from the dropdown OR type your own
5. Click "Classify Ticket"
6. View results: category, priority, sentiment, cost, PII found
```

### Using the Python Class directly

```python
from classifier import SupportTicketClassifier

# Create classifier
clf = SupportTicketClassifier(
    openai_api_key="sk-your-key-here",
    model="gpt-4o-mini"
)

# Run pipeline
result = clf.run("My package hasn't arrived after 10 days. Order #98765.")

# Access results
print(result.category)          # DELIVERY_ISSUE
print(result.priority)          # HIGH
print(result.sentiment)         # FRUSTRATED
print(result.confidence)        # 0.94
print(result.summary)           # Package is 10 days overdue
print(result.suggested_action)  # Contact courier and escalate
print(result.pii_found)         # ['ORDER_ID']
print(result.estimated_cost_usd) # 0.000175
```

---

## Sample Tickets

The app includes 5 built-in sample tickets for testing:

| Sample | Tests |
|--------|-------|
| Angry Delivery Issue | Basic classification, FRUSTRATED sentiment |
| Payment Problem with PII | PII redaction (credit card, email, phone) |
| Prompt Injection Attack | LLM Guard blocking |
| Technical Support | ACCOUNT_ACCESS category |
| Product Defect | PRODUCT_DEFECT category, replacement request |

---

## Output Fields

Every ticket returns a `TicketResult` object with these fields:

| Field | Type | Example |
|-------|------|---------|
| `original_text` | str | Raw ticket text |
| `redacted_text` | str | Text with PII removed |
| `pii_found` | list | `["EMAIL", "PHONE"]` |
| `injection_detected` | bool | `False` |
| `injection_reason` | str | `""` or reason string |
| `category` | str | `DELIVERY_ISSUE` |
| `priority` | str | `HIGH` |
| `sentiment` | str | `ANGRY` |
| `summary` | str | One sentence summary |
| `suggested_action` | str | Recommended agent action |
| `confidence` | float | `0.94` |
| `input_tokens` | int | `770` |
| `output_tokens` | int | `95` |
| `estimated_cost_usd` | float | `0.000175` |
| `fallback_used` | bool | `False` |
| `error` | str or None | `None` |

### Valid Category Values

```
DELIVERY_ISSUE    PAYMENT_PROBLEM    PRODUCT_DEFECT    ACCOUNT_ACCESS
REFUND_REQUEST    GENERAL_INQUIRY    TECHNICAL_SUPPORT    COMPLAINT
```

### Valid Priority Values

```
CRITICAL    HIGH    MEDIUM    LOW
```

### Valid Sentiment Values

```
ANGRY    FRUSTRATED    NEUTRAL    SATISFIED    CONFUSED
```

---

## Security

### PII Redaction Patterns

| Type | Example Input | After Redaction |
|------|--------------|-----------------|
| EMAIL | `john@gmail.com` | `[EMAIL_REDACTED]` |
| PHONE | `9876543210` | `[PHONE_REDACTED]` |
| CREDIT_CARD | `4111-1111-1111-1111` | `[CREDIT_CARD_REDACTED]` |
| ORDER_ID | `Order #98765` | `[ORDER_ID_REDACTED]` |
| ZIP_CODE | `560001` | `[ZIP_CODE_REDACTED]` |
| IP_ADDRESS | `192.168.1.1` | `[IP_ADDRESS_REDACTED]` |

### LLM Guard — Injection Patterns Detected

```
"ignore previous instructions"
"forget everything"
"you are now a"
"act as DAN / jailbreak"
"system prompt / system instruction"
<system> tags
[INST] tokens
"override safety / guidelines"
"disregard your training"
"pretend you are"
```

### Fail-Safe Design

- If LLM Guard crashes → defaults to `False` (allow ticket through)
- If JSON validation fails twice → returns safe default values
- API key is never stored in code — always read from environment variable

---

## Cost Tracking

Pricing based on **gpt-4o-mini** (as of mid-2025):

| Token Type | Price |
|------------|-------|
| Input | $0.150 per 1M tokens |
| Output | $0.600 per 1M tokens |

### Typical cost per ticket

| Scenario | Approx. Cost |
|----------|-------------|
| Normal ticket | ~$0.000150 |
| Ticket with fallback | ~$0.000250 |
| Blocked injection | ~$0.000050 |
| 1000 tickets/day | ~$0.15/day |
| 1000 tickets/day for a month | ~$4.50/month |

---

## Tech Stack

| Tool | Version | Purpose |
|------|---------|---------|
| Python | 3.10+ | Core language |
| LangChain | 0.3.0+ | LLM pipeline framework |
| LangChain-OpenAI | 0.2.0+ | OpenAI integration |
| OpenAI | 1.40.0+ | GPT models |
| Tiktoken | 0.7.0+ | Token counting |
| Streamlit | 1.35.0+ | Web UI |

---

## Known Bugs Fixed

The original `classifier.py` had 8 bugs — all fixed in this version:

| # | Bug | Fix |
|---|-----|-----|
| 1 | `TicketResult` dataclass missing fields | Added all 16 fields |
| 2 | EMAIL regex `[A-Za-Z0-9]` wrong capital | Fixed to `[A-Za-z0-9]` |
| 3 | `StrOutputParser` missing `()` | Fixed to `StrOutputParser()` |
| 4 | Guard methods called without underscore | Fixed to `_regex_check` / `_llm_check` |
| 5 | `json.dumps()` used instead of `json.loads()` | Fixed — dumps=TO string, loads=FROM string |
| 6 | `is not` used instead of `not in` | Fixed — `is not` checks identity, `not in` checks membership |
| 7 | `return True, {}, ""` returned empty dict | Fixed to `return True, data, ""` |
| 8 | `self.redactor = PIIRedactor` missing `()` | Fixed to `PIIRedactor()` — must be instance not class |

---

## Future Improvements

- [ ] Add database to store all classified tickets
- [ ] Add ticket history page in Streamlit
- [ ] Support multiple languages (Tamil, Hindi, etc.)
- [ ] Add email/webhook notification for CRITICAL tickets
- [ ] Export results to CSV or Excel
- [ ] Add authentication to the Streamlit app
- [ ] Deploy to Streamlit Cloud or AWS
- [ ] Add unit tests for each pipeline step
- [ ] Support file upload (classify tickets from CSV)
- [ ] Dashboard with charts — tickets by category, priority over time

---

## Author

**Nataraj S B**  
Career switcher from Garments Industry → ML / AI Engineer  
📍 Tamil Nadu, India  
🔗 GitHub: [github.com/connectnataraj-boop](https://github.com/connectnataraj-boop)  
🔗 LinkedIn: [linkedin.com/in/nataraj-sb-b5a84a3b7](https://linkedin.com/in/nataraj-sb-b5a84a3b7)

---

> Built as part of an active ML/GenAI portfolio.  
> Other projects: RAG HR Chatbot · CNN Dog vs Cat Classifier · Twitter Sentiment Analysis (BERT)
