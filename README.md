# Seedance 2.5 Skills for Dola AI

Two OpenCode-compatible skills for Dola AI video generation with Seedance 2.5:

| Skill | Default output | Skill directory |
| --- | ---: | --- |
| `seedance-2-5-dola-ai-15s` | One 15-second video | [`skills/seedance-2-5-dola-ai-15s`](skills/seedance-2-5-dola-ai-15s) |
| `seedance-2-5-dola-ai-30s` | One 30-second video | [`skills/seedance-2-5-dola-ai-30s`](skills/seedance-2-5-dola-ai-30s) |

## Included behavior

Both skills define:

- Explicit Seedance 2.5 model selection.
- Single-call video generation with a hard-locked default duration.
- Text-to-video and image-to-video workflows.
- Automatic parameter resolution and generation without an approval prompt.
- Bulk TXT/Notepad prompt queues with persistent checkpoints.
- Skip-and-continue processing when an individual prompt fails.
- One end-of-queue retry for deferred items, followed by discard on a second failure.
- An opt-in 15-minute Dola AI watchdog that remains disabled until its activation command is given.
- Prompt-crafting guidance for opening, development, resolution, camera, lighting, mood, and negative constraints.

The complete requirements and operating rules are in each directory's `SKILL.md`.

## Install

Copy the skill directory you want into the project's local OpenCode skills directory:

```sh
cp -R skills/seedance-2-5-dola-ai-15s /path/to/project/.opencode/skills/
```

Or install the 30-second version:

```sh
cp -R skills/seedance-2-5-dola-ai-30s /path/to/project/.opencode/skills/
```

Install both directories if both durations are needed. Keep each directory name aligned with the `name` value in its frontmatter.

## Activation

Normal video generation is automatic according to the selected skill's rules. The watchdog is not enabled by loading a skill; it requires that skill's explicit activation command:

- 15-second skill: `dola ai 15 second mode activate`
- 30-second skill: `dola ai 30 second mode activate`

## Repository layout

```text
skills/
├── seedance-2-5-dola-ai-15s/
│   └── SKILL.md
└── seedance-2-5-dola-ai-30s/
    └── SKILL.md
```
