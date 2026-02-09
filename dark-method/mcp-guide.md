# DARK-METHOD MCP Integration Guide

This guide lists **ready-to-use MCP (Model Context Protocol) servers** that can assist DARK-METHOD agents, reduce manual operations, and automate parts of the workflow. Configure them in Cursor (Settings > Features > MCP) or via project `.cursor/mcp.json`.

---

## Overview: Where MCPs Help

| Phase | Agent | Manual today | MCP can automate / assist |
|-------|--------|----------------|----------------------------|
| Onboarding | Onboarding | Create folder + artifact files | **Filesystem** — create `projects/{slug}/`, write template files |
| Voice | Voice Director | Record or run TTS | **TTS** — generate narration from voice-directed script |
| Visual | Visual Director | Generate assets | **Image** — concept art, thumbnails; **Video** — AI video from script |
| Packaging | Packaging | Create thumbnails | **Image** — generate thumbnail images from concepts |
| Publishing | Publishing | Upload to YouTube | **YouTube** — upload video, set title, description, chapters, tags |

---

## Recommended MCPs by Phase

### 1. Filesystem — Onboarding & all agents (NOT TO USE)

**Purpose:** Let the AI create project folders and write artifact files (e.g. `projects/my-video/`, `production-brief.md`) without you doing it manually.

**Server:** `@modelcontextprotocol/server-filesystem` (official)

- **Tools:** `write_file`, `create_directory`, `read_text_file`, `list_directory`, `move_file`, etc.
- **Use in DARK-METHOD:** Onboarding can create `projects/{slug}/` and the six artifact files; any agent can write/update artifacts in the current project folder.
- **Install:** `npx -y @modelcontextprotocol/server-filesystem@latest <allowed-dir1> [<allowed-dir2> ...]`
- **Config (Cursor):** Allow the repository root (or `projects/` + `dark-method/`) so the server can create and write only under your project.

**Example MCP config (add to Cursor or `.cursor/mcp.json`):**

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "C:/Projetos/dark.channel.maker"
      ]
    }
  }
}
```

Replace the path with your repo root. On Windows use forward slashes or escaped backslashes.

---

### 2. Text-to-Speech — Voice Director

**Purpose:** Generate narration audio from the voice-directed script so the user doesn’t have to record or run TTS manually.

**Preferred option (no payment):** **Coqui TTS MCP** — local TTS via [Coqui TTS](https://github.com/coqui-ai/TTS). No API key or credits. Use when ElevenLabs credits are low or you prefer free TTS.

**Options:**

| MCP | Best for | Notes |
|-----|----------|--------|
| **Coqui TTS MCP** | **Preferred (free)** | Local TTS; no API key. Tools: `list_models`, `text_to_speech`. Output: WAV. This repo: **tools/mcp-coqui-tts/**. See **Coqui TTS setup** below. |
| **ElevenLabs MCP** | High-quality TTS, many voices | Official; TTS, STT, voice cloning. Requires ElevenLabs API key and credits. [GitHub](https://github.com/elevenlabs/elevenlabs-mcp) |
| **Hume MCP** | Expressive TTS, voice design | Octave TTS; acting instructions. Requires Hume API key. [Docs](https://dev.hume.ai/docs/text-to-speech-tts/mcp-server) |
| **RealtimeTTS MCP** | Multiple engines (OpenAI, ElevenLabs, Azure, Coqui) | Python; `synthesize`, `stream`. Low latency. [PromptGenius](https://www.promptgenius.net/mcp/specialized-tools/text-to-speech) |
| **tts-mcp (OpenAI)** | Simple OpenAI TTS | Uses OpenAI TTS API; multiple voices, formats. |
| **Fish Audio MCP** | TTS, streaming, multiple voices | Fish Audio API. [GitHub](https://github.com/da-okazaki/mcp-fish-audio-server) |

**Use in DARK-METHOD:** Voice Director prefers **Coqui TTS** (free). After you approve the voice-directed script and authorize TTS, the agent calls Coqui TTS MCP with the script text and `output_directory: projects/{currentProject}/audio/`; output is `narration.wav`. If you prefer ElevenLabs and have credits, say so and the agent uses ElevenLabs instead.

**Coqui TTS MCP — setup (free, no API key):**

1. **Install:** From repo root: `cd tools/mcp-coqui-tts` then `uv sync` (or `pip install -e .`). Requires Python 3.9–3.11. First TTS run downloads a model (~hundreds of MB).
2. **`.cursor/mcp.json`:** Add the Coqui TTS server. Example (replace path with your repo root):
   ```json
   "coqui-tts": {
     "command": "uv",
     "args": [
       "run",
       "--project",
       "C:/Projetos/dark.channel.maker/tools/mcp-coqui-tts",
       "mcp-coqui-tts"
     ]
   }
   ```
   If you use a venv: `"command": "<path-to>/tools/mcp-coqui-tts/.venv/Scripts/python.exe"`, `"args": ["-m", "mcp_coqui_tts.server"]`.
3. **Restart Cursor** (or reload MCP). Voice Director will use Coqui TTS by default for narration.

**Example (Coqui TTS):** "Generate TTS for the narration in `projects/my-video/script.md` and save to `projects/my-video/audio/`." Output: `narration.wav`.

**Example (ElevenLabs):** Configure with your API key; then in chat: "Generate TTS for the narration in `projects/my-video/script.md` using voice X and save to `projects/my-video/audio/narration.mp3`."

**Example (ElevenLabs):** Configure with your API key; then in chat: “Generate TTS for the narration in `projects/my-video/script.md` using voice X and save to `projects/my-video/audio/narration.mp3`.”

---

### 3. Image generation — Visual Director & Packaging

**Purpose:** Generate concept art, scene art, or thumbnail images from the visual blueprint or packaging concepts.

**Centralized image output folder:** All image MCPs that write to disk (Stability AI, Gemini) **must use the same output folder** so you have one place to manage and so the agents can create it once before any call. Use **`projects/_mcp_images`** (absolute path in config, e.g. `C:/Projetos/dark.channel.maker/projects/_mcp_images`).

- Set **Stability AI** `IMAGE_STORAGE_DIRECTORY` and **Gemini** `OUTPUT_IMAGE_PATH` to this same path.
- **Agents create this folder once** at the start of image generation (before calling any image MCP), so the directory always exists and you avoid ENOENT errors and wasted tokens.
- Replicate and Fal-AI return URLs; the agent downloads directly to the project folder, so they do not use `_mcp_images`.

**Framework workflow (mandatory):** DARK-METHOD fixes the image-generation order. Agents (Visual Director, Packaging) **must** try **Stability AI MCP first** for every scene or thumbnail when authorized; only if the tool is unavailable or returns an error do they try the next. Order: **1. Stability AI** → **2. Gemini** → **3. Replicate** → **4. Fal-AI**. No reordering. See **dark-method/dark-method.system.md** (§ Image generation workflow) and the agent files for step-by-step flows.

**Preferred option:** **Stability AI MCP** — high-quality image generation and manipulation (generate, edit, upscale, remove background) via [Stability AI](https://platform.stability.ai). Pay-as-you-go (~$0.03/image on Core). Use this first when configured.

**Options:**

| MCP | Best for | Notes |
|-----|----------|--------|
| **Stability AI MCP** | **Preferred** | Generate, edit, upscale, remove background. Tools: `generate-image`, `generate-image-sd35`, `remove-background`, `upscale-fast`, etc. [GitHub](https://github.com/tadasant/mcp-server-stability-ai). Requires API key at [platform.stability.ai](https://platform.stability.ai/account/keys). Set `IMAGE_STORAGE_DIRECTORY` to the **centralized folder** (see above). See **Stability AI setup** below. |
| **Gemini Image Generator MCP** | **Free tier** | Text-to-image via Google Gemini 2.5 Flash Image. Free key at [Google AI Studio](https://aistudio.google.com/apikey). Tools: `generate_image_from_text`, `transform_image_from_file`. [GitHub](https://github.com/qhdrl12/mcp-server-gemini-image-generator). Set `OUTPUT_IMAGE_PATH` to the **same centralized folder**. See **Gemini setup** below. |
| **DALL·E / Azure OpenAI DALL·E 3** | Thumbnails, concept art | Text-to-image; configurable size, quality. Requires OpenAI or Azure API key. |
| **Replicate MCP (GongRzhe)** | FLUX Schnell (fallback) | Single tool: `generate_image`. Uses Replicate API (pay-per-use). [GitHub](https://github.com/GongRzhe/Image-Generation-MCP-Server). Configure `REPLICATE_API_TOKEN`. Returns URLs; agent downloads to project folder. |
| **Fal MCP** | FLUX, SD (last fallback) | When Stability AI/Gemini/Replicate fail or require payment. [GitHub](https://github.com/raveenb/fal-mcp-server). Configure `FAL_KEY`. Returns URLs; agent saves to project folder. |
| **Creatify MCP** | Video + TTS + avatars | Full video generation, not only images; see below. |

**Stability AI MCP — setup (preferred):**

1. **Install:** No clone needed. Run via `npx -y mcp-server-stability-ai`. Requires Node.js.
2. **API key:** [Stability AI → Account Keys](https://platform.stability.ai/account/keys). Get an API key (25 free credits; then ~$0.01/credit).
3. **`.cursor/mcp.json`:** Add the Stability AI server. Set `STABILITY_AI_API_KEY`. Set `IMAGE_STORAGE_DIRECTORY` to the **centralized image folder** (same as Gemini): e.g. `C:/Projetos/dark.channel.maker/projects/_mcp_images`. The Visual Director/Packaging agent creates this folder before the first image-MCP call so you avoid ENOENT and wasted tokens.
   ```json
   "stability-ai": {
     "command": "npx",
     "args": ["-y", "mcp-server-stability-ai"],
     "env": {
       "STABILITY_AI_API_KEY": "sk-...",
       "IMAGE_STORAGE_DIRECTORY": "C:/Projetos/dark.channel.maker/projects/_mcp_images"
     }
   }
   ```
4. **Restart Cursor** (or reload MCP). Visual Director and Packaging use Stability AI first when configured.

**Gemini Image Generator MCP — setup (free tier):**

1. **Clone:** from repo root, `cd tools/mcp-gemini-image-generator` then `git clone https://github.com/qhdrl12/mcp-server-gemini-image-generator mcp-server-gemini-image-generator` (this creates the subfolder with the project).
2. **Install:** run `uv sync` from the **subfolder** where `pyproject.toml` is: `cd tools/mcp-gemini-image-generator/mcp-server-gemini-image-generator` then `uv sync`. Install [uv](https://github.com/astral-sh/uv#installation) if needed.
3. **API key (free):** [Google AI Studio → Create API Key](https://aistudio.google.com/apikey).
4. **`.cursor/mcp.json`:** set `GEMINI_API_KEY` to your key. Set `OUTPUT_IMAGE_PATH` to the **same centralized folder** as Stability AI: e.g. `C:/Projetos/dark.channel.maker/projects/_mcp_images`. The `--directory` must point to the folder that contains `pyproject.toml` (e.g. `.../tools/mcp-gemini-image-generator/mcp-server-gemini-image-generator`). Agents create `projects/_mcp_images` before any image call.
5. **Restart Cursor** (or reload MCP).

**Replicate Try for Free (Replicate MCP):** Use the **Try for Free** collection so you can run models without purchasing credit first. In Replicate MCP: call `get_collection` with slug **`try-for-free`** to list free models, or use these directly:

- **Image:** `google/imagen-4`, `black-forest-labs/flux-1.1-pro`, `black-forest-labs/flux-kontext-pro`, `ideogram-ai/ideogram-v3-turbo`, `black-forest-labs/flux-dev`
- **Video:** `minimax/video-01`, `luma/reframe-video`, `topazlabs/video-upscale`
- **Upscaling / restoration:** `topazlabs/image-upscale`, `szcho/codeformer`, `tencentarc/gfpgan`

Free runs are limited per model; after the limit, add billing. See [Try for Free](https://replicate.com/collections/try-for-free).

**Replicate and payment:** Replicate charges per prediction; there is no separate "free MCP". Both the previous generic Replicate MCP and the current **GongRzhe Image Generation MCP** use the same Replicate API and `REPLICATE_API_TOKEN`. If Replicate asks for payment, it is account/platform billing—switching MCP servers does not change that. The GongRzhe server simplifies the flow (one `generate_image` tool, default model `black-forest-labs/flux-schnell`) and returns image URLs; the agent downloads from those URLs to the project assets folder.

**This repo: Replicate via GongRzhe Image Generation MCP (fallback).** The project uses [GongRzhe/Image-Generation-MCP-Server](https://github.com/GongRzhe/Image-Generation-MCP-Server) (run via npx). It exposes a single tool:
- **`generate_image`** — prompt, optional seed, aspect_ratio, output_format, num_outputs; returns JSON with image URL(s). The Visual Director (or Packaging) then downloads each URL to `projects/{currentProject}/assets/scene-{N}.png` (e.g. via run command: `curl`, `Invoke-WebRequest`, etc.). Use when Stability AI and Gemini are not used.

**.cursor/mcp.json** (Replicate, optional):
```json
"replicate": {
  "command": "npx",
  "args": ["-y", "@gongrzhe/image-gen-server"],
  "env": { "REPLICATE_API_TOKEN": "..." }
}
```
Optional env: `MODEL` (default `black-forest-labs/flux-schnell`). No local build required; restart Cursor (or reload MCP) after changing config.

**Use in DARK-METHOD:**  
- **Visual Director (Stability AI preferred, then Gemini, then Replicate, then Fal):** Prefer **Stability AI MCP** when configured (high quality, pay-as-you-go). If Stability AI is unavailable or returns an error, use **Gemini Image Generator** (free tier); then Replicate; then Fal-AI. “Generate concept art for scene 1 from the description in `visuals.md`.”  
- **Packaging:** Same order: Stability AI first, then Gemini, then Fal. “Generate a thumbnail image from the chosen concept in `packaging.md` (1280x720).”

---

### 4. Video generation — Visual Director (optional)

**Purpose:** Generate or assemble video from script/visual blueprint to reduce manual editing.

**Options:**

| MCP | Best for | Notes |
|-----|----------|--------|
| **Creatify MCP** | Avatar videos, TTS, URL-to-video, AI editing | 12 tools; avatar + TTS; 140+ voices. [GitHub](https://github.com/TSavo/creatify-mcp) |
| **JSON2Video MCP** | Programmatic video from API | Generate video via json2video API. [GitHub](https://github.com/omergocmen/json2video-mcp-server) |
| **Fal MCP** | AI video models | Fal.ai video models. |

**Use in DARK-METHOD:**  
- **Creatify:** Good for “faceless” avatar + TTS videos; Voice Director + Visual Director outputs can drive script and style; Packaging/Publishing stay separate.  
**Timeline assembly (Resolve):** Timeline assembly is **not** automated via MCP. The Editor agent generates **DaVinci Resolve–compatible Python scripts** (using the official Resolve scripting API) and writes them to a **per-video folder** under the Fusion Scripts Comp directory: **`C:\Users\frD\AppData\Roaming\Blackmagic Design\DaVinci Resolve\Support\Fusion\Scripts\Comp\{currentProjectFolder}\`** (e.g. `build_timeline.py`, `edit-instructions.json`). The user runs the script manually in DaVinci Resolve (free or Studio) to create the project, import media, and build the timeline. See **dark-method/agents/editor.agent.md**.

---

### 5. YouTube — Publishing

**Purpose:** Upload the final video and set metadata (title, description, chapters, tags) without using YouTube Studio.

**Options:**

| MCP | Best for | Notes |
|-----|----------|--------|
| **YouTube Uploader MCP** (anwerj) | Upload + OAuth2, multi-channel | Binary; OAuth2; no CLI/Studio. [GitHub](https://github.com/anwerj/youtube-uploader-mcp) |
| **YouTube MCP** (ZubeidHendricks) | Video management, Shorts, analytics | Broader YouTube API. [GitHub](https://github.com/ZubeidHendricks/youtube-mcp-server) |
| **Pipedream YouTube Data** | YouTube Data API | Managed; search, metadata, playlists. |

**Use in DARK-METHOD:**  
- **Publishing agent:** After the user approves the publish checklist, use YouTube Uploader MCP to upload the file and set title, description, chapters, tags from `publish.md`.  
- **Setup:** Google Cloud project + OAuth 2.0 credentials → `client_secret.json`; configure MCP with path to that file.

**Example (YouTube Uploader MCP):**

```json
{
  "mcpServers": {
    "youtube-uploader-mcp": {
      "command": "C:/path/to/youtube-uploader-mcp-windows-amd64.exe",
      "args": ["-client_secret_file", "C:/path/to/client_secret.json"]
    }
  }
}
```

**This repo:** The project uses [anwerj/youtube-uploader-mcp](https://github.com/anwerj/youtube-uploader-mcp). See **tools/youtube-uploader-mcp/README.md** for download, OAuth2 (`client_secret.json`), and Cursor config. Binary goes in `tools/youtube-uploader-mcp/`; client secret in `.cursor/client_secret.json` (gitignored).

---

## Other useful MCPs

- **Transcribe MCP** — Transcribe audio/video to text (captions, or to align script with recording). [GitHub](https://github.com/transcribe-app/mcp-transcribe)
- **mcp-youtube-extract** — Get video info and transcripts (reference or research). [GitHub](https://github.com/sinjab/mcp_youtube_extract)
- **Cloudinary MCP** — Upload and manage media (host thumbnails or assets). [GitHub](https://github.com/felores/cloudinary-mcp-server)

---

## How to configure MCPs in Cursor

1. **Global:** Settings → Features → MCP → “+ Add New MCP Server” → set Name, Type (Command), Command (and env vars if needed).  
2. **Project:** Create `.cursor/mcp.json` in the repo root with a `mcpServers` object (see examples above).  
3. **Security:** For the filesystem server, allow only the repo root (or specific subdirs) so the AI cannot write outside the project.

---

## Example: minimal project MCP setup

Save as `.cursor/mcp.json` in the repository root (or merge into existing MCP config). Replace paths and API keys with your own; do not commit secrets.

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "<YOUR_REPO_ROOT_ABSOLUTE_PATH>"
      ]
    }
  }
}
```

Add TTS, image, or YouTube servers as needed (see sections above). After configuration, Cursor will expose the MCP tools to the AI; you can say e.g. “Create the project folder and artifact files for slug my-new-video” (Onboarding) or “Generate TTS for the script in projects/my-video/script.md” (Voice).

---

## Agent MCP authorization

Agents that can use MCPs (Voice Director, Visual Director, Audio Engineer, Editor, Packaging, Publishing) **prompt for authorization in a separate step after you approve the artifact**. For **image generation** (Visual Director, Packaging), agents follow the framework workflow: **Stability AI first**, then Gemini, then Replicate (Visual Director only), then Fal-AI. See § Image generation above and dark-method.system.md § Image generation workflow.

- First, the agent presents only the **approval gate** (Approve as-is / Approve with adjustments / Not approved). No MCP question is mixed with that prompt.
- After you approve (e.g. Approve as-is), the agent asks in a **new, separate message** whether you authorize automatic execution of the MCP's activities (e.g. generate TTS, generate thumbnails, upload to YouTube).
- If you say **yes** and the MCP is available, the agent calls the MCP to automate the manual step; if **no** (or MCP not configured), the agent provides **Manual Step** instructions. Then handoff to the next agent.

This keeps the approval decision and the MCP authorization decision separate and avoids confusing context.

---

## Consistency with DARK-METHOD rules

- **One agent at a time:** MCPs are tools the agent (or user) can call during that agent’s step; they do not replace the agent or skip the approval gate.  
- **Approval gate:** Automating a manual step (e.g. TTS, upload) still requires the user to approve the agent’s output before moving to the next agent. For agents that both produce an artifact and can automate via MCP (Visual Director, Voice Director, Audio Engineer, Editor, Packaging, Publishing), the approval gate is presented **alone** first; MCP authorization is a **separate step** after approval, so the user is not asked to approve and authorize in the same prompt.  
- **Single artifact:** Each agent still produces one primary artifact; MCPs help create or fill it (e.g. write files, generate audio).  
- **State awareness:** When using the filesystem MCP, the AI must respect `dark-method/.current-project` and, for episode-level artifacts, the **artifact base path**: when `projects/{currentProjectFolder}/channel-brief.md` exists, read `dark-method/.current-video` and write under `projects/{currentProjectFolder}/{currentVideoFolder}/`; otherwise under `projects/{currentProjectFolder}/`.

---

## References

- [Cursor MCP docs](https://cursor.com/docs/context/mcp)  
- [Official MCP registry](https://registry.modelcontextprotocol.io/)  
- [MCPList – Media](https://www.mcplist.ai/function/media)  
- [Model Context Protocol – connect local servers](https://modelcontextprotocol.io/docs/develop/connect-local-servers)
