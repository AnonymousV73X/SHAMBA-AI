# 🌿 Shamba AI — Crop Disease Detection

> **Detect crop diseases instantly. Save your harvest. Save your income.**

Shamba AI is a full-stack AI-powered crop disease detection platform built for Kenyan smallholder farmers. Point your phone at a sick leaf, get a diagnosis in seconds — treatment steps, product recommendations, and prevention tips included. Works on 2G/3G. No login required.

---

## ✨ Features

| Feature | Details |
|---|---|
| 🔍 **Instant AI Diagnosis** | Upload a leaf photo → disease name, confidence score & severity in seconds |
| 🧠 **Dual Inference Engine** | Local ONNX (MobileNetV2, PlantVillage) with Gemini Vision API fallback |
| 💬 **AI Chat Assistant** | Ask follow-up questions about your diagnosis in context |
| 📄 **PDF Report Export** | Save and share your diagnosis report |
| 🌍 **Bilingual — EN / SW** | Full English & Swahili UI, switchable mid-session |
| 🌙 **Dark Mode** | Deep black-green theme, preference persisted |
| 📱 **Mobile-First** | Optimised for low-end Android devices on 2G/3G |
| 🚫 **No Login Required** | Anonymous scanning — account optional for scan history |
| 👨‍💼 **Admin Dashboard** | Manage users, diseases, and view scan analytics |
| 📞 **Expert Connect** | KALRO hotline modal for escalating to human agronomists |

---

## 🦠 Supported Crops & Diseases

| Crop | Diseases Detected |
|---|---|
| 🌽 **Maize** | Lethal Necrosis (MLND), Gray Leaf Spot, Rust, Fall Armyworm |
| 🍅 **Tomato** | Early Blight, Late Blight, Bacterial Spot, Yellow Leaf Curl Virus |
| 🥬 **Kale (Sukuma Wiki)** | Black Rot, Downy Mildew, Aphids, Cabbage Worm |
| 🥔 **Potato** | Late Blight |
| 🧅 **Onion** | Purple Blotch |
| 🫘 **Beans** | Gemini Vision open-ended diagnosis |

> The Gemini Vision fallback can diagnose **any** visible disease, disorder, or pest damage — including physiological disorders (nutrient deficiency, sunscald, blossom end rot) not in the local model.

---

## 🏗️ Architecture

```
shamba-ai/
├── frontend/
│   ├── index.html              ← Scan page (EN/SW, dark mode, crop picker, upload)
│   ├── result.html             ← Diagnosis result card + AI chat panel
│   ├── history.html            ← User scan history
│   ├── login.html              ← Login / register
│   └── admin_dashboard.html   ← Admin panel (users, diseases, analytics)
│
├── backend/
│   ├── app.py                  ← FastAPI entry point, router mounting, lifespan
│   ├── config.py               ← Env config (DB, JWT, Gemini, ONNX paths)
│   ├── routes/
│   │   ├── detect.py           ← POST /api/detect — core detection endpoint
│   │   ├── chat.py             ← POST /api/chat — Gemini-powered Q&A
│   │   ├── auth.py             ← Register, login, JWT
│   │   ├── history.py          ← Scan history (authenticated)
│   │   └── admin.py            ← Admin CRUD endpoints
│   ├── ai/
│   │   ├── model.py            ← ONNX MobileNetV2 wrapper + PlantVillage label mapping
│   │   ├── predict.py          ← Inference pipeline: ONNX → Gemini fallback
│   │   ├── gemini_vision.py    ← Gemini Vision API integration
│   │   └── advice.py           ← Disease advice JSON lookup
│   ├── db/
│   │   └── database.py         ← SQLite (WAL), schema init, demo seed
│   └── utils/
│       ├── image_utils.py      ← Image validation, resize, format check
│       └── auth_utils.py       ← JWT sign/verify, PBKDF2 password hashing
│
├── model/
│   ├── plantvillage_mobilenetv2.onnx   ← Local CNN model (38 classes)
│   ├── plantvillage_labels.json        ← PlantVillage class labels
│   ├── labels.json                     ← App disease labels (mapped)
│   └── disease_advice.json             ← Treatment, products, prevention per disease
│
├── uploads/                    ← Saved scan images
├── requirements.txt
└── README.md
```

---

## 🧠 AI Inference Pipeline

```
User uploads image
        │
        ▼
┌─────────────────────┐
│  ONNX MobileNetV2   │  ← Local, fast, offline-capable
│  (PlantVillage 38)  │
└────────┬────────────┘
         │ mapped result?
    ┌────┴────┐
   YES        NO / unmapped
    │          │
    ▼          ▼
Return     ┌──────────────────┐
result     │  Gemini Vision   │  ← Open-ended diagnosis, any disease
           │  API (fallback)  │
           └──────────────────┘
                    │
                    ▼
          Map to known DB entry
          or return raw diagnosis
                    │
                    ▼
         Advice lookup (treatment
         steps, products, prevention)
```

---

## 🚀 Getting Started

### Prerequisites

- Python 3.9+
- `pip`
- Gemini API key (for fallback + chat)

### Installation

```bash
# 1. Clone the repo
git clone https://github.com/yourname/shamba-ai.git
cd shamba-ai

# 2. Install Python dependencies
pip install -r requirements.txt

# 3. Set environment variables (or edit config.py)
export GEMINI_API_KEY="your-gemini-api-key"
export JWT_SECRET="your-secret-key"

# 4. Start the server
cd backend
python app.py
# or
uvicorn app:app --reload --host 0.0.0.0 --port 8001
```

App runs at **http://localhost:8001**

### Default Admin Credentials

```
Email:    admin@shamba.ai
Password: admin123
```

> ⚠️ Change these before deploying to production.

---

## 📡 API Reference

### Detection

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/api/detect` | Optional | Detect disease from image. Multipart: `image` (file) + `crop_type` (string) |

**Response:**
```json
{
  "disease": "Tomato_Early_Blight",
  "display": "Early Blight",
  "confidence": 91.4,
  "severity": "Moderate",
  "symptoms": ["..."],
  "treatment_steps": [{ "step": 1, "text": "..." }],
  "products": [{ "name": "Mancozeb", "type": "Fungicide", "availability": "Agrovets" }],
  "prevention": ["..."],
  "source": "local_onnx"
}
```

### Chat

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/api/chat` | Optional | Ask follow-up about diagnosis. Body: `{ message, context, history, lang }` |

### Auth

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/auth/register` | Create account |
| `POST` | `/api/auth/login` | Login → JWT token |
| `GET` | `/api/auth/me` | Current user info |

### History

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `GET` | `/api/history` | Required | List user's past scans |
| `GET` | `/api/history/{id}` | Required | Single scan result |

### Admin

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/admin/stats` | Dashboard stats |
| `GET/POST` | `/api/admin/diseases` | List / add diseases |
| `PUT/DELETE` | `/api/admin/diseases/{id}` | Update / delete disease |
| `GET` | `/api/admin/users` | List all users |
| `PUT` | `/api/admin/users/{id}/ban` | Ban / unban user |

### Health

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/health` | Server health check |

---

## 🛠️ Tech Stack

**Backend**
- [FastAPI](https://fastapi.tiangolo.com/) + Uvicorn
- SQLite with WAL mode (via SQLAlchemy)
- JWT auth (PyJWT) + PBKDF2 password hashing (passlib)
- ONNX Runtime — MobileNetV2 local inference
- Google Gemini Vision API — open-ended fallback diagnosis + chat
- Pillow — image validation & preprocessing

**Frontend**
- Vanilla HTML / CSS / JS — zero frameworks, zero build step
- Outfit font (Google Fonts)
- Bilingual i18n system (EN / SW) with `localStorage` persistence
- Dark mode via CSS custom properties + `data-theme` attribute
- PDF export via browser print API

---

## 🌍 Localisation

The UI is fully bilingual. Language is saved to `localStorage` under key `shamba_lang` and shared across all pages.

| Key | English | Swahili |
|-----|---------|---------|
| Crops | Maize, Tomato, Kale... | Mahindi, Nyanya, Sukuma Wiki... |
| Actions | Analyse Crop | Chunguza Zao |
| Upload | Tap to upload or drag & drop | Gonga ili kupakia au buruta & acha |
| Chat | Ask Expert | Uliza Mtaalamu |

---

## 🔒 Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `DATABASE_URL` | `sqlite:///./shamba.db` | Database connection string |
| `JWT_SECRET` | *(insecure default)* | **Change in production** |
| `GEMINI_API_KEY` | — | Required for fallback diagnosis + chat |

---

## 📦 Deployment Notes

- The SQLite DB auto-creates and seeds on first startup
- ONNX model file (`plantvillage_mobilenetv2.onnx`) must be present in `/model/` for local inference — server starts without it and falls back to Gemini
- Set `CORS` origins explicitly in `app.py` before going to production (currently `allow_origins=["*"]`)
- Serve behind Nginx or Caddy in production; FastAPI serves the frontend as static files in development

---

## 📄 License

MIT License — free for agricultural, educational, and commercial use.

---

*🌿 Shamba AI — Kuwawezesha wakulima wa Kenya kwa AI*  
*Empowering Kenyan farmers with artificial intelligence*