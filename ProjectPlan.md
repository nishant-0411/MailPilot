## ProjectPlan: Automatic Email Reply Mail Agent (MailPilot)

### TL;DR
Build a Python-based mail agent that reads unread emails, generates automated replies through OpenAI, and sends responses via SMTP. Keep architecture modular with clear phases, and include instructions for environment setup, dependencies, config, tests, and deploy.

---

## Phase 1: Kickoff + scaffold

1. Create base repository layout.
   - `main.py`
   - `requirements.txt`
   - `README.md` (update with feature summary)
   - `.gitignore`
   - `.env.example`
2. Create folders and empty `__init__.py` files:
   - `email_service/`
   - `agent/`
   - `llm/`
   - `config/`
   - `utils/`
   - `tests/`
3. Add initial dependencies to `requirements.txt`:
   - `python-dotenv`
   - `openai`
   - `langchain`
   - `pytest`
   - `pytest-mock` (optional)
   - `pytz` (recommended)
4. Document commands in README:
   - `python -m pip install -r requirements.txt`
   - `cp .env.example .env`
   - `python main.py`

---

## Phase 2: Configuration + environment setup

1. Add `.env.example` entries:
   - `OPENAI_API_KEY`
   - `IMAP_HOST`, `IMAP_PORT`, `IMAP_USER`, `IMAP_PASSWORD`/`IMAP_APP_PASSWORD` 
   - `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASSWORD`/`SMTP_APP_PASSWORD`
   - Optional: `AUTO_REPLY_SUBJECT_PREFIX`, `AUTO_REPLY_TEMPLATE`, `LOG_LEVEL`
2. Implement `config/settings.py`:
   - Parse `.env` using `python-dotenv` and `os.getenv`
   - Provide typed config object (dataclass or simple namespace)
3. Add logging utility in `utils/logger.py`.

---

## Phase 3: Email integration (IMAP + SMTP)

1. `email_service/imap_client.py`:
   - Connect to IMAP server with TLS
   - Fetch unseen messages from INBOX
   - Parse sender, subject, body and message ID
   - Mark as read (or keep unread until sent?)
2. `email_service/smtp_client.py`:
   - Connect to SMTP server with TLS
   - Send text/html message replies
   - Include In-Reply-To / References headers
3. Add unit tests to `tests/test_imap_smtp.py` using mocks.

---

## Phase 4: LLM + agent logic

1. `llm/openai_handler.py`:
   - Initialize OpenAI client from config
   - Provide `generate_reply(prompt, context)` with basic prompt template
   - Rate limit / error retry logic
2. `agent/reply_agent.py`:
   - Function `compose_reply(email_context)`:
      - include sender greeting
      - include original email thread snippet
      - injection protection (strip malicious characters)
   - Function `process_unread_emails()`:
      - fetch emails (IMAP client)
      - generate replies (LLM handler)
      - send replies (SMTP client)
      - update status
2. Add unit tests in `tests/test_agent.py`.

---

## Phase 5: Main workflow + orchestration

1. `main.py`:
   - load config, logger
   - instantiate IMAP/SMTP clients, LLM handler, agent
   - run one cycle `process_unread_emails()`
   - optionally expose schedule or loop with sleep (first pass: single-run)
2. Add flags/options:
   - `--dry-run` (no send, log only)
   - `--once` / `--daemon`
   - `--log-level`
3. Document usage and expected behavior in README.

---

## Phase 6: Testing + quality

1. Pytest tests:
   - integration test with mocked email data
   - unit tests for prompt creation
   - error path tests (IMAP timeout, OpenAI exception)
2. Add pre-commit config (optional): `ruff`, `black`, `isort`.
3. Validate with `python -m pytest`

---

## Phase 7: Optional enhancements (post-MVP)

1. Automatic scheduler (CRON / `APScheduler`).
2. Web dashboard (FastAPI/Flask) and feedback loop (approve/remove individual replies).
3. Thread tracking and deduping.
4. Safety rules: match keywords and skip if potential spam or sensitive content.
5. Retry and backoff for mail provider errors.

---

## File creation order (recommended)

1. `.env.example`, `requirements.txt`, `.gitignore`, `README.md`
2. `config/settings.py`, `utils/logger.py`
3. `email_service/imap_client.py`, `email_service/smtp_client.py`
4. `llm/openai_handler.py`, `agent/reply_agent.py`
5. `main.py`
6. `tests/test_*` files

---

## Installation checklist

- Python 3.11+ installed
- Create and activate virtual env
- `pip install -r requirements.txt`
- Copy `.env.example` to `.env` and fill credentials
- Confirm IMAP/SMTP credentials and network allowlists
- Optional: install pre-commit hooks

---

## Verification checklist

1. `python main.py` with `--dry-run` returns no unhandled errors.
2. `pytest` passes all tests.
3. Evaluate reply quality with one sample email.
4. Log output checks for fetched/sent counts and errors.

---

## Notes

- no code produced yet; this is a phase plan for implementation.
- keep the first version simple and make each phase a commit.

MailPilot/
├── main.py
├── requirements.txt
├── README.md
├── .gitignore
├── .env.example
├── email_service/
│       ├── init.py
|       ├── imap_client.py
│       └── smtp_client.py
├── agent/
│       ├── init.py
│       └── reply_agent.py
├── llm
│       ├── init.py
│       └── openai_handler.py
├── config/
│       ├── init.py
│       └── settings.py
├── utils/
│       ├── init.py
│       └── logger.py
└── tests/
        ├── init.py
        ├── test_imap_smtp.py
        └── test_agent.py