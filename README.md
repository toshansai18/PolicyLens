# PolicyLens
AI-powered institutional policy comparison using semantic clause matching and change detection.
# PolicyLens --- Institution Policy Comparison Assistant

> **From text differences to policy differences.**

PolicyLens compares two versions of an institutional policy at the
**clause level**. It extracts text, uses OCR when needed, aligns
corresponding clauses with semantic and structural signals, detects
additions/removals, distinguishes wording-only changes from substantive
changes, preserves source citations, explains matching evidence, and
provides a grounded PolicyLens Copilot.

## Problem

Institutional policies such as academic regulations, attendance rules,
examination rules, hostel policies, disciplinary policies, and research
policies change frequently.

Traditional document diff tools mainly show changed words. They do not
reliably answer:

> **Did the policy meaning actually change?**

Example:

**Old:** Students must submit assignments within 7 days.

**New:** Assignments must be submitted within one week.

Expected result: **WORDING-ONLY CHANGE**

But:

**Old:** Students must submit assignments within 7 days.

**New:** Students must submit assignments within 14 days.

Expected result: **SUBSTANTIVE CHANGE**

## Key Features

### 1. Clause-Level Semantic Alignment

Matches corresponding clauses using: - Semantic embeddings - Heading
similarity - Structural similarity - Position/context

### 2. Semantic Diff

Classifies: - Unchanged - Wording-only - Substantive - Added - Removed

### 3. Explainable Matching

Shows: - Semantic similarity - Heading similarity - Token overlap -
Overall match confidence - Clause lineage

### 4. Clause Lineage

Tracks a clause even when section numbering changes.

``` text
2025: §3.1 Attendance Requirement
              |
              | semantic match
              v
2026: §4.1 Attendance Eligibility
```

### 5. Policy Change Signals

Identifies observable dimensions such as: - Numeric rule - Deadline -
Eligibility - Penalty - Permission - Obligation - Responsibility

### 6. OCR Support

For scanned/image PDFs:

``` text
PDF → Text extraction
       ↓
   usable text?
    /       \
  YES        NO
   ↓          ↓
normal      OCR
pipeline     ↓
             text
```

OCR uses `pytesseract` with the Tesseract executable.

### 7. PolicyLens Copilot

A comparison-aware chatbot that can answer: - What changed between the
policies? - What changed about attendance? - How many clauses were
added? - Why were these clauses matched? - Where is this rule
mentioned? - What are the substantive changes?

The Copilot should answer from comparison evidence and provide citations
rather than behaving as an unrestricted chatbot.

### 8. Source Citations

Each clause keeps: - Document - Version - Section - Heading - Page -
Original text

### 9. Hand-Labelled Evaluation

Reports: - Labelled pairs - Correct matches - Matching accuracy -
Wording-only accuracy - Change-type accuracy

**Only actual measured results should be presented.**

## System Flow

``` text
Policy V1 ──┐
            ├→ OCR/Text Extraction → Clause Extraction
Policy V2 ──┘
                         ↓
                 Clause Embeddings
                         ↓
                 Hybrid Clause Matching
                  /        |        \
             Semantic   Heading   Structure
                         ↓
                  Aligned Clauses
                         ↓
                   Semantic Diff
                         ↓
          ┌──────────────┼──────────────┐
          ↓              ↓              ↓
        Added         Removed        Matched
                                        ↓
                              Change Classification
                                 /             \
                                ↓               ↓
                         Wording-only      Substantive
                                \               /
                                 \             /
                                  ↓           ↓
                              Evidence + Lineage
                                      ↓
                              Source Citations
                                      ↓
                              Comparison UI
                                      ↓
                              Policy Copilot
                                      ↓
                              Evaluation
```

## Technology Stack

### Frontend

-   React
-   Vite
-   CSS / Tailwind-compatible styling
-   Lucide icons
-   Recharts where required

### Backend

-   Python
-   FastAPI
-   Uvicorn

### Document Processing

-   PyMuPDF (`fitz`)
-   `python-docx`
-   `pytesseract`
-   Tesseract OCR

### Semantic Matching

-   Sentence Transformers
-   Hugging Face
-   Cosine similarity

### AI / Copilot

-   OpenAI API or Gemini API, when configured
-   Structured, evidence-grounded prompts

### Storage

-   Local files / SQLite for MVP
-   MongoDB or cloud storage can be added later if required

## Suggested Project Structure

``` text
policylens/
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   ├── parser.py
│   │   ├── clause_extractor.py
│   │   ├── matcher.py
│   │   ├── comparator.py
│   │   ├── pipeline.py
│   │   ├── ocr.py
│   │   ├── copilot.py
│   │   └── evaluator.py
│   ├── uploads/
│   ├── results/
│   ├── requirements.txt
│   └── .env
├── frontend/
│   ├── src/
│   ├── package.json
│   └── vite.config.js
├── evaluation/
│   ├── labelled_pairs.json
│   └── evaluate.py
├── sample_data/
├── README.md
└── .gitignore
```

## Main API

### Health

``` http
GET /health
```

Expected:

``` json
{
  "status": "ok",
  "service": "policylens-api"
}
```

### Compare

``` http
POST /compare
```

Multipart fields:

``` text
old_policy
new_policy
```

### Copilot

``` http
POST /copilot
```

Example:

``` json
{
  "question": "What changed about attendance?",
  "comparison_id": "cmp_001"
}
```

### Evaluation

``` http
GET /evaluate
```

Returns the actual hand-labelled benchmark metrics.

## Matching Strategy

A configurable hybrid score can combine:

``` text
Semantic similarity
+ Heading similarity
+ Structural similarity
+ Position/context
```

Example:

``` text
Final Match Score =
0.65 × semantic similarity
+ 0.20 × heading similarity
+ 0.15 × structural similarity
```

These weights and thresholds should be validated against the
hand-labelled dataset before being described as optimal.

## Change Classification

### Unchanged

The aligned clauses express equivalent requirements.

### Wording-only

The wording changed while the underlying requirement remained
equivalent.

### Substantive

The actual rule changed, such as: - Numeric threshold - Deadline -
Eligibility condition - Penalty - Permission - Obligation - Responsible
party

### Added

A clause appears only in the new version.

### Removed

A clause appears only in the old version.

## Example Demo

### Substantive

``` text
OLD: Minimum attendance = 75%
NEW: Minimum attendance = 80%

→ SUBSTANTIVE CHANGE
```

### Wording-only

``` text
OLD: Students must submit within 7 days.
NEW: Assignments must be submitted within one week.

→ WORDING-ONLY CHANGE
```

### Added

``` text
NEW: Online courses must follow separate assessment guidelines.

→ ADDED
```

### Removed

A clause present in Version 1 but absent from Version 2.

## Evaluation

Create a ground-truth dataset:

``` json
[
  {
    "old_clause_id": "v1_3_1",
    "new_clause_id": "v2_3_1",
    "expected_match": true,
    "expected_type": "SUBSTANTIVE"
  }
]
```

Calculate:

``` text
Matching Accuracy =
Correctly matched pairs / Total labelled pairs
```

and:

``` text
Wording-only Accuracy =
Correct wording-only classifications / Total wording-only cases
```

## Running Locally

### Backend

``` bash
cd backend
python -m venv .venv
```

Windows:

``` bash
.venv\Scripts\activate
```

Install:

``` bash
pip install -r requirements.txt
```

Run:

``` bash
uvicorn app.main:app --reload --port 8000
```

Check:

``` text
http://127.0.0.1:8000/health
```

### Frontend

``` bash
cd frontend
npm install
npm run dev
```

Open the Vite URL, normally:

``` text
http://localhost:5173
```

## OCR Setup

`pytesseract` is a Python wrapper. The Tesseract executable must also be
installed.

Verify:

``` bash
tesseract --version
```

If Tesseract is not in PATH, configure its executable path in the OCR
module.

Normal text PDFs do not require OCR.

## Environment Variables

Create:

``` text
backend/.env
```

Example:

``` env
OPENAI_API_KEY=your_api_key
GEMINI_API_KEY=your_api_key
```

Only configure the provider used by the implementation.

Never commit API keys.

## What Makes PolicyLens Different?

PolicyLens should not be presented as simply:

> "An AI tool that compares PDFs."

The stronger positioning is:

> **PolicyLens converts policy versions into an auditable semantic
> change map.**

The pipeline is:

``` text
Documents
→ OCR / Text Extraction
→ Clause Extraction
→ Semantic Alignment
→ Semantic Diff
→ Wording vs Substantive Classification
→ Change Signals
→ Clause Lineage
→ Evidence & Citations
→ Policy Copilot
→ Human Evaluation
```

The goal is to move from:

> **"These words changed."**

to:

> **"This policy clause changed, this is how its meaning changed, and
> these are the exact sources supporting the conclusion."**

## Demo Checklist

-   [ ] Backend health check works
-   [ ] Frontend connects to backend
-   [ ] Two policy files upload
-   [ ] Text PDF works
-   [ ] Scanned PDF OCR works
-   [ ] Clauses extracted
-   [ ] Clauses aligned
-   [ ] Added detected
-   [ ] Removed detected
-   [ ] Wording-only detected
-   [ ] Substantive detected
-   [ ] Match evidence shown
-   [ ] Clause lineage shown
-   [ ] Section/page citations shown
-   [ ] Copilot answers from comparison evidence
-   [ ] Hand-labelled evaluation completed
-   [ ] Actual accuracy numbers recorded
-   [ ] No fabricated benchmark claims

## Future Extensions

-   Multi-version policy timeline
-   Policy dependency graphs
-   Human review workflow
-   Compliance monitoring
-   Multilingual comparison
-   Enterprise document repositories
-   Change notifications
-   Advanced OCR/layout extraction
-   Exportable PDF/CSV reports
-   Role-specific summaries

------------------------------------------------------------------------

## Project Tagline

**PolicyLens --- From text differences to policy differences.**
