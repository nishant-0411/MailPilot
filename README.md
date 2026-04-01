# MailPilot ✉️🤖

**MailPilot is an AI-powered automatic email reply agent.** It intelligently reads your incoming emails and generates context-aware, appropriate replies using Large Language Models (LLMs).

> **Note:** This project is currently in early development. Follow the project phases below for implementation status.

## ✨ Features

- **Automated Email Monitoring:** Connects to your inbox via IMAP and fetches new, unread messages
- **Smart Context Extraction:** Understands the intent, sentiment, and context of incoming emails
- **AI-Generated Replies:** Uses OpenAI/LangChain to draft intelligent, relevant, and polite responses automatically
- **Secure Email Sending:** Sends replies via SMTP with proper thread tracking (In-Reply-To/References headers)
- **Modular Architecture:** Clean separation of concerns with dedicated modules for email, LLM, and agent logic
- **Configuration Management:** Environment-based configuration with type safety
- **Comprehensive Testing:** Unit tests with mocks for all major components

## 🛠️ Tech Stack

- **Language:** Python 3.11+
- **AI/LLM:** OpenAI API with LangChain integration
- **Email Protocol:** IMAP for reading emails, SMTP for sending replies
- **Core Libraries:** 
  - `python-dotenv` - Environment configuration
  - `openai` - OpenAI API client
  - `langchain` - LLM orchestration
  - `pytest` - Testing framework
  - `pytz` - Timezone handling
- **Built-in:** `imaplib`, `smtplib`, `email` for email operations

## 🚀 Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/your-username/MailPilot.git
   cd MailPilot
   ```

2. **Create a virtual environment (recommended):**
   ```bash
   python3 -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

3. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

4. **Set up Environment Variables:**
   ```bash
   cp .env.example .env
   ```
   Edit `.env` and configure your credentials:
   ```env
   # OpenAI Configuration
   OPENAI_API_KEY=your_openai_api_key_here
   
   # IMAP Configuration (Email Reading)
   IMAP_HOST=imap.gmail.com
   IMAP_PORT=993
   IMAP_USER=your.email@example.com
   IMAP_APP_PASSWORD=your_app_password
   
   # SMTP Configuration (Email Sending)
   SMTP_HOST=smtp.gmail.com
   SMTP_PORT=587
   SMTP_USER=your.email@example.com
   SMTP_APP_PASSWORD=your_app_password
   
   # Optional Settings
   AUTO_REPLY_SUBJECT_PREFIX=Re:
   LOG_LEVEL=INFO
   ```

## 💻 Usage

### Basic Usage
Run the agent for a single cycle:
```bash
python main.py
```

### Advanced Options
```bash
# Dry run (fetch and generate replies, but don't send)
python main.py --dry-run

# Run once and exit (default behavior)
python main.py --once

# Run as daemon with continuous monitoring
python main.py --daemon

# Set log level
python main.py --log-level DEBUG
```

## 🏗️ Project Structure

```
MailPilot/
├── main.py                 # Entry point and orchestration
├── requirements.txt        # Python dependencies
├── README.md              # This file
├── .gitignore             # Git ignore patterns
├── .env.example           # Environment variables template
├── email_service/         # Email handling modules
│   ├── __init__.py
│   ├── imap_client.py     # IMAP email reading
│   └── smtp_client.py     # SMTP email sending
├── agent/                 # Core agent logic
│   ├── __init__.py
│   └── reply_agent.py     # Reply generation and processing
├── llm/                   # LLM integration
│   ├── __init__.py
│   └── openai_handler.py  # OpenAI API wrapper
├── config/                # Configuration management
│   ├── __init__.py
│   └── settings.py        # Environment variable parsing
├── utils/                 # Utility functions
│   ├── __init__.py
│   └── logger.py          # Logging configuration
└── tests/                 # Test suite
    ├── __init__.py
    ├── test_imap_smtp.py  # Email service tests
    └── test_agent.py      # Agent logic tests
```

## � Implementation Phases

This project follows a structured development approach:

- **Phase 1:** ✅ Project scaffold and repository setup
- **Phase 2:** ⏳ Configuration and environment setup  
- **Phase 3:** 🔄 Email integration (IMAP + SMTP)
- **Phase 4:** ⏳ LLM integration and agent logic
- **Phase 5:** ⏳ Main workflow and orchestration
- **Phase 6:** ⏳ Testing and quality assurance
- **Phase 7:** ⏳ Optional enhancements (scheduler, web dashboard)

See `ProjectPlan.md` for detailed phase information and implementation checklist.

## 🧪 Testing

Run the test suite:
```bash
pytest
```

Run tests with coverage:
```bash
pytest --cov=.
```

## 🔮 Future Enhancements

- [ ] **Automatic Scheduler:** CRON/APScheduler integration for regular monitoring
- [ ] **Web Dashboard:** FastAPI/Flask interface for reviewing and approving drafts
- [ ] **Thread Tracking:** Advanced email thread management and deduplication
- [ ] **Safety Rules:** Keyword matching to skip spam or sensitive content
- [ ] **Local LLM Support:** Integration with open-source models (Llama 3) for privacy
- [ ] **OAuth2 Integration:** Gmail API and Outlook REST API support
- [ ] **Custom Personas:** Configurable reply styles (Professional, Friendly, Concise)

## 🔧 Verification Checklist

Before deployment, ensure:
- [ ] `python main.py --dry-run` executes without errors
- [ ] `pytest` passes all tests
- [ ] Environment variables are properly configured
- [ ] IMAP/SMTP credentials work and network access is allowed
- [ ] Reply quality is acceptable with sample emails

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

---
*Built to save your time and keep your inbox at zero.*
