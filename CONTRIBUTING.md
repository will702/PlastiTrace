# Contributing to PlastiTrace

Thank you for contributing. This guide covers local setup, verification, and what to check before opening a pull request.

## Prerequisites

| Tool | Version | Used for |
|------|---------|----------|
| Python | 3.11+ | Desktop app, API, ML pipeline |
| Node.js | 20+ | Web frontend (`web/`) |
| npm | 10+ | Web dependencies |
| Webcam | Optional | Desktop realtime testing |

On macOS, install Xcode Command Line Tools if NumPy or OpenCV fail to build:

```bash
xcode-select --install
```

## Clone and setup

```bash
git clone https://github.com/will702/PlastiTrace.git
cd PlastiTrace

# Python environment
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Web frontend
cd web
cp .env.example .env.local
npm install
cd ..
```

Place the model weights at `models/plastitrace.pth` if they are not already present.

## Run locally

### Desktop app

```bash
source venv/bin/activate
python app.py
```

### API + web (two terminals)

```bash
# Terminal 1
source venv/bin/activate
python api.py

# Terminal 2
cd web && npm run dev
```

Open `http://localhost:3000`. The frontend reads `NEXT_PUBLIC_API_URL` from `web/.env.local` (defaults to `http://localhost:5001`).

### Trust layer tests

```bash
source venv/bin/activate
python test_trust.py
```

## Repo layout

See the architecture diagrams and project tree in [README.md](README.md). Key boundaries:

- **`app.py`** - Desktop only (PyQt5, realtime workers)
- **`api.py`** - Web backend only (stateless image classify)
- **`web/`** - Next.js frontend (deployed to Vercel)
- **`workers/`** - QThread capture and inference (desktop)
- **`ml/`**, **`vision/`**, **`realtime/`** - Shared ML and vision logic
- **`location/`**, **`domain/`** - Drop-off map data (desktop)
- **`trust/`**, **`feedback/`** - Quality gating and dataset tooling

## Before submitting

1. **No secrets** - Do not commit `.env`, `.env.local`, API keys, or tokens.
2. **No cache files** - `__pycache__/`, `*.pyc`, `.DS_Store` must stay out of git (see `.gitignore`).
3. **No agent tooling** - Do not commit `.cursor/`, `.agents/`, `.claude/`, `.gemini/`, or `skills-lock.json`.
4. **Web changes** - Run `cd web && npm run build` and fix any TypeScript or build errors.
5. **Python import check** - After UI changes, verify: `python -c "from ui.main_window import MainWindow"`.
6. **README accuracy** - If you add or remove entry points, update README.md and this file.

## Deployment notes

| Component | Host | Config |
|-----------|------|--------|
| Frontend | Vercel | Root directory: `web`, env: `NEXT_PUBLIC_API_URL` |
| API | Render / Railway / HF Spaces | Gunicorn recommended; set `debug=False` |
| Desktop | Local / PyInstaller | Package `app.py` with model weights |

Production API checklist:

- Set `debug=False` in `api.py` or use Gunicorn
- Restrict CORS to your Vercel domain
- Serve over HTTPS (required for mobile camera on web)
- Add upload size limits and file type validation

## Questions

Open a GitHub issue if setup steps fail or documentation is out of date.
