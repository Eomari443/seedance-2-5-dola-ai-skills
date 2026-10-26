---
name: seedance-2-5-dola-ai-30s
description: Seedance 2.5 skill for Dola AI 30-second — specialized Seedance 2.5 video generation with hard-locked single-segment 30-second output, bulk TXT/Notepad prompt queueing, resilient skip-and-continue failure handling, end-of-run retry analysis, and an opt-in 15-minute Dola AI 30-Second Mode watchdog. Normal video generation runs on autopilot without confirmation; the cron/watchdog stays OFF until the user manually says “dola ai 30 second mode activate”. Always generates each video as one complete call and preserves the original Seedance 2.5 generation rules.
---

# Seedance 2.5 skill for Dola AI 30-second — Seedance 2.5 — 30s Hard Lock

## Hard Lock (non-negotiable, overrides everything below)

- **Duration is locked at 30 seconds.** Always pass `duration: 30`.
- **One single continuous segment.** One tool call = one complete 30-second video. This is the only delivery path.
- **Model is locked to Seedance 2.5.** Always pass `model_version: "seedance_2.5"` explicitly in every call.
- **Skill identity:** This skill is **Seedance 2.5 skill for Dola AI 30-second**.
- **Bulk runs must not stop because one prompt fails.** A failed item is deferred, checkpointed, and the next pending prompt is processed.
- **No infinite retries.** Failed prompts receive at most one sanitized retry after the main queue finishes; a second failure is discarded.
- **Autopilot generation is the default.** Never ask for `go`, `confirm`, `yes`, `start`, or any other approval before generating a video.
- **Dola AI 30-Second Mode is OFF by default.** Never register, start, or run the cron/watchdog until the user manually activates it by saying **“dola ai 30 second mode activate”** (case-insensitive; a clearly equivalent direct activation command is also valid).

### Anti-spreading rules — never do any of these:

- Never split into two 15-second clips or any multi-part sequence.
- Never propose, offer, suggest, or mention a two-segment / two-clip / chaptered / multi-part delivery.
- Never offer a "single 15s compressed" alternative or any shorter-compressed fallback.
- Never present a "platform limitation", "tool cap", "runtime doesn't support 30s" disclaimer, Plan B, fallback, or workaround.
- Never ask the user to choose between split options, segment counts, or duration fallbacks.
- Never use the words "split", "segments", "clips", "chapters", "15s", "fallback", or "Plan B" in pre-generation status text.

### Special case — user explicitly requests under 30 seconds:

If the user explicitly requests a duration shorter than 30 seconds, still use **one single call**, with `duration` set to the requested length (within the 5–30s range supported by the tool). No splitting. This is the only exception to the 30-second default, and it only applies when the user themselves specifies a shorter duration.

### Longer than 30 seconds:

If the user requests longer than 30 seconds: still **one single 30-second call** compressing the narrative arc. No splitting.

### Maximum videos per reply:

**2 (hard cap)** — each is its own complete single-segment video.

---

## Core Capability

This skill generates videos using the Seedance 2.5 model (`model_version: "seedance_2.5"`). Seedance 2.5 produces complete 30-second videos in a single generation call. Always generate one continuous 30-second segment. A single 30-second call is the only path.

## Model Specification

- **Model:** Seedance 2.5 (`seedance_2.5`) — always specify explicitly.
- **Duration:** 30 seconds, single continuous segment.
- **Supported ratios:** 21:9, 16:9, 4:3, 1:1, 3:4, 9:16.
- **Maximum videos per reply:** 2 (hard cap).
- **Input modes:** text-to-video (`text_to_video`), image-to-video (`image_to_video`).

---

## Workflow

### 1. Planning Phase

Design the concept as **one cohesive 30-second narrative arc** covering beginning, middle, and end. Structure the prompt with clear pacing and shot progression:

- **Opening (0–5s):** establish subject, scene, atmosphere.
- **Development (5–20s):** main action, escalation, camera movement.
- **Resolution (20–30s):** climax, payoff, closing shot.

Include all visual details: subjects and their appearance, wardrobe, props, action beats, scene/environment, style, lighting, color grade, camera movement, mood, and pacing.

### 2. Autopilot Generation Gate

Generation is **automatic by default**. The user's prompt, pasted prompt list, or uploaded TXT file is sufficient authorization to begin generation.

- Never ask for `go`, `confirm`, `yes`, `start`, or any equivalent confirmation.
- Do not present a blocking confirmation summary before calling the generation tool.
- If a non-essential parameter is missing, infer a reasonable value from the prompt or current run context instead of asking.
- If ratio is not specified and cannot be inferred from context, use `9:16`.
- Preserve all hard-locked Seedance 2.5, duration, safety, prompt-quality, and delivery rules.
- **Dola AI 30-Second Mode and cron state do not affect this normal autopilot generation behavior.** Videos must continue to generate normally while Dola AI 30-Second Mode is OFF.

### 3. Generation Phase

As soon as a valid generation request is available:

Call `text_to_video` (or `image_to_video` if reference images are used) with:

- `model_version: "seedance_2.5"`
- `duration: 30`
- `ratio:` resolved ratio
- `prompt:` comprehensive full 30-second description
- `image_reference_url_list:` only URLs returned by `FileBatchUpload` (for image-to-video)

Deliver the resulting video via `NotifyHuman`, including a renderable video asset (`video_url`, `output_url`, `url`, `result_url`, or local path).

For a single generation item, that is the complete generation flow. **One call. One 30-second video. No splitting.** Bulk runs repeat this same per-item flow through the persistent queue defined below.

---

## Bulk TXT / Notepad Autopilot Workflow

Use this workflow when the user pastes many prompts from Notepad or provides a `.txt` file containing multiple prompts. The original Seedance 2.5 generation rules still apply to every individual video.

### Bulk Input Parsing

1. Read the complete text input before starting generation.
2. Extract prompts in their original order. Prefer explicit numbering, headings, separators, or blank-line boundaries when present.
3. Do not shorten, summarize, or creatively rewrite successful source prompts unless the normal 30-second prompt-crafting rules require formatting for the generation call.
4. Assign every prompt a stable `prompt_id` and keep the original prompt text immutable as `source_prompt`.
5. Build a persistent queue with these states: `pending`, `generating`, `completed`, `deferred_retry`, `discarded`.
6. Save/checkpoint queue state after every status change so the run can resume after a crash, empty session, tool interruption, or watchdog restart without duplicating completed videos.

### Bulk Autopilot Start

A pasted multi-prompt block or uploaded `.txt` file starts the bulk queue automatically. No initial confirmation and no per-item confirmation is required. Parse the queue, checkpoint it, and begin processing immediately.

### Continuous Queue Processing

- Process the queue in original order.
- Respect the existing **2 videos per reply/execution-cycle hard cap** by processing at most 2 generation items in one cycle, then automatically continue from the next `pending` item in the next cycle until the queue is exhausted. The queue itself must not be considered complete merely because one cycle reached the cap.
- Before each generation call, mark the item `generating` and checkpoint it.
- On success, mark it `completed`, store the returned renderable asset/session metadata, checkpoint, and deliver the video through `NotifyHuman`.
- Then immediately continue with the next `pending` prompt.

### Error Isolation — Skip, Record, Continue

If any individual prompt fails for any reason, **do not stop the bulk run**. This includes, but is not limited to:

- safety/moderation rejection or a flagged/violating term,
- invalid or unsupported prompt content,
- tool/API exception, timeout, temporary generation failure,
- runtime crash or interrupted call,
- missing, blank, or empty session ID,
- missing output URL / non-renderable result,
- any other error that prevents a valid video asset from being returned.

For the failed item:

1. Capture the error type/message and the original `source_prompt`.
2. Mark the item `deferred_retry`.
3. Do not retry it immediately.
4. Continue with the next `pending` prompt so one failure never blocks later videos.

### End-of-Main-Queue Retry Pass

Only after there are no `pending` or `generating` items left in the main queue:

1. Collect all `deferred_retry` items.
2. Analyze the recorded failure and the prompt itself to identify the smallest likely cause.
3. If the failure is safety-related, only retry when the **underlying requested video is allowed**. Never disguise, encode, or preserve disallowed intent merely to bypass a safety system.
4. For an otherwise allowed prompt, replace only the minimum problematic wording with neutral, policy-compliant wording while preserving the intended scene as closely as possible. Do not broadly rewrite the prompt when a narrow fix is sufficient.
5. For technical failures such as timeout, crash, empty session ID, or missing output, retry the original prompt unchanged unless the error itself indicates a prompt-format problem.
6. Retry each failed prompt **once only**.
7. If the retry succeeds, mark it `completed`, save the asset/session metadata, and deliver it through `NotifyHuman`.
8. If that retry fails for any reason, mark it `discarded`, record the final failure reason, and never automatically use that prompt again in the same run.

### Completion Condition

The bulk job is complete only when **every prompt** is either `completed` or `discarded`, with no items left in `pending`, `generating`, or `deferred_retry`. When complete, persist the final state and stop all automatic resume activity for that run.

---

## Dola AI 30-Second Mode — 15-Minute Watchdog / Cron Contract

The watchdog/cron is **manual opt-in only**. It must never activate merely because this skill loaded, a video was requested, or a bulk queue was created.

### Dola AI 30-Second Mode State

- **Default state:** `dola_ai_30_second_mode_active = false`.
- While `dola_ai_30_second_mode_active = false`, do **not** register, start, schedule, wake, or execute `seedance-2-5-dola-ai-30s-watchdog`.
- Normal Seedance 2.5 video generation and bulk queue processing still run in full autopilot while Dola AI 30-Second Mode is OFF.
- Activate Dola AI 30-Second Mode only after a direct user command such as **“dola ai 30 second mode activate”** / **“dola ai 30 second mode activate kar do”**. Treat this as explicit authorization to enable the watchdog.
- Once activated, persist `dola_ai_30_second_mode_active = true` and register/start the watchdog below. No extra confirmation is required.
- Do not infer Dola AI 30-Second Mode activation from phrases like “generate videos”, “continue”, “bulk run”, “autopilot”, or from the presence of pending prompts. Activation must be explicit.

### Watchdog Configuration (only while Dola AI 30-Second Mode is active)

- **Job name:** `seedance-2-5-dola-ai-30s-watchdog`
- **Cron schedule:** `*/15 * * * *` (every 15 minutes)
- **Scope:** active/incomplete bulk runs. If no bulk run is active, the watchdog may no-op while Dola AI 30-Second Mode remains armed.
- **Required state:** the watchdog must read the same persisted queue/checkpoint used by the bulk workflow.

Every watchdog execution, **only when `dola_ai_30_second_mode_active = true`**, must:

1. Load the active run state.
2. If there is no active bulk run, do nothing for that tick; do not create or invent work.
3. If the run is already complete (`completed`/`discarded` only), do not resume generation for that run.
4. If one or more prompts remain `pending`, `generating`, or `deferred_retry`, check whether generation has made progress since the previous watchdog check.
5. If generation is not actively progressing, send the active voice/agent/controller a resume instruction equivalent to: **"Continue Seedance 2.5 Dola AI 30-second video generation from the saved queue. Resume with the next pending prompt; do not restart completed items."**
6. The resumed worker must use the saved queue state and the Error Isolation rules above. It must not duplicate already completed videos.
7. Do not prematurely enter the retry pass while normal `pending` items still exist.
8. When the main queue is exhausted, allow the one-time end-of-main-queue retry pass, then finish the run when no retryable work remains.

> A Markdown skill can define this cron/watchdog contract, but the host runtime must actually support/register scheduled execution. The host must **not** register the scheduler at skill startup. Register the 15-minute schedule only after the user explicitly activates Dola AI 30-Second Mode.

## Parameter Resolution

Resolve these parameters automatically before generation:

- **Duration:** 30 seconds default (or custom shorter duration only if the user explicitly specifies one)
- **Ratio:** 21:9, 16:9, 4:3, 1:1, 3:4, 9:16; use the user's value when given, otherwise infer from context, otherwise default to `9:16`
- **Detailed content:** subject, action, scene, style, camera movement, mood, lighting, color palette, shot progression
- **Input type:** text-only or with reference images
- **Negative constraints:** anything to exclude

If the user provides only a brief concept, infer reasonable parameters and proceed automatically. Do not ask for confirmation and do not infer any split or fallback.

---

## Image-to-Video with Seedance 2.5

When using reference images:

1. Inspect all local image files with `Read` first (visual inspection is required before reasoning about a frame's content).
2. Upload local images via `FileBatchUpload` to obtain public URLs.
3. Use only the returned URLs in `image_reference_url_list`.
4. Describe in the prompt which visual characteristics must be preserved (subject, style, composition, lighting).

---

## Generation Rules

- **One call, one 30-second video** — never split into multiple clips. Never propose a fallback. Never mention two-segment delivery.
- **Never exceed 2 videos per reply** — hard cap.
- **Never ask for generation confirmation.** Resolve missing non-essential parameters automatically and proceed.
- Use `model_version: "seedance_2.5"` explicitly in every call.
- Use `duration: 30` for full-length generation.
- Deliver videos via `NotifyHuman` — include a renderable video asset.
- **Audio is visual-only** — preserve this generation assumption internally; do not block generation with a confirmation step.
- **Bulk TXT/Notepad runs:** never terminate the entire run because one item fails; defer that item and continue the queue.
- **Retry policy:** one end-of-run retry maximum per failed prompt; second failure = `discarded`.
- **Checkpointing:** persist queue state after every item transition so the watchdog can safely resume without duplicate generation.
- **Dola AI 30-Second Mode gate:** cron/watchdog remains OFF unless the user explicitly says **“dola ai 30 second mode activate”** (or a clearly equivalent direct activation command). Normal generation never depends on Dola AI 30-Second Mode.

---

## Prompt Crafting for 30-Second Videos

A strong 30-second prompt includes:

- **Opening hook (0–5s):** subject, scene, atmosphere established.
- **Development (5–20s):** main action, escalation, camera progression.
- **Resolution / payoff (20–30s):** climax, punchline, closing shot.
- **Technical specifics:** lighting, color palette, lens, camera style, pacing, frame-rate feel.
- **Atmosphere:** mood, tone, environmental details.
- **Negative constraints:** what to avoid (text, watermark, wrong palette, drift, etc.).

---

## Tool Call Pattern — Single 30-Second Segment (the only pattern)

### Text-to-video:

```
text_to_video(
  model_version: "seedance_2.5",
  duration: 30,
  ratio: "<resolved-ratio>",
  prompt: "<comprehensive 30-second narrative prompt>"
)
```

### Image-to-video (add reference URLs only from FileBatchUpload):

```
image_to_video(
  model_version: "seedance_2.5",
  duration: 30,
  ratio: "<resolved-ratio>",
  prompt: "<comprehensive 30-second narrative prompt>",
  image_reference_url_list: ["<url-from-FileBatchUpload>", ...]
)
```
