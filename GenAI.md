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
1) Online / Cloud-Based (Already Available Solutions)
Architecture (practical)
	•Upload video (or extracted audio) to a cloud service.
	•Cloud performs transcription and summarization.
	•Export summary (and any available chapters/highlights).
	•Separately (if needed) generate clips/screenshots using returned timestamps.
Strengths
	•Fast to start, minimal engineering.
	•Typically strong summarization quality.
Risks / Tradeoffs (important for this project)
	•Uploading large videos is time-consuming and brittle for batch folders.
	•Privacy/compliance risk (video leaves local environment).
	•Many services won’t produce your exact artifact package (clips + screenshots + 	markdown linking) out-of-the-box.
	•Timestamp alignment may be imperfect unless you still do local extraction using 	timestamps.
When I would choose this
	•Non-sensitive content, low volume, need fastest “first results.”
	•The business is okay with cloud handling the raw video.

2) Hybrid Build: Local Media Processing + Cloud LLM API 
Why this fits your chosen environment (small office server, GPU possible)
	•Keeps heavy media work local (clips/screenshots stay accurate).
	•Allows one cloud call for “intelligence” (highlights, takeaways) using 	transcript + timestamps.
	•Scales well in batch mode with predictable outputs.
Architecture (end-to-end)
Local (deterministic pipeline):
	1.Scan folder + validate video
		•read duration + basic metadata
		•mark corrupted/unreadable files
	2.Extract audio + generate time index
		•ensures every transcript chunk maps to an exact time range
	3.Transcribe
		•can be local (preferred for privacy/cost control) or cloud
	4.Chunk transcript
		•chunk into time windows (e.g., 60–120 seconds) with {start_ts, end_ts, 		text}
	5.LLM call (cloud) for structured “highlight plan”
		•LLM receives only transcript chunks + constraints
		•returns strict JSON: summary + highlight segments + action items
	6.Local asset extraction
		•cut highlight clips using JSON {start_ts, end_ts}
		•extract screenshots at JSON {timestamp}
	7.Generate Summary.md
		•render markdown with links to assets
	8.Write logs + batch report
		•per-video and overall run report
Robust JSON schema (critical for reliability)
This is what the LLM must output (single response), so the app can reliably generate assets and Summary.md:
{
  "video": {
    "filename": "string",
    "duration_seconds": 0,
    "processed_at_iso": "YYYY-MM-DDTHH:MM:SSZ"
  },
  "summary": {
    "high_level": "string (max ~150-250 words)",
    "read_time_minutes_target": 5
  },
  "highlights": [
    {
      "id": "H001",
      "title": "string",
      "why_it_matters": "string",
      "start_ts": "HH:MM:SS",
      "end_ts": "HH:MM:SS",
      "confidence": 0.0,
      "screenshot_ts": ["HH:MM:SS"]
    }
  ],
  "takeaways": [
    { "type": "takeaway|action_item", "text": "string", "owner": "string|null", "due": "YYYY-MM-DD|null" }
  ],
  "notes": {
    "ambiguities_or_missing_context": ["string"],
    "assumptions_used": ["string"]
  }
}
Validation rules (non-negotiable):
	•start_ts < end_ts
	•timestamps must fall within duration
	•confidence in [0,1]
	•max highlights count (example: 8–15) to keep Summary.md readable
Handling ambiguity + user review flow (rubric)
Some videos are ambiguous (multiple topics, unclear goals). The system should:
	1)Force the LLM to list:
		•ambiguities_or_missing_context
		•assumptions_used
	2)Provide a review mode:
		•generate assets for top N highlights by confidence
		•include “Additional candidate highlights” in markdown (not clipped yet)
		•user can approve more highlights later (re-run only clipping/screenshot 		step)
This keeps the first pass fast and avoids wrong “best highlights.”
Bulk generation thinking (errors, naming, report)
Batch processing principles:
	1)Each video runs independently (failure doesn’t stop batch).
	2)Store intermediate artifacts:
		o	transcript JSON with timestamps
		o	LLM JSON output
	3)Retries:
		•if LLM JSON invalid → retry once with “repair” instruction (still single 		final JSON saved)
		•if clip extraction fails → mark highlight as failed but continue others
	4)Naming conventions (predictable + collision-safe):
		•clips: H001_00h12m03s_00h13m15s.mp4
		•screenshots: H001_00h12m08s.png
	5)Reporting:
		•per video: run_log.json (status, errors, counts, timings)
		•batch: batch_report.json (processed/failed/partial + reasons)
Tradeoffs
•More engineering than a pure cloud tool.
•Some cloud cost (LLM call), but reduced by chunking and structured output.

3) Fully Offline: Local Transcription + Local LLM + Local Pipeline
	1)Architecture
	Same pipeline as Hybrid, except the “LLM call” runs on a locally hosted model, 	and transcription is also local.
	2)Strengths
		•Highest privacy (no transcript leaves server).
		•Predictable ongoing cost (mainly compute).
	3)Tradeoffs
		•More operational work (model hosting, monitoring).
		•Output quality and consistency depends on local model capability; requires 		careful schema enforcement and retries.
	4)When I would choose this
		•Highly sensitive content where even transcript cannot leave the machine.
		•Very high volume where cloud LLM costs become large.

Suggested choice out of the above three
In practice, a hybrid approach (local media processing combined with cloud-based LLM summarization) often provides a good balance between output quality, scalability, and operational control. However, the final choice depends on privacy constraints, available compute resources, and cost considerations, and all three approaches described above can be valid depending on these constraints.



## Problem 2: **Zero-Shot Prompt to generate 3 LinkedIn Post**

Design a **single zero-shot prompt** that takes a user’s persona configuration + a topic and generates **3 LinkedIn post drafts** in **3 distinct styles**, each aligned to the user’s voice and constraints. The output must be structured so the app can: show 3 drafts to the user. Assume we are consuming **OpenAI API / Gemini API** with **one prompt call** (no fine-tuning). Your prompt must reliably produce valid, structured output. [READ MORE ABOUT THE PROJECT](./linkedin-automation.md)

**TASK:** Write a prompt that can work.

### Your Solution for problem 2:
You are an expert LinkedIn ghostwriter and compliance-focused editor.

TASK  
Given a user Persona Configuration + a Topic (and optional context), generate exactly 3 LinkedIn-ready post drafts in 3 distinct styles while preserving the user’s voice and obeying all constraints. The user must approve one draft before publishing or scheduling.

IMPORTANT OUTPUT RULES (MUST FOLLOW)  
- Output MUST be valid JSON only. No markdown. No code fences. No extra keys. No trailing commas.  
- Use UTF-8 plain text. Escape characters as needed for valid JSON.  
- If required info is missing or ambiguous, do NOT guess. Instead, list clarification_questions and proceed with safe assumptions in assumptions_used.  
- Do not fabricate personal facts, metrics, employers, credentials, or results. Only use what is in persona/topic input.  
- Avoid spammy engagement bait, prohibited claims, and repetitive phrasing.

INPUTS (JSON)  
persona_config: <<PASTE_PERSONA_JSON_HERE>>  
request: <<PASTE_REQUEST_JSON_HERE>>

Where request JSON has:  
{
  "topic": "string",
  "optional_context": "string|null",
  "audience": "string|null",
  "goal": "string|null",
  "posting_preference": {
    "mode": "immediate|scheduled",
    "scheduled_datetime_local": "YYYY-MM-DDTHH:MM:SS|null",
    "timezone": "IANA_TZ|null"
  }
}

STYLE VARIANTS (must be meaningfully different)  
1) "concise_insight": brief, sharp viewpoint + 1–2 key takeaways  
2) "story_based": short narrative (setup → tension → lesson), still professional  
3) "actionable_checklist": steps/framework/checklist format  

LINKEDIN DRAFT RULES  
- Each draft must be LinkedIn-ready, written in the user’s voice, and comply with persona do/don’t guidelines.  
- Keep each draft between 900 and 1,800 characters unless persona_config explicitly specifies otherwise.  
- No hashtags unless persona_config explicitly allows them; if allowed, use max 3 relevant hashtags.  
- No external links unless persona_config explicitly allows them.  
- Include a non-spammy CTA only if persona_config allows CTAs; otherwise omit.  
- Do not mention “AI”, “ChatGPT”, “prompt”, or “as an AI”.

RELIABILITY / SELF-CHECK (internal, but reflected in JSON fields)  
- For each draft provide: a constraints_check object with booleans for key constraints and a short notes string.  
- Provide a similarity_score_between_drafts (0–1) where lower is more distinct. Target <= 0.35; if higher, rewrite to increase distinctness.

OUTPUT JSON SCHEMA (MUST MATCH EXACTLY)  
{
  "version": "1.0",
  "input_echo": {
    "topic": "string",
    "audience": "string|null",
    "goal": "string|null",
    "posting_preference": {
      "mode": "immediate|scheduled",
      "scheduled_datetime_local": "string|null",
      "timezone": "string|null"
    }
  },
  "clarification_questions": [
    {
      "id": "Q1",
      "question": "string",
      "why_needed": "string",
      "suggested_options": ["string"]
    }
  ],
  "assumptions_used": ["string"],
  "drafts": [
    {
      "draft_id": "D1",
      "style": "concise_insight",
      "title": "string",
      "post_text": "string",
      "key_points": ["string"],
      "tone_tags": ["string"],
      "compliance_flags": ["string"],
      "constraints_check": {
        "follows_persona_voice": true,
        "obeys_dos_donts": true,
        "no_unverifiable_claims": true,
        "not_spammy": true,
        "distinct_from_others": true,
        "notes": "string"
      }
    },
    {
      "draft_id": "D2",
      "style": "story_based",
      "title": "string",
      "post_text": "string",
      "key_points": ["string"],
      "tone_tags": ["string"],
      "compliance_flags": ["string"],
      "constraints_check": {
        "follows_persona_voice": true,
        "obeys_dos_donts": true,
        "no_unverifiable_claims": true,
        "not_spammy": true,
        "distinct_from_others": true,
        "notes": "string"
      }
    },
    {
      "draft_id": "D3",
      "style": "actionable_checklist",
      "title": "string",
      "post_text": "string",
      "key_points": ["string"],
      "tone_tags": ["string"],
      "compliance_flags": ["string"],
      "constraints_check": {
        "follows_persona_voice": true,
        "obeys_dos_donts": true,
        "no_unverifiable_claims": true,
        "not_spammy": true,
        "distinct_from_others": true,
        "notes": "string"
      }
    }
  ],
  "similarity_score_between_drafts": 0.0,
  "recommended_next_step": {
    "user_action": "select_draft_and_approve",
    "approval_required": true,
    "publishing_note": "Publishing must only occur after explicit user approval. If scheduled, store timezone and retry on failure with backoff."
  }
}

COMPLIANCE_FLAGS GUIDANCE  
- Use only when applicable, e.g.:  
  "needs_user_fact_check",  
  "missing_target_audience",  
  "persona_conflict_detected",  
  "cta_disallowed_by_persona",  
  "hashtags_disallowed_by_persona",  
  "link_disallowed_by_persona"

NOW DO THE WORK  
1) Read persona_config and request.  
2) If persona_config conflicts internally (e.g., “no CTA” but “always include CTA”), prefer the stricter constraint and flag persona_conflict_detected.  
3) Generate 3 drafts per the styles above, consistent with persona.  
4) Ensure all output matches the schema exactly, with valid JSON.  
5) If there are no clarification questions, set clarification_questions to []. Do not invent a question.  
6) If there are no assumptions, set assumptions_used to [].



## Problem 3: **Smart DOCX Template → Bulk DOCX/PDF Generator (Proposal + Prompt)**

Users have many Word documents that act like templates (offer letters, certificates, invoices, contracts). They repeatedly change only a few fields (name, date, amount, address, role, etc.). Doing this manually is slow and error-prone, especially in bulk. [READ MORE ABOUT THE PROJECT](./docs-template-output-generation.md)

We want a system that:

1. Converts an uploaded **DOCX** into a reusable **template** by identifying editable fields.
2. Supports **single generation** (form-fill → DOCX/PDF download).
3. Supports **bulk generation** via **Excel/Google Sheet** rows.

### **Task (No coding)**

Submit a **proposal** for building this system using GenAI (OpenAI/Gemini) for “template field detection” and “field schema generation”. We want a practical design, not code.

### Your Solution for problem 3:
Overview
The goal is to convert an uploaded DOCX into a reusable template by automatically detecting editable fields, generating a structured field schema using GenAI, and enabling both single-document generation and bulk generation via spreadsheet rows. The system should preserve the original document’s formatting while replacing only the variable fields.
1) Practical System Architecture
	A. Template Creation (DOCX → Reusable Template)
		1.	Upload & Pre-processing (deterministic):
			•Parse the DOCX into a structured document map (paragraphs, runs, 			tables, headers, footers).
			•Identify candidate variable regions such as blanks, placeholders, 			labeled values (e.g., “Name: ___”), and repeated entities.
			•This step is deterministic and does not rely on GenAI.
		2.	GenAI-assisted Field Detection & Schema Generation:
			•Provide the extracted document map to the GenAI model.
			•The model returns a strict JSON schema describing:
				•field keys (e.g., candidate_name, start_date, amount)
				•field types (text, date, currency, number, address, etc.)
				•required vs optional
				•example values
				•binding locations (where each field appears in the document)
				•confidence scores for each detected field/binding
		3.	User Review & Confirmation (ambiguity handling):
			•The UI presents detected fields with confidence scores and document 			locations.
			•Low-confidence or ambiguous fields are flagged for manual confirmation.
			•The user can rename fields, change types, mark required/optional, or 			remove incorrect fields.
			•Only after user confirmation is the template finalized and stored.
	B. Single Document Generation (Form-fill → DOCX/PDF):
		•	The confirmed schema auto-generates a form UI.
		•	The user fills values.
		•	The system validates required fields and type constraints.
		•	The output DOCX is rendered with replacements and optionally converted 			to PDF for download.
	C. Bulk Document Generation (Excel/Google Sheet):
		•	The system generates a sheet template with one column per schema field.
		•	The user uploads a filled sheet (or connects Google Sheets).
		•	Each row is validated independently and rendered into a DOCX/PDF output.
		•	Outputs are packaged (e.g., ZIP) for download.
2) Robust JSON Schema Design (Template Schema)
A strict JSON schema is stored per template to ensure reliable generation and bulk operations:
•Template metadata: template_id, template_name, source_docx_filename, created_at
•Fields:
	•key (stable identifier)
	•label (UI display name)
	•type (text, date, currency, number, address, etc.)
	•required (boolean)
	•example (string)
	•validation rules (e.g., min/max length, format hints)
	•bindings (document locations + confidence)
•Naming rule: default filename pattern for generated documents (e.g., 	{primary_name}_{template_name}_{date})
This schema enables:
	•consistent form generation,
	•reliable spreadsheet column mapping, and
	•predictable output file naming.
3) Handling Ambiguity + User Review Flow
	•The GenAI output includes confidence scores for detected fields and their 	bindings.
	•Any field or binding below a confidence threshold is marked “needs review.”
	•The template cannot be finalized until the user confirms or edits flagged 	items.
	•A preview mode shows how sample values would appear in the DOCX before saving 	the template, ensuring formatting is preserved and replacements are correct.
4) Bulk Generation Thinking (Errors, Naming, Report)
	•Row-level isolation: Each spreadsheet row is processed independently so one 	failure does not block the entire batch.
	•Validation: Missing required fields or invalid formats cause that row to fail 	with a clear error message.
	•Predictable naming: Output files follow a deterministic naming rule derived 	from the schema; collisions are resolved by appending an index.
	•Batch report: The system produces a structured job report summarizing total 	rows, success count, failure count, and per-row error reasons. This enables 	auditing and retrying only failed rows.
5) Tradeoffs & Practical Considerations
•Strengths:
	•Automates template creation and bulk generation with minimal manual setup.
	•GenAI accelerates field detection while deterministic parsing ensures 	formatting preservation.
	•User review prevents silent schema mistakes.
•Limitations:
	•Field detection may not be perfect for highly complex DOCX layouts, hence the 	need for review.
	•Initial setup requires careful schema confirmation to ensure high-quality bulk 	outputs.

This proposal provides a practical, production-oriented design that uses GenAI specifically for field detection and schema generation, ensures reliability through strict JSON schemas, manages ambiguity with user review, and supports robust bulk generation with error reporting and predictable naming, aligned with the stated evaluation criteria.



## Problem 4: Architecture Proposal for 5-Min Character Video Series Generator

We want to build a system that helps a user create a short video series (around **5 minutes per episode**) using predefined characters. The user defines characters (image + personality) and relationships once, then provides a short story/situation for an episode. The system outputs a complete episode package and optionally a final video. [READ MORE ABOUT THE PROJECT](./char-based-video-generation.md)

### **Task**

Create a **small, clear architecture proposal** (no code, no prompts) describing how you would design and build this system.

### Your Solution for problem 4:
1) Core concept: “Series Bible” + “Episode Job”
	The system has two persistent layers:
	1.Series Bible (created once, reused)
		•Characters: reference images, personality/speaking rules, visual 		consistency 	notes
		•Relationships: structured relationship graph (mentor/rival/parent-child, 		etc.)
		•Optional world/setting rules (tone, location, recurring themes) 
		char-based-video-generation
	2.Episode Job (created per episode)
		•Episode prompt (situation/conflict/outcome)
		•Selected cast (subset of characters)
		•Episode style goal (comedy/drama/etc.), language, narration vs dialogue, 		format (9:16/16:9)
		•Duration target ≈ 5 minutes 
2) Practical architecture (high level components)
	A.Data + Versioning
		•Store Series Bible as versioned JSON (so character updates don’t break 		older episodes).
		•Store each Episode Job as versioned JSON plus generated artifacts (script, 		shot list, prompts, audio plan, render status). 
	B.Generation Pipeline (per episode)
		1.Input validation
			•Ensure selected characters exist, relationships are defined, required 			preferences present.
		2.Story planner
			•Convert the short episode situation into a structured outline: scenes, 			beats, estimated time per scene to hit ~5 minutes.
		3.Script generator
			•Produce scene-by-scene script with dialogues and optional narration.
			•Enforce character behavior and relationship rules (e.g., rival tone, 			mentor language).
		4.Shot list / storyboard planner (preferred)
			•For each scene: shot description, camera intent, required characters, 			background needs.
		5.Asset plan
			•Visual asset list per shot: background + character appearances (using 			reference images / consistency notes).
			•Audio plan: per-line voice lines by character, narration, music cues.
		6.(Optional) Rendering orchestrator
			•If user chooses “render final video”: generate visuals, synthesize 			voices, assemble timeline to ~5 minutes, output final video.
			•Otherwise output the “production-ready package” (script + breakdown + 			prompts + voice lines). 
This matches the required outputs: script, breakdown, prompts/assets, audio plan, optional final render. 
3) Robust JSON schema design (for reliability)
Use strict schemas so every stage is machine-checkable.
Series Bible (stored once)
	•series_id, title, style_rules, world_rules
	•characters[]: character_id, name, reference_images[], visual_notes, 	personality_rules, speaking_style, voice_profile_ref
	•relationships[]: from_character_id, to_character_id, type, rules, do_not_do
Episode Package (generated per episode)
	•episode_id, series_id, episode_prompt, cast[], target_duration_seconds, format, 	language
	•outline[]: scene beats + estimated duration
	•script[]: scene-by-scene lines (speaker, text, emotion, duration estimate)
	•shot_list[]: shot_id, scene_id, description, required_assets
	•assets[]: asset_id, type (bg/character/prop), reference/prompt, 	linked_scene_or_shot
	•audio_plan[]: line_id, speaker, voice_profile_ref, text, timing, music cues
	•render_plan: timeline segments, total duration, status
Strict schemas reduce regeneration errors and allow partial regeneration (e.g., regenerate only shot list). 
4) Handling ambiguity + user review flow (iteration-friendly)
Ambiguity is expected (tone, ending, pacing). Add explicit checkpoints:
	1.Outline Review
		•After story planner, show outline + estimated timings.
		•User can adjust: number of scenes, ending, tone intensity, 		narration/dialogue ratio.
	2.Script Review
		•User approves or edits dialogue style, character behavior adherence.
	3.Selective Regeneration
		a)User can regenerate only one layer:
			•“Regenerate Scene 3 dialogue”
			•“Regenerate shot list only”
			•“Swap character A with character B”
			This supports easy iteration as required. 
5) Bulk generation thinking (episodes at scale) + reliability
Even if users generate many episodes:
	•Treat each episode as a job with independent status: 		queued/running/needs_review/failed/completed.
	•Stage-level outputs are saved so failures don’t lose progress (outline saved 	even if rendering fails).
	•Standard naming and packaging:
		a)output/<series_id>/<episode_id>/script.json, script.md, shot_list.json, 		audio_plan.json, assets/, final.mp4 (if rendered)
	•Job report:
		a)scene count, duration estimate vs actual, missing assets, failures with reasons.
This supports repeatable episode production while keeping characters consistent across episodes.