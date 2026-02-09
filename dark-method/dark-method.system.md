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
- **Current project folder** is stored in `dark-method/.current-project` (one line = project name/slug, no path).
- **Artifact base path** — where agents read and write **episode-level** artifacts (production-brief, script, visuals, audio, packaging, publish, research):
  - **When `projects/{currentProjectFolder}/channel-brief.md` does NOT exist (Single video):** artifact base path = **projects/{currentProjectFolder}/**. All episode artifacts live there.
  - **When `projects/{currentProjectFolder}/channel-brief.md` EXISTS (Repeatable channel system):** artifact base path = **projects/{currentProjectFolder}/{currentVideoFolder}/**, where `{currentVideoFolder}` is the value in **dark-method/.current-video** (one line = video slug, e.g. `rodada-1-brasileirao-2026`). The Producer creates this folder and sets `.current-video` when starting a new episode (deriving the video slug from the episode goal/theme so each video has its own folder and files are not mixed).
- **Onboarding** asks first: "Do you want to create a video for a **new project** or an **existing project**?"
  - **New project:** Onboarding creates the project folder at **projects/{slug}** (and channel-brief.md if scope = Repeatable channel system). If **Repeatable channel system**, it does **not** create the six episode artifact files (production-brief, script, visuals, audio, packaging, publish) in the project root—those are created per-video by the Producer and downstream agents. Then it writes the slug to `dark-method/.current-project`.
  - **Existing project:** Onboarding lists the folders in **projects/** and lets the user select one. It writes the selected project slug to `dark-method/.current-project`. No new folder is created; the user is creating a **new video** (or new episode) for that existing project. The **Producer** will create a **per-video folder** (named from the video’s purpose/theme) and set `.current-video`. Next step: run Producer to create the Episode Brief (or Production Brief) for this video.
- If `.current-project` is missing or empty, the user must run Onboarding first (new project or select existing project).
- **Channel-level** (channel-brief.md) always lives at **projects/{currentProjectFolder}/channel-brief.md**. Episode-level paths are always under the **artifact base path** above (so for Repeatable channels, one folder per video; for Single video, the project folder).

---

## Channel vs Episode (Repeatable Channel System)

When the user selects **Repeatable channel system** at Onboarding, the framework segregates **channel-level** (general) information from **video/episode-level** (specific) information. This follows content-agency practice: one governing channel brief, then per-episode briefs and artifacts.

### Channel-level (general; applies to all episodes)

- Stored in **projects/{currentProjectFolder}/channel-brief.md** (created by Onboarding when scope = "Repeatable channel system").
- **Channel-level** includes: channel name, positioning/tagline, default target audience (sophistication, interests), default tone (e.g. dramatic, calm, ominous), format description (e.g. "animated roundup, 8–12 min per episode"), optional recurring structure (e.g. "hook → context → blocks → CTA"), optional branding guidelines.
- Agents must **not re-ask** channel-level information once it is in channel-brief.md. Downstream agents read channel-brief.md when present for consistency (tone, voice, format, branding).

### Episode-level (video-specific; applies to this video only)

- Stored under the **artifact base path** (see Project Folders): **production-brief.md** (Episode Brief when channel-brief exists), **script.md**, **visuals.md**, **audio.md**, **packaging.md**, **publish.md**. Optionally **research.md** (created by Script Architect when the theme is researchable). For **Repeatable channel system**, the artifact base path is **projects/{currentProjectFolder}/{currentVideoFolder}/** (one folder per video, so each episode’s files stay separate).
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

## Presenting choices to the user (UX in Cursor)

Cursor chat does **not** provide native clickable option chips or suggestion buttons. To keep choices easy and low-friction:

1. **When you offer multiple-choice options** (e.g. new project vs existing project, Single video vs Repeatable channel), present them in this format:
   - **1** — [Option A label]  
   - **2** — [Option B label]  
   Then add: **"Reply with 1 or 2, or type the option name."** So the user can type a single character or short phrase.
2. **For "what to do next"** use **slash commands**: tell the user to run **/run-producer** (or the next agent). In Cursor, the user can type **/** in the chat input to open the command list and **click** the command—that is the clickable UX for next steps.
3. **Keep option labels short** so the user can type them quickly if they prefer (e.g. "new project", "existing project").
4. **Accept equivalent replies**: treat "1", "new", "new project" as the same when the first option is "New project"; same for 2 and "existing project".

This gives a consistent, minimal-typing experience until Cursor adds native interactive choice UI.

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

## DaVinci Resolve (Editor — no MCP)

Timeline assembly is **not** automated. The **Editor agent** generates **importable scripts** for DaVinci Resolve (free or Studio): Python scripts using the official Resolve scripting API. Scripts and data are written to a **per-video folder** under the Fusion Scripts Comp directory: when **channel-brief.md** exists (Repeatable channel), use **`Comp\{currentProjectFolder}\{currentVideoFolder}\`** (e.g. `brasileirao-2026-animado\rodada-1-brasileirao-2026`); otherwise **`Comp\{currentProjectFolder}\`** (e.g. `build_timeline.py`, `edit-instructions.json`). The user runs the script manually in Resolve (Workspace > Scripts or Console) to create the project, import media, and build the timeline. No Resolve MCP or external automation is used. See **dark-method/agents/editor.agent.md** for the API reference and script format.

---

## Image generation workflow (MCP)

When agents use MCP for image generation (Visual Director for scene assets, Packaging for thumbnails), they **must** follow this order—no reordering or skipping:

1. **Stability AI MCP** (preferred) — Use **generate-image** (server **stability-ai**) first for every scene or thumbnail when the MCP is configured. Copy the saved file from the **centralized MCP image folder** (`projects/_mcp_images`; same path as `IMAGE_STORAGE_DIRECTORY` / `OUTPUT_IMAGE_PATH`) to the **artifact base path** (`projects/{currentProjectFolder}/assets/` or with `{currentVideoFolder}/` when Repeatable channel; same for `thumbnail-{N}.png`).
2. **Gemini Image Generator MCP** (fallback) — Only when Stability AI is not used for that image (tool unavailable or returned an error). Use **generate_image_from_text**; copy the returned file path to the project folder.
3. **Replicate MCP** (fallback) — Only when Stability AI and Gemini were not used. Use **generate_image**; download the returned URL(s) to the project folder.
4. **Fal-AI MCP** (last fallback) — Only when all of the above failed or require payment.

Agents must **try Stability AI first** for each image; only on unavailability or error do they try the next option. **Before calling any image MCP**, agents create the **centralized image folder** (`projects/_mcp_images`) once so it exists and tokens are not wasted on ENOENT. All image MCPs that write to disk (Stability AI, Gemini) use this same folder. See **dark-method/mcp-guide.md** for MCP setup and **dark-method/agents/visual-director.agent.md** and **packaging.agent.md** for step-by-step workflows.

---

## MCP authorization gate (user-facing)

Some agents (Visual Director, Packaging, Voice Director) use MCPs to generate assets (images, TTS). **Before** calling any MCP, they ask for your explicit authorization in a **separate step** after you approve the artifact.

- **When you see the MCP authorization question:** The framework is **waiting for your reply**. Nothing will be generated until you respond.
- **To authorize:** Reply **yes** or **authorize** (or e.g. **ElevenLabs** if the agent offers that choice).
- **To decline:** Reply **no** (or skip); the agent will point you to the Manual Step.

Agents must present this gate in a **visually distinct block** (see agent files: heading **MCP AUTHORIZATION — WAITING FOR YOUR REPLY** and the single question) so it is easy to notice among other text.

---

## Cursor Instructions

- **Always start with agents/onboarding.agent.md.**  
- Do **not** skip agents.  
- Do **not** merge agent responsibilities.  
- Do **not** reorder the workflow.  
- Respect approval gates: do not proceed to the next agent until the user has approved the current artifact.
