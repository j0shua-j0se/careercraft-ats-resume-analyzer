# CareerCraft — ATS-optimised resume analyser (Gemini)

A Streamlit web app that compares a PDF résumé against a pasted job description
using Google's Gemini model, and returns a match percentage, the keywords the
résumé is missing, and a profile summary.

Built during the **SmartInternz / SmartBridge Generative AI internship**
(Jun–Jul 2024) as a guided project, and deployed publicly.

- **Live app:** https://resume-analyser-q098.onrender.com
  *(free Render instance — the first request can take ~60 s while it wakes up)*
- **Demo video:** https://youtu.be/0x7n1-EovxE
- **Project report:** [`docs/careercraft-project-report.pdf`](docs/careercraft-project-report.pdf)

## What it actually does

1. **PDF text extraction** — `PyPDF2` reads the uploaded résumé page by page
   (`input_pdf_text`).
2. **Prompting** — the résumé text and the job description are inserted into a
   single instruction prompt that casts the model as an experienced ATS and
   fixes the response format: match percentage on the first line, missing
   keywords on the second, profile summary in the third section.
3. **Model call** — `google-generativeai` with `gemini-pro`
   (`get_gemini_response`), wrapped in error handling that surfaces failures in
   the UI instead of crashing the app.
4. **Result parsing and display** — the percentage is parsed out of the first
   response line and drawn as a matplotlib doughnut chart next to the full
   text feedback.
5. **UI** — a single-page Streamlit layout (custom CSS, offerings section, FAQ)
   with a job-description text area and a PDF uploader.

## Tech stack

Python 3.9 · Streamlit · `google-generativeai` (Gemini) · PyPDF2 ·
python-dotenv · Pillow · matplotlib

## Running it locally

```bash
git clone https://github.com/j0shua-j0se/careercraft-ats-resume-analyzer
cd careercraft-ats-resume-analyzer
python -m venv .venv && .venv\Scripts\activate    # Windows
# source .venv/bin/activate                        # macOS / Linux
pip install -r requirements.txt
cp .env.example .env        # then put your own Gemini API key in .env
streamlit run app.py
```

### API key handling

The app reads `GOOGLE_API_KEY` from a local `.env` via `python-dotenv`
(`load_dotenv()` in `app.py`). **`.env` is git-ignored and is not in this
repository** — only [`.env.example`](.env.example) is, as a template. Get your
own key from [Google AI Studio](https://aistudio.google.com/app/apikey) and keep
it out of version control. On Render the same variable is set as an environment
variable in the service settings, not as a file.

## Repository layout

```
app.py          Streamlit app: UI, PDF extraction, prompt, Gemini call, chart
images/         icons used in the UI
requirements.txt
.env.example    template for the one required environment variable
.python-version 3.9.6
docs/           project report (PDF)
```

## Limitations and known issues

Stated plainly, because they are the interesting part of the project:

- **The match percentage is model-generated, not computed.** There is no
  embedding similarity or deterministic keyword scoring behind it, so the same
  résumé and job description can produce slightly different numbers between
  runs. It is directional feedback, not a measurement.
- **No retrieval and no evaluation set.** The app is a single prompt call, not a
  RAG pipeline, and output quality has not been measured against a labelled
  benchmark.
- **Parsing is brittle** — the percentage is taken from the first line of free
  text (`response.split('\n')[0]`), so a formatting change in the model's answer
  falls back to 0% with a visible error.
- **PDF-only, text-only** — scanned or image-based résumés extract nothing
  (no OCR), and layout/columns can garble the extracted text.
- `gemini-pro` and Streamlit's `use_column_width` are both dated; the model name
  and image API would need updating against current versions.

## Possible next steps

Deterministic keyword matching alongside the model's judgement, embedding-based
section similarity, a small labelled set of résumé/JD pairs to measure whether
the scores mean anything, structured (JSON) model output instead of parsed free
text, and OCR for scanned résumés.
