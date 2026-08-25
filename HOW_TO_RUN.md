# How to run this demo

Files in this folder:

- `app.py` — the local "server": a small Python program that extracts text from uploaded files and calls the two agents. You start it once and leave it running in a terminal window.
- `index.html` — the page you open in your browser to upload/paste files and click Submit.
- `.env` — your real gateway credentials. **Already created for you** — just open it and paste in your real token (see Step 1 below). Never share, email, zip, or commit this file.
- `.env.example` — a template showing what `.env` should contain, safe to share (no real secrets in it).
- `.gitignore` — makes sure `.env` never gets committed if this folder is ever put under version control.
- `HOW_TO_RUN.md` — this file.

They only talk to each other on your own PC. Nothing here is exposed to the internet.

## One-time setup

Open a terminal (Command Prompt or PowerShell) and navigate to this folder, then run:

```
pip install flask flask-cors python-docx pypdf anthropic python-dotenv
```

Then open `.env` in a text editor and replace `your-real-gateway-token-here` with your actual token — it should already have the correct `ANTHROPIC_BASE_URL` filled in. Save the file. You only need to do this once; `app.py` will read it automatically every time it starts.

## Every time you want to run the demo

**Step 1 — nothing to type.** Credentials now come from the `.env` file automatically. (If you ever need to rotate your token, just edit `.env` again — no terminal commands needed.)

**Step 2 — start the server.** In a terminal, from this folder:

```
python app.py
```

You should see something like:

```
Model: us.anthropic.claude-sonnet-4-6
Gateway configured: True
Starting server on http://localhost:5000 ...
```

Leave this window open. This is the "server running" state — it's just waiting for the page to talk to it. Closing the window stops it.

**Step 3 — open the page.** Double-click `index.html`, or right-click it and choose "Open with" your browser. At the top of the page you should see a green dot and "Connected — model: ...". If you see a red dot instead, the server isn't running or your credentials weren't set — go back to Step 1/2.

**Step 4 — use it.** For the job posting: either paste the text into the box, or click the file picker below it and choose a `.docx` file (not both — picking one disables the other). Do the same for the resume. Click **Submit**.

What happens after you click Submit, automatically — and now visibly, in the **live pipeline** panel on the left:
1. **Pre-processing** lights up first: the page sends whatever you provided to the server, and if you uploaded a `.docx`/`.pdf`, the server extracts the plain text from it.
2. **Job Description Agent** (Agent 1) lights up next: it reads the job posting and builds a requirements checklist. Its box shows a moving progress bar and a running timer while it works, then turns green with the exact number of seconds it took once it's done.
3. The connector arrow between boxes animates while a stage is handing its output to the next one, and turns green once the handoff completes.
4. **Resume Analyser Agent** (Agent 2) lights up: it reads that checklist plus the resume and scores each item, same live timer treatment.
5. **Decision** lights up last — no extra model call here, just the final assembly of both agents' output — and the total time across all stages appears at the bottom of the pipeline panel.
6. The page displays the checklist results and an overall recommendation below, as before. If any stage fails, its box turns red and the error message appears in the usual error box.

## Overriding a status (human-in-the-loop review)

Each row in the checklist results table has a **Status** dropdown instead of a fixed tag. If you (as HR) know something the AI didn't have evidence for — or disagree with its call — change the dropdown for that row. Once you do, a small "AI said: ..." note appears next to it so the original AI call is never lost, just superseded.

Once at least one row has been changed, the **Revise Fitment Analysis** button lights up. Clicking it sends the current statuses (including which ones you touched) to a third, lightweight agent whose only job is to recompute the overall verdict (`strong_fit` / `possible_fit` / `not_a_fit`) and rewrite the reasoning to match your corrected picture — it does not re-read the resume or re-litigate your override. After it runs, an "HR-revised" tag appears next to the verdict banner. Nothing is saved to disk — refreshing the page or resubmitting a new resume clears all of this, same as the rest of the demo.

You'll also see a 5th box in the live pipeline panel, "HR Revision," connected to Decision by a dashed (not solid) line. It's deliberately drawn differently from the other four boxes: it's optional, human-triggered, can run zero times or several times per screening (a counter shows "revised ×N" if you click it more than once), and it does not count toward the pipeline's "Total time" — that figure stays a measure of the automated AI-only run.

**Step 5 — when you're done**, go back to the terminal window and press `Ctrl+C` to stop the server.

## If something goes wrong

- **Red dot, "Cannot reach local server"** — the server isn't running. Go back to Step 2.
- **Red dot, "no API credentials are set"** — check that `.env` exists in this folder (not just `.env.example`) and that the token line was actually filled in and saved. Then stop the server (Ctrl+C) and run `python app.py` again — it only reads `.env` at startup.
- **"Agent returned non-JSON output"** — rare; the model didn't follow the output format. Just click Submit again.
- **"only .docx and .pdf files are supported"** — this demo handles `.docx` and text-based `.pdf` only (not old `.doc`, or images). Save the file as one of those first, or paste the text instead.
- **"this PDF has little or no extractable text"** — the PDF you uploaded is likely a scanned or photographed document with no real text underneath, just an image of text. This demo doesn't do OCR. Paste the text instead, or use a `.docx`/text-based `.pdf`.
