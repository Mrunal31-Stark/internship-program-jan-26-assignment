# GenAI Assignment:

**Evaluation Criteria**

We will score your submission on:

* Clarity and practicality of architecture
* Robust JSON schema design
* Prompt quality (zero-shot, reliable, minimal hallucination risk)
* Handling of ambiguity + user review flow
* Bulk generation thinking (errors, naming, report)

## Problem 1: **Proposal for “Video-to-Notes”**

We have a local folder of long videos (3–4 hours each, 200MB+). Watching them fully is slow. We need an automated way to generate a “summary package” per video: **Summary.md** + highlight clips + screenshots, all organized per video. [READ MORE ABOUT THE PROJECT](./Video-summary-platform.md)

### **Task**

Prepare a **pre-processed solution proposal** comparing  **three approaches** **:**

1. **Online/Cloud-Based (Already Available Solutions)**
2. **Build Our Own Using LLM APIs (Hybrid: local media processing + cloud LLM)**
3. **Build Fully Offline Using Open-Source Models (Local transcription + local LLM + pipeline)**

No code required. We want a **clear, practical proposal** with architecture and tradeoffs.

### Your Solution for problem 1:

Input:
Folder with long videos (3–4 hours, 200MB+ each)

Output (per video):

output/<video_name>/
   Summary.md
   clips/
   screenshots/


Constraints:

Batch mode

Timestamp-aligned clips & screenshots

5–10 minute readable summary

Scales to many long videos

Approach 1 — Online / Cloud-Based (Already Available Solutions)

Examples:

OpenAI (Whisper + GPT + Assistants)

Google Cloud (Speech-to-Text + Vertex AI)

AssemblyAI

Descript

 Architecture
Local Folder
   ↓
Upload to Cloud Storage (S3 / GCS)
   ↓
Cloud Transcription (Whisper / STT API)
   ↓
LLM Summarization + Highlight Detection
   ↓
Timestamp Extraction
   ↓
Return JSON Summary Spec
   ↓
Local FFmpeg Processing
   ↓
Generate Summary.md + Assets

✔ Strengths

Highest transcription accuracy

Fast scaling

Minimal infra maintenance

Built-in diarization, topic segmentation

Production-ready reliability

 Weaknesses

Expensive for long videos

Data privacy risk

Upload time for 200MB+ files

Rate limits for batch jobs

 JSON Output Schema (Cloud LLM Contract)
{
  "video_metadata": {
    "filename": "string",
    "duration_seconds": 0,
    "processed_at": "ISO8601"
  },
  "high_level_summary": "string",
  "highlights": [
    {
      "title": "string",
      "start_timestamp": "HH:MM:SS",
      "end_timestamp": "HH:MM:SS",
      "description": "string",
      "importance_score": 0.0,
      "confidence_score": 0.0
    }
  ],
  "key_takeaways": [
    "string"
  ],
  "uncertain_sections": [
    {
      "reason": "string",
      "timestamp": "HH:MM:SS"
    }
  ]
}


✔ Deterministic
✔ Forces timestamps
✔ Confidence scoring
✔ Ambiguity surfaced

 Prompt Strategy (Zero-Shot, Low Hallucination)

System Prompt:

You are a factual video summarization engine.
Use only the provided transcript segments.
Do not invent content not present in the transcript.
Output strictly valid JSON according to schema.
If uncertain, mark in "uncertain_sections".

Why this works:

Forces grounding

Prevents hallucinated highlights

Encourages uncertainty reporting

Best For

Enterprise

Fast deployment

Accuracy > cost

 Approach 2 — Hybrid (Local Media + Cloud LLM APIs)

This is likely the most practical balance.

Architecture
Batch Folder
   ↓
Local Processing (FFmpeg chunking)
   ↓
Local Transcription (Whisper small/medium)
   ↓
Segmented Transcript (5–10 min chunks)
   ↓
Cloud LLM (Summarize + Highlight Selection)
   ↓
Highlight JSON Spec
   ↓
Local Clip + Screenshot Extraction
   ↓
Markdown Generator

 Key Design Principle

Do NOT send full 4-hour transcript at once.

Instead:

Chunk transcript by time (e.g., 5 minutes)

Summarize per chunk

Aggregate summaries

Final compression pass

This reduces:

Token overflow

Cost

Hallucination risk

 Improved JSON Schema (Production-Ready)
{
  "version": "1.0",
  "video": {
    "filename": "string",
    "duration_seconds": 0
  },
  "processing": {
    "transcript_model": "string",
    "llm_model": "string",
    "processed_at": "ISO8601"
  },
  "summary": {
    "high_level": "string",
    "reading_time_minutes": 0
  },
  "highlights": [
    {
      "id": "HL_001",
      "start_sec": 0,
      "end_sec": 0,
      "title": "string",
      "description": "string",
      "keywords": ["string"],
      "confidence": 0.0
    }
  ],
  "takeaways": ["string"],
  "review_required": false
}


✔ Numeric timestamps (less parsing errors)
✔ Versioning
✔ Traceability
✔ Model tracking
✔ Review flag

Ambiguity Handling Strategy

LLM instructed to:

Mark low-confidence highlights (<0.6)

Avoid summarizing unclear segments

Flag audio quality issues

User review flow:

If review_required = true
   → user inspects only flagged highlights
   → approve / delete / edit


Scales well for bulk processing.

Error Handling (Batch Mode Thinking)

Per video:

Transcription failure → skip + log

LLM timeout → retry 2 times

Invalid JSON → auto-repair prompt

Asset extraction mismatch → re-cut using raw timestamps

Generate:

output/report.json

{
  "total_videos": 20,
  "successful": 18,
  "failed": 2,
  "average_processing_time_min": 14.2
}

✔ Strengths

Balanced cost

Good privacy

High accuracy

Scalable

Robust batch control

Weaknesses

Some API dependency

Needs orchestration logic

 Approach 3 — Fully Offline (Open Source Only)

Components:

Whisper (local)

LLaMA

Mistral

FFmpeg

Orchestration script

Architecture
Video Folder
   ↓
Local GPU Transcription (Whisper large-v3)
   ↓
Semantic Segmentation (Embedding clustering)
   ↓
Local LLM Summarization
   ↓
Highlight Scoring
   ↓
Clip + Screenshot Extraction
   ↓
Markdown Assembly

⚠ Critical Reality

Offline LLMs:

Lower summarization quality

More hallucination risk

Need more prompt engineering

Slower

But:

✔ Zero data leaves system
✔ One-time infra cost
✔ Unlimited scale

📦 Schema (Same as Hybrid)

Important design decision:
Keep schema identical across all approaches

This ensures:

Replaceable backend

Comparable output

Easy benchmarking

📊 Performance Consideration

For 4-hour video:

Whisper large-v3 → 1–2× real-time on good GPU

LLaMA 8B summarization → slow on CPU

Requires 16–32GB RAM minimum

 Side-by-Side Comparison
Factor	Cloud	Hybrid	Offline
Accuracy	⭐⭐⭐⭐	⭐⭐⭐⭐	⭐⭐⭐
Cost	High	Medium	Low (after infra)
Privacy	Low	Medium	High
Setup	Easy	Moderate	Complex
Batch reliability	High	High	Medium
Hallucination control	Good	Very Good	Harder
Long-term scalability	API-bound	Flexible	Hardware-bound
 Recommended Approach

For practical deployment:

👉 Hybrid Approach

Because:

Scales

Lower hallucination risk

Controlled cost

Local heavy lifting

Clean review workflow

Clear asset alignment

📘 Markdown Template (Deterministic)
# Video Summary

## Metadata
- Filename:
- Duration:
- Processed:

## High-Level Summary

(150–250 words)

## Key Highlights

### [00:15:32 – 00:22:10] Title
Description
Clip: clips/HL_001.mp4
Screenshot: screenshots/HL_001.jpg

## Key Takeaways

- Bullet
- Bullet
- Bullet


Readable in 5–10 minutes.

🚀 Final Architecture Recommendation (Production-Grade)
Orchestrator (Batch Runner)
   ├── Transcription Engine
   ├── Chunk Processor
   ├── LLM Highlight Engine
   ├── Validation Layer
   ├── Asset Extractor
   ├── Markdown Builder
   └── Batch Report Generator


Add:

Logging

Retry system

Confidence scoring

JSON validation before clip cutting

## Problem 2: **Zero-Shot Prompt to generate 3 LinkedIn Post**

Design a **single zero-shot prompt** that takes a user’s persona configuration + a topic and generates **3 LinkedIn post drafts** in **3 distinct styles**, each aligned to the user’s voice and constraints. The output must be structured so the app can: show 3 drafts to the user. Assume we are consuming **OpenAI API / Gemini API** with **one prompt call** (no fine-tuning). Your prompt must reliably produce valid, structured output. [READ MORE ABOUT THE PROJECT](./linkedin-automation.md)

**TASK:** Write a prompt that can work.

### Your Solution for problem 2:

You need to put your solution here.

## Problem 3: **Smart DOCX Template → Bulk DOCX/PDF Generator (Proposal + Prompt)**

Users have many Word documents that act like templates (offer letters, certificates, invoices, contracts). They repeatedly change only a few fields (name, date, amount, address, role, etc.). Doing this manually is slow and error-prone, especially in bulk. [READ MORE ABOUT THE PROJECT](./docs-template-output-generation.md)

We want a system that:

1. Converts an uploaded **DOCX** into a reusable **template** by identifying editable fields.
2. Supports **single generation** (form-fill → DOCX/PDF download).
3. Supports **bulk generation** via **Excel/Google Sheet** rows.

### **Task (No coding)**

Submit a **proposal** for building this system using GenAI (OpenAI/Gemini) for “template field detection” and “field schema generation”. We want a practical design, not code.

### Your Solution for problem 3:

You need to put your solution here.

## Problem 4: Architecture Proposal for 5-Min Character Video Series Generator

We want to build a system that helps a user create a short video series (around **5 minutes per episode**) using predefined characters. The user defines characters (image + personality) and relationships once, then provides a short story/situation for an episode. The system outputs a complete episode package and optionally a final video. [READ MORE ABOUT THE PROJECT](./char-based-video-generation.md)

### **Task**

Create a **small, clear architecture proposal** (no code, no prompts) describing how you would design and build this system.

### Your Solution for problem 4:

You need to put your solution here.
