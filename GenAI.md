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

Below is the complete content consolidated into **one single Markdown file**, clean and ready to paste directly into your `.md` file.

---

```markdown
# Automated Long-Video Summary System Proposal

## 1. Problem Statement

We have a local folder containing long videos (3–4 hours each, 200MB+ per file). Watching them end-to-end to extract useful insights is time-consuming and does not scale.

We need a fully automated batch system that processes every video in a folder and generates a structured, concise “summary package” that can be consumed in 5–10 minutes.

For each input video, the system must generate:

- `Summary.md` (structured Markdown)
- Highlight clips aligned with timestamps
- Screenshots tied to highlights
- Predictable per-video folder structure

---

## 2. Required Output Structure

```

input/videos/
video_name1.mp4
video_name2.mp4

output/
video_name1/
Summary.md
clips/
screenshots/
video_name2/
Summary.md
clips/
screenshots/

```

---

## 3. System Requirements

- Must handle large files (200MB+)
- Must process long-duration videos (3–4 hours)
- Must run in batch mode (no manual work per video)
- Highlights must contain accurate timestamps
- Generated clips and screenshots must match timestamps exactly
- Summary must be readable in 5–10 minutes
- Output must be structured and predictable

---

# Approach 1: Online / Cloud-Based Solution

## Architecture

```

Local Folder
↓
Upload to Cloud Storage
↓
Cloud Transcription API
↓
Cloud LLM Summarization + Highlight Detection
↓
Timestamp Extraction (JSON Output)
↓
Local FFmpeg Processing
↓
Generate Summary.md + Assets

````

## Strengths

- Very high transcription accuracy
- Built-in diarization and segmentation
- Easy scaling
- Production-ready infrastructure

## Weaknesses

- Higher cost for long videos
- Upload time for large files
- Data privacy concerns
- API rate limits for batch workloads

---

## JSON Output Schema (Cloud Contract)

```json
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
````

### Why This Schema Is Robust

* Explicit timestamps
* Confidence scoring
* Structured ambiguity reporting
* Deterministic output format
* Easy validation

---

## Prompt Strategy (Zero-Shot, Low Hallucination Risk)

System instructions:

* Use only provided transcript segments
* Do not invent information
* Do not infer missing context
* Output strictly valid JSON
* Mark uncertain sections explicitly
* Do not output any text outside JSON

This ensures grounded outputs and reduces hallucination risk.

---

## Best Fit

* Enterprise environment
* Accuracy priority
* Fast deployment required

---

# Approach 2: Hybrid (Local Processing + Cloud LLM APIs)

This is the most practical and balanced architecture.

## Architecture

```
Batch Folder
   ↓
Local Video Processing (FFmpeg)
   ↓
Local Transcription (Whisper small/medium)
   ↓
Transcript Chunking (5–10 minute segments)
   ↓
Cloud LLM (Summarize + Highlight Selection)
   ↓
Validated JSON Output
   ↓
Local Clip + Screenshot Extraction
   ↓
Markdown Builder
   ↓
Batch Report Generator
```

---

## Key Design Principle

Never send the full 4-hour transcript to the LLM at once.

Instead:

1. Split transcript into 5–10 minute chunks
2. Summarize each chunk independently
3. Aggregate chunk summaries
4. Run final compression pass
5. Select highlights with scoring

Benefits:

* Prevents token overflow
* Reduces cost
* Improves consistency
* Minimizes hallucination risk

---

## Production-Ready JSON Schema

```json
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
```

### Schema Strengths

* Versioned
* Numeric timestamps (reduces parsing errors)
* Model traceability
* Confidence scoring
* Review flag
* Replaceable backend compatibility

---

## Ambiguity Handling and Review Flow

LLM rules:

* Flag highlights with confidence < 0.6
* Mark unclear audio sections
* Avoid summarizing inaudible segments

Review logic:

```
If review_required = true:
    User reviews flagged highlights only
    Approve / Edit / Remove
```

This reduces manual workload while preserving reliability.

---

## Batch Error Handling

Per video:

* Transcription failure → Log and skip
* LLM timeout → Retry up to 2 times
* Invalid JSON → Trigger repair prompt
* Clip mismatch → Re-extract using timestamps

Global report:

```json
{
  "total_videos": 20,
  "successful": 18,
  "failed": 2,
  "average_processing_time_min": 14.2
}
```

---

## Strengths

* Balanced cost
* Strong hallucination control
* Better privacy than full cloud
* Scalable batch operation
* Production-ready reliability

## Weaknesses

* API dependency remains
* Requires orchestration layer

---

# Approach 3: Fully Offline (Open Source Only)

## Architecture

```
Video Folder
   ↓
Local GPU Transcription (Whisper large)
   ↓
Semantic Segmentation (Embeddings + Clustering)
   ↓
Local LLM Summarization
   ↓
Highlight Scoring
   ↓
Clip + Screenshot Extraction
   ↓
Markdown Assembly
   ↓
Batch Reporting
```

---

## Characteristics

* Fully local processing
* No external APIs
* Requires strong GPU + 16–32GB RAM

---

## Challenges

* Lower summarization quality
* Higher hallucination risk
* Slower processing
* Requires advanced prompt tuning
* Hardware-bound scaling

---

## Advantages

* Full data privacy
* No API rate limits
* One-time infrastructure cost
* Unlimited internal processing

---

# Comparison Table

| Factor                | Cloud     | Hybrid    | Offline           |
| --------------------- | --------- | --------- | ----------------- |
| Accuracy              | High      | High      | Medium            |
| Cost                  | High      | Medium    | Low (after setup) |
| Privacy               | Low       | Medium    | High              |
| Setup Complexity      | Low       | Medium    | High              |
| Batch Reliability     | High      | High      | Medium            |
| Hallucination Control | Good      | Very Good | Harder            |
| Scalability           | API-bound | Flexible  | Hardware-bound    |

---

# Recommended Approach

Hybrid Architecture

Reasons:

* Balanced accuracy and cost
* Strong hallucination control
* Efficient review workflow
* Reduced API token usage
* Production-grade reliability
* Replaceable backend flexibility

---

# Deterministic Markdown Template

```markdown
# Video Summary

## Metadata
- Filename:
- Duration:
- Processed Date:

## High-Level Summary
(150–250 words)

## Key Highlights

### [00:15:32 – 00:22:10] Highlight Title
Description

Clip: clips/HL_001.mp4  
Screenshot: screenshots/HL_001.jpg  

## Key Takeaways

- Actionable point 1
- Actionable point 2
- Actionable point 3
```

---

# Final Production Architecture

```
Batch Orchestrator
   ├── Transcription Engine
   ├── Chunk Processor
   ├── LLM Highlight Engine
   ├── JSON Validator
   ├── Asset Extractor (FFmpeg)
   ├── Markdown Builder
   ├── Logging System
   └── Batch Report Generator
```

---


## Robust JSON Schema Design

* Versioning
* Confidence scoring
* Numeric timestamps
* Traceable model usage
* Review flag

## Prompt Quality

* Zero-shot grounded in transcript
* Strict JSON-only output
* Explicit uncertainty handling
* Minimal hallucination risk

## Handling of Ambiguity + User Review Flow

* Confidence thresholds
* Flagged highlights
* Human-in-the-loop validation

## Bulk Generation Thinking

* Retry logic
* JSON validation
* Error logging
* Global batch report
* Predictable folder structure



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
