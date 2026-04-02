## ProjectPlan: MailPilot

### Project goal

Build a Python-based email reply agent that:

- reads unread emails from an inbox through IMAP
- extracts the useful message context
- generates a draft reply using OpenAI
- sends the reply through SMTP
- keeps the codebase modular, testable, and safe enough for an MVP

This document is written as an implementation guide, not just an idea list. Each phase explains:

- what we are building
- why that phase matters
- what to install
- which files to create
- what each file is responsible for
- how to verify the phase before moving on

---

## 1. Development strategy

### Recommended approach

Build this project in small commits and keep the first version simple:

1. scaffold the repo
2. load configuration correctly
3. connect to email safely
4. generate replies
5. orchestrate the full workflow
6. add tests and polish

### MVP definition

The first working version of MailPilot should be able to:

- read unread emails from one inbox
- generate one reply per message
- send the reply back in the same thread
- support a `--dry-run` mode
- log what happened clearly

### Non-goals for the first version

These are useful later, but should not block the MVP:

- web dashboard
- multi-account support
- advanced thread memory
- human approval UI
- full spam detection
- OAuth-based provider integrations

---

## 2. Recommended final structure

```text
MailPilot/
├── main.py
├── requirements.txt
├── .env.example
├── .gitignore
├── README.md
├── config/
│   ├── __init__.py
│   └── settings.py
├── email_service/
│   ├── __init__.py
│   ├── imap_client.py
│   ├── smtp_client.py
│   └── parser.py
├── llm/
│   ├── __init__.py
│   ├── openai_handler.py
│   └── prompts.py
├── agent/
│   ├── __init__.py
│   └── reply_agent.py
├── utils/
│   ├── __init__.py
│   ├── logger.py
│   └── helpers.py
└── tests/
    ├── __init__.py
    ├── test_settings.py
    ├── test_imap_smtp.py
    ├── test_parser.py
    └── test_agent.py
```

### Why this structure works

- `config/` keeps environment and runtime configuration in one place.
- `email_service/` isolates mailbox logic from business logic.
- `llm/` isolates prompt and model behavior.
- `agent/` holds the orchestration logic for processing one email or many.
- `utils/` keeps logging and small shared helpers reusable.
- `tests/` mirrors the main modules and makes the project easier to extend.

---

## 3. Dependency plan

### Core dependencies for the MVP

Add these to `requirements.txt`:

- `openai`
- `python-dotenv`
- `pytest`

### Recommended quality dependencies

Add these when you begin testing and cleanup:

- `pytest-mock`
- `ruff`
- `black`

### Optional dependencies

Only add these if you actually use them:

- `APScheduler` for scheduled execution
- `fastapi` and `uvicorn` for a later dashboard or review API

### Dependency note

The current draft mentions `langchain`, but for a first version it is probably unnecessary. A direct `openai` integration is simpler, easier to test, and reduces moving parts. You can always add LangChain later if the agent becomes more complex.

---

## 4. Setup checklist

Before coding the main features:

1. Install Python `3.11+`
2. Create a virtual environment
3. Activate the virtual environment
4. Install dependencies from `requirements.txt`
5. Copy `.env.example` to `.env`
6. Fill in OpenAI and mail credentials
7. Run the app once in `--dry-run` mode after the scaffold exists

### Suggested commands

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
```

---

## 5. Phase-by-phase implementation plan

## Phase 1: Repository scaffold

### Goal

Create the initial project structure so later phases have a clean place to live.

### Why this phase matters

Without a clean scaffold, configuration, mail handling, LLM logic, and tests quickly get mixed together. This phase makes the rest of the project easier to build and debug.

### Install in this phase

- nothing beyond Python itself if you are only creating files

### Files to create

#### Root files

- `main.py`
- `requirements.txt`
- `.env.example`
- `.gitignore`
- `README.md`

#### Package files

- `config/__init__.py`
- `config/settings.py`
- `email_service/__init__.py`
- `email_service/imap_client.py`
- `email_service/smtp_client.py`
- `email_service/parser.py`
- `llm/__init__.py`
- `llm/openai_handler.py`
- `llm/prompts.py`
- `agent/__init__.py`
- `agent/reply_agent.py`
- `utils/__init__.py`
- `utils/logger.py`
- `utils/helpers.py`
- `tests/__init__.py`
- `tests/test_settings.py`
- `tests/test_imap_smtp.py`
- `tests/test_parser.py`
- `tests/test_agent.py`

### What each file should eventually contain

- `main.py`: CLI entry point and one-cycle orchestration
- `requirements.txt`: project dependencies
- `.env.example`: template of all required environment variables
- `.gitignore`: ignore `.venv`, `.env`, caches, and test artifacts
- `config/settings.py`: environment parsing and validation
- `email_service/imap_client.py`: mailbox reading logic
- `email_service/smtp_client.py`: email sending logic
- `email_service/parser.py`: email parsing and cleanup helpers
- `llm/openai_handler.py`: OpenAI client wrapper
- `llm/prompts.py`: prompt templates and prompt-building helpers
- `agent/reply_agent.py`: main processing flow
- `utils/logger.py`: logging setup
- `utils/helpers.py`: small shared utilities
- `tests/*`: unit tests for each major module

### Deliverables

- project folders exist
- import paths are valid
- empty modules are in place
- `README.md` and `ProjectPlan.md` describe the intended system

### Exit criteria

- the repo structure exists
- Python can import the packages without path errors

---

## Phase 2: Configuration and environment loading

### Goal

Load all runtime settings from `.env` safely and validate them early.

### Why this phase matters

Email agents fail in confusing ways when credentials or ports are wrong. A clean settings layer makes the rest of the app predictable and easier to debug.

### Install in this phase

Add to `requirements.txt`:

- `python-dotenv`

### Files to create or implement

- `config/settings.py`
- `.env.example`
- `tests/test_settings.py`

### Responsibilities

#### `config/settings.py`

Implement a settings model that:

- loads values from environment variables
- converts ports to integers
- provides defaults where reasonable
- validates required values
- exposes a single `load_settings()` function

Suggested fields:

- `OPENAI_API_KEY`
- `OPENAI_MODEL`
- `IMAP_HOST`
- `IMAP_PORT`
- `IMAP_USER`
- `IMAP_PASSWORD` or `IMAP_APP_PASSWORD`
- `SMTP_HOST`
- `SMTP_PORT`
- `SMTP_USER`
- `SMTP_PASSWORD` or `SMTP_APP_PASSWORD`
- `AUTO_REPLY_SUBJECT_PREFIX`
- `LOG_LEVEL`
- `DRY_RUN_DEFAULT`

#### `.env.example`

Document every supported variable with safe example values.

#### `tests/test_settings.py`

Test:

- missing required variables
- valid parsing of integers and booleans
- default values

### Example `.env.example` shape

```env
OPENAI_API_KEY=
OPENAI_MODEL=gpt-4.1-mini

IMAP_HOST=imap.gmail.com
IMAP_PORT=993
IMAP_USER=your.email@example.com
IMAP_APP_PASSWORD=

SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your.email@example.com
SMTP_APP_PASSWORD=

AUTO_REPLY_SUBJECT_PREFIX=Re:
LOG_LEVEL=INFO
DRY_RUN_DEFAULT=true
```

### Deliverables

- one place to load all settings
- one place to document all env vars
- tests for config behavior

### Exit criteria

- invalid config fails fast with clear error messages
- valid config loads consistently

---

## Phase 3: Logging and shared utilities

### Goal

Create lightweight utilities used by every other module.

### Why this phase matters

When the agent fetches mail, calls the LLM, and sends replies, you need consistent logging to understand what happened in each step.

### Install in this phase

- no extra package required if standard library logging is enough

### Files to create or implement

- `utils/logger.py`
- `utils/helpers.py`

### Responsibilities

#### `utils/logger.py`

Set up:

- log formatting
- log level control from settings
- a reusable `get_logger(__name__)` style helper

#### `utils/helpers.py`

Keep small shared helpers here, such as:

- trimming text
- normalizing subject lines
- safe fallback string cleanup

### Deliverables

- app-wide logging utility
- small helper functions that prevent repeated code

### Exit criteria

- every later module can log through one consistent interface

---

## Phase 4: Email reading with IMAP

### Goal

Connect to the mailbox, fetch unread emails, and convert them into clean internal data objects.

### Why this phase matters

If email parsing is messy, the LLM receives poor context and generates poor replies. This phase creates the raw inputs for the whole agent.

### Install in this phase

- no external package required if using `imaplib` and `email` from the standard library

### Files to create or implement

- `email_service/imap_client.py`
- `email_service/parser.py`
- `tests/test_parser.py`
- `tests/test_imap_smtp.py`

### Responsibilities

#### `email_service/imap_client.py`

Implement methods such as:

- connect to IMAP using SSL
- select inbox
- search unread messages
- fetch raw message content
- optionally mark messages as seen after successful processing

#### `email_service/parser.py`

Implement helpers that:

- parse sender name and address
- extract subject
- extract plain text body
- gracefully handle multipart emails
- keep message ID and thread headers
- strip signatures or excessive quoted text if possible

### Suggested internal email model

The parsed email object should include:

- `message_id`
- `from_name`
- `from_email`
- `subject`
- `body_text`
- `references`
- `in_reply_to`

### Deliverables

- IMAP connection logic
- reliable parsing for common email shapes
- tests using mocked IMAP interactions

### Exit criteria

- the app can fetch unread messages from a test inbox
- parsed content is usable for reply generation

---

## Phase 5: Email sending with SMTP

### Goal

Send generated replies back to the sender while preserving thread context.

### Why this phase matters

Reading mail is only half the system. This phase makes the project actually useful by turning drafts into sent replies.

### Install in this phase

- no external package required if using `smtplib` and `email.message`

### Files to create or implement

- `email_service/smtp_client.py`
- `tests/test_imap_smtp.py`

### Responsibilities

#### `email_service/smtp_client.py`

Implement methods to:

- connect to SMTP with TLS
- authenticate
- build reply messages
- set `In-Reply-To` and `References` headers
- send text replies
- support dry-run behavior

### Deliverables

- SMTP sending flow
- reply message construction
- tests for header generation and send behavior

### Exit criteria

- a reply can be sent to a controlled test account
- thread headers are included correctly

---

## Phase 6: OpenAI integration

### Goal

Turn email context into a clean reply draft using the OpenAI API.

### Why this phase matters

This is the intelligence layer of MailPilot. The output quality here determines whether the reply feels helpful, safe, and relevant.

### Install in this phase

Add to `requirements.txt`:

- `openai`

### Files to create or implement

- `llm/openai_handler.py`
- `llm/prompts.py`
- `tests/test_agent.py`

### Responsibilities

#### `llm/prompts.py`

Define prompt-building helpers for:

- a system instruction describing the assistant behavior
- an email reply prompt template
- tone and safety guidance

#### `llm/openai_handler.py`

Implement:

- OpenAI client initialization
- a `generate_reply()` function
- basic retry or graceful failure handling
- response normalization

### Prompt design goals

The prompt should tell the model to:

- reply politely and clearly
- stay grounded in the incoming email
- avoid inventing facts
- avoid making commitments the user did not authorize
- keep responses concise unless the email requires detail

### Deliverables

- prompt helper module
- model wrapper
- tests for prompt construction or mocked generation flow

### Exit criteria

- given sample email input, the module returns a usable draft reply

---

## Phase 7: Agent orchestration

### Goal

Create the core business logic that connects reading, generating, and sending.

### Why this phase matters

This phase turns separate modules into an actual agent.

### Files to create or implement

- `agent/reply_agent.py`
- `tests/test_agent.py`

### Responsibilities

#### `agent/reply_agent.py`

Implement functions such as:

- `process_single_email(email_obj)`
- `process_unread_emails()`
- reply safety checks
- dry-run logic
- result summaries for logs

### Suggested workflow

For each unread email:

1. parse the email
2. check if it should be skipped
3. generate a reply draft
4. send the reply or log it in dry-run mode
5. record success or failure

### Simple safety rules for MVP

Skip or flag emails when:

- the body is empty
- the sender is the same as the mailbox user
- the email appears auto-generated
- the message looks like spam or no-reply mail

### Deliverables

- one service that coordinates all modules
- clear success and failure paths

### Exit criteria

- the agent can process a small batch of unread emails from end to end

---

## Phase 8: CLI entry point and execution modes

### Goal

Make the project easy to run from the command line.

### Why this phase matters

A good CLI lets you test safely, especially with `--dry-run`, and makes the app ready for cron or deployment later.

### Files to create or implement

- `main.py`
- `README.md`

### Responsibilities

#### `main.py`

Implement:

- settings loading
- logger initialization
- service construction
- CLI flags
- one execution cycle

### Recommended CLI flags

- `--dry-run`
- `--once`
- `--max-emails`
- `--log-level`

### Keep v1 simple

Prefer a single-run command first. A long-running daemon can be added later after the one-cycle flow is stable.

### Deliverables

- runnable entry point
- documented commands in `README.md`

### Exit criteria

- `python main.py --dry-run` completes without unhandled exceptions

---

## Phase 9: Testing and code quality

### Goal

Add enough tests and formatting tools to trust future changes.

### Why this phase matters

Mail automation touches real inboxes. Even a simple test suite helps prevent accidental bad behavior.

### Install in this phase

Add to `requirements.txt`:

- `pytest`
- `pytest-mock`
- `ruff`
- `black`

### Files to create or implement

- `tests/test_settings.py`
- `tests/test_parser.py`
- `tests/test_imap_smtp.py`
- `tests/test_agent.py`

Optional later:

- `.pre-commit-config.yaml`

### What to test

- settings parsing
- subject normalization
- parser behavior for multipart emails
- skipped email logic
- dry-run behavior
- successful send behavior with mocks
- OpenAI failure handling

### Validation commands

```bash
pytest
ruff check .
black --check .
```

### Deliverables

- repeatable tests
- linting and formatting guardrails

### Exit criteria

- tests pass
- basic style checks pass

---

## Phase 10: Post-MVP improvements

### Goal

Extend the project only after the MVP is stable.

### Good next additions

- scheduler support with `APScheduler`
- approval workflow before sending
- dashboard with `FastAPI`
- better thread memory
- sender allowlist and blocklist
- spam and sensitive-content filters
- reply persona settings
- provider-specific adapters for Gmail or Outlook

### Important note

Do not begin these until the single-run MVP is tested and safe in dry-run mode.

---

## 6. File creation order

Use this order to avoid jumping around too much:

1. `.gitignore`
2. `requirements.txt`
3. `.env.example`
4. `config/__init__.py`
5. `config/settings.py`
6. `utils/__init__.py`
7. `utils/logger.py`
8. `utils/helpers.py`
9. `email_service/__init__.py`
10. `email_service/parser.py`
11. `email_service/imap_client.py`
12. `email_service/smtp_client.py`
13. `llm/__init__.py`
14. `llm/prompts.py`
15. `llm/openai_handler.py`
16. `agent/__init__.py`
17. `agent/reply_agent.py`
18. `main.py`
19. `tests/__init__.py`
20. `tests/test_settings.py`
21. `tests/test_parser.py`
22. `tests/test_imap_smtp.py`
23. `tests/test_agent.py`
24. `README.md`

---

## 7. Recommended contents for `requirements.txt`

### Minimal MVP version

```txt
openai
python-dotenv
pytest
```

### Better developer experience version

```txt
openai
python-dotenv
pytest
pytest-mock
ruff
black
```

### Avoid over-installing early

Only add more dependencies when a real need appears. That keeps setup easier and the codebase simpler.

---

## 8. Verification checklist by milestone

### After Phase 1

- folders exist
- imports do not fail

### After Phase 2

- settings load from `.env`
- missing values produce readable errors

### After Phase 4

- unread emails can be fetched and parsed

### After Phase 5

- test reply can be sent successfully

### After Phase 6

- OpenAI returns a draft reply for sample input

### After Phase 8

- `python main.py --dry-run` works end to end

### After Phase 9

- tests pass locally

---

## 9. Suggested commit plan

To keep the project manageable, use one commit per phase where possible:

1. scaffold project structure
2. add settings and env loading
3. add logging helpers
4. add IMAP reading and parsing
5. add SMTP sending
6. add OpenAI reply generation
7. add agent workflow
8. add CLI entry point
9. add tests and code quality tools
10. add optional enhancements

---

## 10. Final recommendation

If you want the smoothest build path, start with this exact sequence:

1. `requirements.txt`
2. `.env.example`
3. `config/settings.py`
4. `utils/logger.py`
5. `email_service/parser.py`
6. `email_service/imap_client.py`
7. `email_service/smtp_client.py`
8. `llm/prompts.py`
9. `llm/openai_handler.py`
10. `agent/reply_agent.py`
11. `main.py`
12. tests

This keeps the project moving from foundation to integration without overcomplicating the first version.
