# DARK-METHOD

A gated, incremental, agency-style workflow for creating **dark** (faceless) YouTube channels that produce **animated, script-driven videos** with no external content dependency.

---

## What It Is

- **One agent active at a time**
- **One artifact per agent**
- **Explicit user approval between agents**
- **No skipping steps**
- **No cross-agent responsibilities**

AI agents: **Think**, **Plan**, **Structure**, **Specify**.  
Human user: **Approves**, **Executes manual steps**, **Publishes**.

---

## Workflow Order

1. **Onboarding Agent** — Consent and setup  
2. **Producer Agent** — Production Brief  
3. **Script Architect Agent** — Narration script (when the theme is researchable—e.g. "first round Brasileirao 2026"—fetches real web data and writes **research.md**; script uses that data)  
4. **Retention Editor Agent** — Retention-optimized script  
5. **Voice Director Agent** — Voice-directed script  
6. **Visual Director Agent** — Visual blueprint (animation)  
7. **Editor Agent** — Editing playbook  
8. **Audio Engineer Agent** — Audio mix guide  
9. **Packaging Agent** — Titles and thumbnails  
10. **Publishing Agent** — Publish-ready checklist  

---

## How to Start

1. **Open the repository root** (e.g. `dark.channel.maker`) in Cursor so rules load.
2. Run **Onboarding** first: say *"Start DARK-METHOD"* or *"Run the Onboarding agent"*. Onboarding asks first: **"Do you want to create a video for a new project or an existing project?"**
   - **New project:** You provide project name/slug, scope (Single video / Repeatable channel system), language, experience level. Onboarding creates **projects/{slug}/** and, if Repeatable channel, **channel-brief.md** with channel-level inputs. All artifacts for that project go there.
   - **Existing project:** Onboarding lists the folders in **projects/** and you select one. No new folder is created; you are creating a **new video** (or new episode) for that project. Next: run Producer to create the Episode Brief (or Production Brief) for this video.
3. **If new project and scope = "Repeatable channel system":** Onboarding also asks for **channel-level** inputs (channel name, positioning, default audience, default tone, format) and writes them to **channel-brief.md**. Each episode then gets its own **Episode Brief** (production-brief.md) from the Producer; channel-level info is not re-asked. See **Channel vs episode** below.
4. **Researchable themes and suggested sources:** When the video is about a real-world topic (e.g. "first round of Brasileirao 2026"), the **Producer** can ask for **suggested sources for research** (e.g. official league site, trusted news); the **Script Architect** will also prompt for suggested sources before researching. Providing trusted URLs or publication names helps keep the script accurate and reliable—the agent prioritizes these when fetching data.
5. **Replying to choices:** When the agent offers options (e.g. new vs existing project), it shows **1** and **2**; reply with **1** or **2** or type the option name. For "next step", type **/** and **click** the slash command (e.g. **/run-producer**).
6. Provide required inputs when asked; approve each artifact before moving to the next agent.
7. Execute manual steps when indicated.

**Always start with agents/onboarding.agent.md.** Do not skip or reorder agents. Each project has its own folder under **projects/**; do not mix projects.

---

## Channel vs episode (Repeatable channel system)

When you select **Repeatable channel system** at Onboarding, the framework segregates **channel-level** (general) information from **video/episode-level** (specific) information, following content-agency practice.

- **Channel-level** (in **channel-brief.md**, created by Onboarding): channel name, positioning/tagline, default audience, default tone, format description, optional recurring structure and branding. Downstream agents read this for consistency; they **do not re-ask** channel-level information.
- **Episode-level** (in **production-brief.md** and downstream): this episode/video goal, this episode length, this episode topic, this episode tone (or "per channel"), plus script, visuals, audio, packaging, publish. The Producer outputs an **Episode Brief** that references the channel brief; Script Architect, Retention Editor, Voice Director, Visual Director, Editor, Audio Engineer, Packaging, and Publishing may read channel-brief for tone, format, and branding when present.
- **Single video:** No channel-brief; production-brief.md is the full video brief and the Producer asks for all inputs.

---

## MCP integration (optional)

**MCPs (Model Context Protocol servers)** can assist agents and automate manual steps: filesystem (create project folders and artifact files), TTS (generate narration from the voice-directed script), image generation (thumbnails, concept art), YouTube (upload and metadata). See **dark-method/mcp-guide.md** for recommended MCPs by phase, config examples, and Cursor setup. Copy `.cursor/mcp.json.example` to `.cursor/mcp.json` and fill in paths and API keys.

**Image generation workflow (framework default):** For scene images (Visual Director) and thumbnails (Packaging), agents use **Stability AI MCP first** (preferred), then Gemini (free tier), then Replicate, then Fal-AI. Configure Stability AI in `.cursor/mcp.json` with `STABILITY_AI_API_KEY` to use it as the primary image generator. **Centralized image output:** All image MCPs that write to disk (Stability AI, Gemini) use the same folder **`projects/_mcp_images`**; set `IMAGE_STORAGE_DIRECTORY` and `OUTPUT_IMAGE_PATH` to that path. Agents create this folder once before any image-MCP call to avoid ENOENT and wasted tokens.

---

## Using DARK-METHOD in Cursor (Repository Root)

1. **Open the repository root** (e.g. `dark.channel.maker`) in Cursor (File → Open Folder). Rules in **.cursor/rules/** and commands in **.cursor/commands/** at root load automatically.
2. **Start the workflow:** In chat, type **/start-dark-method** or **/run-onboarding** (or type `/` and select **start-dark-method** or **run-onboarding** from the slash menu). The AI will follow the Onboarding agent. You will be asked for a **project name or slug** (e.g. `my-first-video`). Onboarding creates **projects/{slug}/** and writes the slug to `dark-method/.current-project`; all later agents write artifacts in **projects/{slug}/**.
3. **Run the next agent:** After approving an artifact, the agent will tell you to run the next step. **Click or type the slash command** (e.g. **/run-producer**, **/run-retention-editor**): in chat, type `/` and select the command, or type the full command (e.g. `/run-retention-editor`). No need to type "Run the Retention Editor agent"—just use the command.
4. **Slash commands (one click / one type):**  
   - **/start-dark-method** — Start DARK-METHOD (runs Onboarding: new or existing project)  
   - **/run-onboarding** — Same as start-dark-method; Onboarding agent  
   - **/run-producer** — Production Brief  
   - **/run-script-architect** — Narration script  
   - **/run-retention-editor** — Retention-optimized script  
   - **/run-voice-director** — Voice-directed script  
   - **/run-visual-director** — Visual blueprint  
   - **/run-editor** — Editing playbook + DaVinci Resolve importable scripts (Fusion Scripts Comp per-video folder)  
   - **/run-audio-engineer** — Audio mix guide  
   - **/run-packaging** — Packaging kit  
   - **/run-publishing** — Publish-ready checklist  
5. **New video project:** Run **/run-onboarding** again with a new project name; a new folder is created under **projects/** and set as current. Each video project stays in its own folder under **projects/**.

---

## Repository Structure (Repository Root)

```
<repository-root>/          e.g. dark.channel.maker
├── .cursor/
│   ├── rules/              # Cursor rules (load when root is open)
│   │   ├── dark-method-system.mdc
│   │   ├── dark-method-artifacts.mdc
│   │   └── dark-method-agent-context.mdc
│   └── commands/           # Slash commands: /start-dark-method, /run-onboarding, /run-producer, etc.
│       ├── start-dark-method.md
│       ├── run-onboarding.md
│       ├── run-producer.md
│       ├── run-script-architect.md
│       ├── run-retention-editor.md
│       ├── run-voice-director.md
│       ├── run-visual-director.md
│       ├── run-editor.md
│       ├── run-audio-engineer.md
│       ├── run-packaging.md
│       └── run-publishing.md
├── .cursorrules
├── dark-method/            # Framework only (agents, system, templates)
│   ├── .current-project   # One line = current project slug (set by Onboarding); artifacts in projects/{slug}/
│   ├── README.md
│   ├── dark-method.system.md
│   ├── mcp-guide.md            # MCP integration: recommended servers, config, automation
│   ├── agents/
│   │   ├── onboarding.agent.md
│   │   ├── producer.agent.md
│   │   ├── script-architect.agent.md
│   │   ├── retention-editor.agent.md
│   │   ├── voice-director.agent.md
│   │   ├── visual-director.agent.md
│   │   ├── editor.agent.md
│   │   ├── audio-engineer.agent.md
│   │   ├── packaging.agent.md
│   │   └── publishing.agent.md
│   └── project/            # Template files only; Onboarding copies structure to new project folders
│       ├── channel-brief.md # Template for Repeatable channel system only
│       ├── production-brief.md
│       ├── script.md
│       ├── visuals.md
│       ├── audio.md
│       ├── packaging.md
│       └── publish.md
└── projects/               # All video projects (default location)
    ├── my-first-video/     # One folder per video project (created by Onboarding)
    │   ├── production-brief.md
    │   ├── script.md
    │   ├── visuals.md
    │   ├── audio.md
    │   ├── packaging.md
    │   └── publish.md
    └── another-video/      # Another project
        └── ...
```

Artifacts for each video project live in **projects/{project-name}/**. The current project slug is stored in **dark-method/.current-project**. Use **dark-method/dark-method.system.md** as the governing system definition.
