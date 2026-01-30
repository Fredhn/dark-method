# DARK-METHOD System Definition

This document defines **DARK-METHOD** as the governing system for dark (faceless) YouTube channel production. All agents and users must comply with these rules.

---

## System Identity

**DARK-METHOD** is a gated, incremental, agency-style workflow for creating animated, script-driven videos with no external content dependency.

- One agent active at a time  
- One artifact per agent  
- Explicit user approval between agents  
- No skipping steps  
- No cross-agent responsibilities  

---

## Project Folders (projects/)

- **Repository root** is the workspace root (e.g. `dark.channel.maker`). Open this folder in Cursor so rules load.
- **Each video project** has its own folder under **projects/** (e.g. `projects/my-first-video/`, `projects/how-it-works/`). Do not mix projects; all artifacts for one project (and per-video artifacts within it) live in that folder. This is the default and only supported location for project folders.
- **Current project folder** is stored in `dark-method/.current-project` (one line = project name/slug, no path). All agents read and write artifacts in **projects/{currentProjectFolder}/**, where `{currentProjectFolder}` is the value in `dark-method/.current-project`.
- **Onboarding** asks first: "Do you want to create a video for a **new project** or an **existing project**?"
  - **New project:** Onboarding creates the project folder at **projects/{slug}** (and channel-brief.md if scope = Repeatable channel system), then writes the slug to `dark-method/.current-project`.
  - **Existing project:** Onboarding lists the folders in **projects/** and lets the user select one. It writes the selected project slug to `dark-method/.current-project`. No new folder is created; the user is creating a **new video** (or new episode) for that existing project. Next step: run Producer to create the Episode Brief (or Production Brief) for this video.
- If `.current-project` is missing or empty, the user must run Onboarding first (new project or select existing project).
- **Artifact paths** are always **projects/{currentProjectFolder}/production-brief.md**, **projects/{currentProjectFolder}/script.md**, etc., where `{currentProjectFolder}` is the value in `dark-method/.current-project`.

---

## Channel vs Episode (Repeatable Channel System)

When the user selects **Repeatable channel system** at Onboarding, the framework segregates **channel-level** (general) information from **video/episode-level** (specific) information. This follows content-agency practice: one governing channel brief, then per-episode briefs and artifacts.

### Channel-level (general; applies to all episodes)

- Stored in **projects/{currentProjectFolder}/channel-brief.md** (created by Onboarding when scope = "Repeatable channel system").
- **Channel-level** includes: channel name, positioning/tagline, default target audience (sophistication, interests), default tone (e.g. dramatic, calm, ominous), format description (e.g. "animated roundup, 8–12 min per episode"), optional recurring structure (e.g. "hook → context → blocks → CTA"), optional branding guidelines.
- Agents must **not re-ask** channel-level information once it is in channel-brief.md. Downstream agents read channel-brief.md when present for consistency (tone, voice, format, branding).

### Episode-level (video-specific; applies to this video only)

- Stored in **production-brief.md** (Episode Brief when channel-brief exists), **script.md**, **visuals.md**, **audio.md**, **packaging.md**, **publish.md**. Optionally **research.md** (created by Script Architect when the theme is researchable).
- **Episode-level** includes: this episode/video goal, this episode target length, this episode topic/theme, this episode tone (or "per channel"), and all script, visual, audio, packaging, and publish artifacts for this video. When the theme is researchable (e.g. real events, league rounds), the user may provide **suggested sources for research** (via Producer or when Script Architect asks); Script Architect prioritizes those sources, fetches web data, writes **research.md** with facts and sources, and uses only that data in the script.
- When **channel-brief.md** exists: **production-brief.md** is the **Episode Brief** (references channel brief; contains episode goal, length, topic; inherits audience and default tone from channel brief). When **channel-brief.md** does not exist (Single video): **production-brief.md** is the full video brief.

### Behavior by scope

| Scope | channel-brief.md | production-brief.md |
|-------|------------------|---------------------|
| **Single video** | Not created | Full video brief (goal, audience, tone, length, boundaries, success criteria) |
| **Repeatable channel system** | Created by Onboarding with channel-level inputs | Episode Brief (this episode goal, length, topic; references channel brief; audience/tone from channel) |

---

## Global Agent Behavior Rules (Mandatory)

These rules apply to **every** agent file.

### 1. INPUT-FIRST RULE

The agent MUST explicitly request all required inputs before producing output.  
If required inputs are missing → **STOP and wait.**

### 2. NO ASSUMPTION RULE

Agents must not invent constraints, audience, tone, or tools.

### 3. SINGLE ARTIFACT RULE

Each agent outputs **one** primary artifact only.

### 4. APPROVAL GATE RULE

Every agent must request explicit approval using **exactly**:

**Please review and choose:**  
- [ ] Approve as-is  
- [ ] Approve with adjustments (describe)  
- [ ] Not approved (explain why)

### 5. REVISION LOOP

If not approved:  
- Ask how to improve  
- Revise  
- Re-submit  
- Repeat approval gate  

### 6. HANDOFF RULE

After approval:  
- Summarize what was approved (1–3 bullets)  
- State which agent comes next  
- **STOP**

### 7. MANUAL STEP RULE

Any non-AI action MUST be labeled **"Manual Step"** and include step-by-step instructions.

### 8. STATE AWARENESS RULE

Agents must respect previously approved artifacts.  
Agents must NOT re-ask approved information.

---

## Fixed Workflow Order

Agents must be run in this order only. No skipping, merging, or reordering.

| # | Agent |
|---|--------|
| 1 | Onboarding Agent |
| 2 | Producer Agent |
| 3 | Script Architect Agent |
| 4 | Retention Editor Agent |
| 5 | Voice Director Agent |
| 6 | Visual Director Agent (Animation) |
| 7 | Editor Agent |
| 8 | Audio Engineer Agent |
| 9 | Packaging Agent |
| 10 | Publishing Agent |

---

## Cursor Instructions

- **Always start with agents/onboarding.agent.md.**  
- Do **not** skip agents.  
- Do **not** merge agent responsibilities.  
- Do **not** reorder the workflow.  
- Respect approval gates: do not proceed to the next agent until the user has approved the current artifact.
