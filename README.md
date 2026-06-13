# 🤖 AI-Based Multilingual Chatbot for Welfare Scheme Awareness

> **Challenge 1.1 — AI & Intelligent Systems Track**
> A low-bandwidth, multilingual conversational assistant deployable over WhatsApp and SMS to help rural Indian users discover government welfare schemes they qualify for — in their own language.

---

## 🌟 Why This Exists

Millions of eligible citizens miss out on government welfare schemes simply because they don't know they exist, can't navigate complex portals, or don't speak English. This chatbot fixes that — a 5-question conversation over WhatsApp is all it takes to get a personalized list of schemes with a document checklist, in Hindi, Tamil, Bengali, Marathi, or English.

---

## ✨ Features at a Glance

| Feature | Details |
|---|---|
| 🌐 **Languages** | English, Hindi, Tamil, Bengali, Marathi + Hinglish code-mixing |
| 🧠 **AI Architecture** | RAG with ChromaDB + Gemini 2.5 Flash — zero hallucinated eligibility rules |
| 📱 **Channels** | WhatsApp (Twilio) + SMS (Twilio) |
| 🎤 **Voice Support** | Voice note transcription via Gemini |
| 🔄 **Eligibility Flow** | 5-question state machine → personalized scheme shortlist |
| 📋 **Schemes Covered** | 10 high-impact government schemes |
| 📄 **Document Checklist** | Downloadable/SMS-able list in user's language |
| ✅ **Anti-Hallucination** | Strict RAG grounding — never invents rules |

---

## 🏛️ Welfare Schemes Covered

| # | Scheme | Benefit | Target |
|---|---|---|---|
| 1 | **PM-KISAN** | ₹6,000/year direct transfer | Farmers |
| 2 | **Kisan Credit Card (KCC)** | Low-interest crop loans | Farmers |
| 3 | **MGNREGA** | 100 days guaranteed employment | Gig/Daily workers |
| 4 | **PM SVANidhi** | Micro-credit working capital loan | Street vendors |
| 5 | **Ayushman Bharat** | ₹5 lakh health insurance cover | BPL families |
| 6 | **PMJJBY** | ₹2 lakh life insurance | General public |
| 7 | **PMUY / Ujjwala** | Free LPG connection | Women (BPL) |
| 8 | **PMAY-G** | Housing construction assistance | Rural poor |
| 9 | **Sukanya Samriddhi Yojana** | Long-term savings for girl child | Parents |
| 10 | **Janani Suraksha Yojana (JSY)** | Cash incentive for institutional delivery | Pregnant women |

---

## 🏗️ Architecture

```
User (WhatsApp / SMS)
        │
        ▼
   Twilio Webhook
        │
        ▼
  FastAPI — app/main.py
        │
        ├── 🎤 Audio? → Gemini transcription (audio_handler.py)
        │
        ├── 🔄 State Machine (chatbot.py)
        │     ├── Language Detection (script + word-level)
        │     ├── 5-question eligibility flow
        │     └── User profile collection
        │
        ├── 🧠 RAG Pipeline
        │     ├── ChromaDB vector store (10 schemes)
        │     ├── HuggingFace embeddings (all-MiniLM-L6-v2)
        │     └── Top-4 relevant documents retrieved
        │
        └── ✨ Gemini 2.5 Flash
              └── Localized recommendation + document checklist
```

---

## 💬 How the Eligibility Flow Works

```
User says: "Hi" / "नमस्ते" / "வணக்கம்" / "নমস্কার"
    └─► Language auto-detected → greeting sent in user's language

Q1 → What is your occupation?
     [ Farmer | Daily wage worker | Street vendor | Homemaker | Other ]

Q2 → Do you live in a rural or urban area?
     [ Rural | Urban ]

Q3 → What is your gender?
     [ Male | Female ]

Q4 → What is your annual household income?
     [ Below ₹1 lakh | ₹1–3 lakh | Above ₹3 lakh ]

Q5 → Any of these apply to your family?
     [ Girl child | Pregnant woman | No pucca house | No LPG | None ]

    └─► RAG retrieves top 4 matching schemes from ChromaDB
    └─► Gemini generates personalized recommendation + document checklist

Follow-up → Ask anything: "How do I apply?" / "What documents do I need?"
Anytime   → Type "checklist" → receive SMS-ready document list
Anytime   → Type "restart" or "hi" → start over
```

---

## ⚙️ Prerequisites

- Python 3.9+
- [Gemini API Key](https://aistudio.google.com/app/apikey) (free)
- [Twilio Account](https://www.twilio.com/) (free trial works)
- [ngrok](https://ngrok.com/) (for local webhook testing)

---

## 🚀 Setup Instructions

### 1. Clone and install dependencies

```bash
git clone <repo-url>
cd AI-Based-Multilingual-Chatbot-for-Welfare-Scheme-Awareness
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### 2. Configure environment variables

```bash
cp .env.example .env
```

Open `.env` and fill in:

```env
GOOGLE_API_KEY="your_gemini_api_key_here"
TWILIO_ACCOUNT_SID="your_twilio_account_sid_here"
TWILIO_AUTH_TOKEN="your_twilio_auth_token_here"
```

### 3. Populate the vector database

```bash
python populate_db.py
```

Expected output:
```
Loaded 10 schemes.
Database populated successfully at './chroma_db'.
```

### 4. Start the FastAPI server

```bash
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

### 5. Expose locally via ngrok

```bash
ngrok http 8000
```

Copy the `https://xxxx.ngrok-free.app` forwarding URL.

### 6. Configure Twilio WhatsApp Sandbox

- Go to **Twilio Console → Messaging → Try it out → Send a WhatsApp message**
- Click **Sandbox settings**
- Set **"When a message comes in"** to:
  ```
  https://<your-ngrok-url>/webhook
  ```
- Method: `POST` → **Save**

### 7. Connect your phone

Send this message from your WhatsApp to **+1 415 523 8886**:
```
join <your-sandbox-keyword>
```

Then send **"Hi"** — the chatbot will respond!

---

## 🔌 API Endpoints

| Endpoint | Method | Description |
|---|---|---|
| `/` | `GET` | API info and feature list |
| `/health` | `GET` | Health check + active session count |
| `/webhook` | `POST` | WhatsApp Twilio webhook |
| `/sms` | `POST` | SMS Twilio webhook |
| `/checklist/{session_id}` | `GET` | Get SMS-able document checklist |
| `/session/{session_id}` | `GET` | Debug: view session state |

---

## 🧪 Testing

### Simulate WhatsApp locally (PowerShell)

```powershell
# Test webhook
Invoke-WebRequest -Uri "http://localhost:8000/webhook" -Method POST -Body "From=whatsapp:+919999999999&Body=Hi&NumMedia=0"

# Test SMS
Invoke-WebRequest -Uri "http://localhost:8000/sms" -Method POST -Body "From=+919999999999&Body=Hi"

# Health check
Invoke-WebRequest -Uri "http://localhost:8000/health"
```

### Language testing

| Language | What to send |
|---|---|
| English | `Hi` |
| Hindi | `नमस्ते` or `main kisan hu` |
| Tamil | `வணக்கம்` |
| Bengali | `নমস্কার` |
| Hinglish | `mera aadhaar kho gaya` |

---

## 📊 Success Metrics

| Metric | Target | How |
|---|---|---|
| Eligibility flow completion | ≥ 80% | 5-question flow, concise messages |
| Comprehension score | ≥ 70% | Clear scheme summaries + document lists |
| Response latency | < 3 seconds | Lightweight embeddings, concise prompts |
| Code-mixing accuracy | High | Script detection + Hinglish word list |

---

## 📁 Project Structure

```
├── app/
│   ├── __init__.py
│   ├── main.py           # FastAPI server + webhook endpoints
│   ├── chatbot.py        # State machine + RAG pipeline
│   └── audio_handler.py  # Voice note transcription
├── data/
│   └── schemes_data.json # Scheme data (multilingual)
├── populate_db.py        # Script to build ChromaDB vector store
├── requirements.txt
├── .env.example
├── USER_FLOW_DIAGRAM.md  # Flow diagrams for 3 personas
└── Pilot_Test_and_Impact.md  # Pilot test report + impact projection
```

---

## 📜 License

MIT License — free to use, modify, and distribute.

---

*Built for Challenge 1.1 — AI & Intelligent Systems Track*
