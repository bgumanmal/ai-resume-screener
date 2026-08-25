# AI Resume Screener

A powerful resume screening system powered by **2 AI agents** that intelligently evaluate candidates against job requirements.

## 🎯 Overview

This project implements an AI-driven resume screening pipeline that:

1. **Agent 1: Requirements Extractor** — Analyzes the job posting and creates a structured checklist of requirements
2. **Agent 2: Resume Screener** — Evaluates the candidate's resume against each requirement with evidence
3. **Human-in-the-Loop Review** — Allows HR reviewers to override AI decisions and re-analyze with their corrections

The system runs entirely on your local machine with a simple Flask server and browser-based UI, keeping all data private.

## ✨ Features

- **Dual-Agent Pipeline**: Job requirements extraction + resume evaluation with handoff pattern
- **Live Progress Tracking**: Real-time pipeline visualization showing each stage with timing
- **Smart Requirements Checklist**: 
  - Categorizes requirements (skill, experience, education, certification)
  - Marks must-haves vs. nice-to-haves
  - Identifies hard constraints
- **Evidence-Based Scoring**: Each item includes resume evidence for the AI's decision
- **Human Review Override**: HR can manually adjust any item status and trigger re-analysis
- **File Support**: Upload `.docx` and text-based PDF documents, or paste text directly
- **Clear Recommendations**: Strong Fit / Possible Fit / Not a Fit with detailed reasoning
- **No Internet Required**: Everything runs locally — credentials only used for API calls

## 🚀 Quick Start

### Prerequisites

- Python 3.8+
- pip
- Claude API credentials (via Anthropic gateway or direct API key)

### Installation

1. **Clone the repository**:
```bash
git clone https://github.com/bgumanmal/ai-resume-screener.git
cd ai-resume-screener
```

2. **Create a virtual environment** (recommended):
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

3. **Install dependencies**:
```bash
pip install flask flask-cors python-docx pypdf anthropic python-dotenv
```

4. **Set up credentials**:
   - Copy `.env.example` to `.env`
   - Fill in your credentials:
     - `ANTHROPIC_BASE_URL` (gateway endpoint)
     - `ANTHROPIC_AUTH_TOKEN` (gateway token), or
     - `ANTHROPIC_API_KEY` (direct API key)
   - Optionally set `ANTHROPIC_MODEL` (defaults to `us.anthropic.claude-sonnet-4-6`)

### Running the Demo

1. **Start the server**:
```bash
python app.py
```

You should see:
```
Model: us.anthropic.claude-sonnet-4-6
Gateway configured: True
Starting server on http://localhost:5000 ...
```

2. **Open the UI**:
   - Double-click `index.html` in the folder, or
   - Right-click → "Open with" → your browser

3. **Use it**:
   - Paste job posting text or upload a `.docx`/`.pdf`
   - Paste resume text or upload a `.docx`/`.pdf`
   - Click **Submit**
   - Watch the pipeline lights up in real time
   - Review results and override any item statuses if needed
   - Click **Revise Fitment Analysis** to re-score with your corrections

4. **Stop the server**:
   - Press `Ctrl+C` in the terminal

## 📋 Project Structure

```
ai-resume-screener/
├── app.py                  # Flask server with 2-agent pipeline + HR revision logic
├── index.html              # Browser UI with live pipeline visualization
├── .env                    # Your credentials (git-ignored, never committed)
├── .env.example            # Template for .env (safe to share)
├── .gitignore              # Prevents .env from being committed
├── HOW_TO_RUN.md           # Detailed setup & troubleshooting guide
├── sample_ID.docx          # Example candidate resume
└── README.md               # This file
```

## 🔄 Pipeline Architecture

### Stage 1: Pre-processing
- Extracts text from uploaded `.docx` or PDF files
- Validates file types
- Detects scanned/image-based PDFs (not supported)

### Stage 2: Requirements Extraction (Agent 1)
- Reads the job posting
- Builds a structured checklist with:
  - `role_title`
  - `requirements` (with type, category, checkable item text)
  - `hard_constraints` (e.g., location, clearances)

**Model**: Claude Sonnet 4 (configurable)

### Stage 3: Resume Screening (Agent 2)
- Takes Agent 1's checklist + candidate resume
- Scores each requirement:
  - `met` / `partially_met` / `not_met` / `unclear_from_resume`
  - Includes evidence quote from the resume
- Checks hard constraints
- Produces overall recommendation:
  - `strong_fit` / `possible_fit` / `not_a_fit`
  - 2–3 sentence rationale

**Model**: Claude Sonnet 4 (configurable)

### Stage 4: Decision Assembly
- Combines Agent 1 + Agent 2 outputs
- Displays full results in the UI

### Stage 5: Human-in-the-Loop Revision (Optional)
- HR reviews the checklist and overrides any item status
- Clicks **Revise Fitment Analysis**
- Agent 3 (Fitment Reviser) re-weighs the items with the human corrections
- Produces a new overall recommendation & rationale that reflects the changes
- Does **NOT** re-examine the resume — treats human edits as ground truth

## 🛠️ Supported File Types

| Format | Support |
|--------|---------|
| `.docx` | ✅ Full support |
| `.pdf` (text-based) | ✅ Full support |
| `.pdf` (scanned/image) | ❌ Not supported — paste text instead |
| Plain text (paste) | ✅ Always works |

## 📊 API Endpoints

### `POST /process`
Runs the full 2-agent pipeline. Returns JSON with both agents' outputs.

**Request**:
- `job_text` or `job_file`: Job posting (text or file)
- `resume_text` or `resume_file`: Resume (text or file)

**Response**:
```json
{
  "requirements": { ... },
  "screening": { ... }
}
```

### `POST /process_stream`
Same pipeline, but streams progress as newline-delimited JSON (NDJSON).

Each line:
```json
{"stage": "preprocess", "status": "start"}
{"stage": "preprocess", "status": "done", "elapsed": 0.45}
{"stage": "agent1", "status": "start"}
{"stage": "agent1", "status": "done", "elapsed": 3.2, "data": {...}}
...
```

### `POST /revise`
Re-scores items after HR overrides. Takes current item statuses + hard constraints.

**Request**:
```json
{
  "role_title": "...",
  "item_results": [
    {"item": "...", "type": "must_have", "status": "met", "evidence": "...", "human_edited": true},
    ...
  ],
  "hard_constraints_check": [...]
}
```

**Response**:
```json
{
  "overall_recommendation": "strong_fit",
  "recommendation_reason": "..."
}
```

### `GET /health`
Health check. Returns model name and credential status.

## ⚙️ Configuration

All settings are in `.env`:

```env
# Anthropic Gateway (recommended for teams)
ANTHROPIC_BASE_URL=https://your-gateway-url
ANTHROPIC_AUTH_TOKEN=your-gateway-token

# OR direct Anthropic API (for individual use)
ANTHROPIC_API_KEY=sk-ant-v7-...

# Optional: model to use (defaults to us.anthropic.claude-sonnet-4-6)
ANTHROPIC_MODEL=us.anthropic.claude-sonnet-4-6
```

## 🐛 Troubleshooting

| Issue | Solution |
|-------|----------|
| **Red dot, "Cannot reach local server"** | Ensure `python app.py` is running; check port 5000 is not in use |
| **Red dot, "no API credentials"** | Check `.env` exists in this folder and token is filled in (not `.env.example`) |
| **"Agent returned non-JSON output"** | Rare; usually model cut off due to token limits. Try again. |
| **"only .docx and .pdf files are supported"** | Upload `.docx` or `.pdf`, or paste text instead |
| **"this PDF has little or no extractable text"** | PDF is likely scanned/image-based. Paste the text instead. |
| **Slow responses** | Model calls take 3–10 seconds depending on document length; normal |

See `HOW_TO_RUN.md` for more detailed troubleshooting.

## 🔐 Privacy & Security

- All processing happens on your local machine
- The `.env` file is git-ignored (never committed)
- Credentials are only used to call the AI model
- No data is logged or stored on external servers
- Uploaded files are only held in memory during processing

## 📝 License

MIT License — feel free to fork, modify, and distribute.

## 👤 Author

Created by **bgumanmal** ([GitHub](https://github.com/bgumanmal))

## 🤝 Contributing

Have ideas for improvements? Found a bug? Open an issue or submit a PR!

---

**Ready to screen resumes smarter?** Clone, set up credentials, and run `python app.py` now!
