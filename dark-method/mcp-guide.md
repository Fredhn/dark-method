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

**Options:**

| MCP | Best for | Notes |
|-----|----------|--------|
| **ElevenLabs MCP** | High-quality TTS, many voices | Official; TTS, STT, voice cloning. Requires ElevenLabs API key. [GitHub](https://github.com/elevenlabs/elevenlabs-mcp) |
| **Hume MCP** | Expressive TTS, voice design | Octave TTS; acting instructions. Requires Hume API key. [Docs](https://dev.hume.ai/docs/text-to-speech-tts/mcp-server) |
| **RealtimeTTS MCP** | Multiple engines (OpenAI, ElevenLabs, Azure, Coqui) | Python; `synthesize`, `stream`. Low latency. [PromptGenius](https://www.promptgenius.net/mcp/specialized-tools/text-to-speech) |
| **tts-mcp (OpenAI)** | Simple OpenAI TTS | Uses OpenAI TTS API; multiple voices, formats. |
| **Fish Audio MCP** | TTS, streaming, multiple voices | Fish Audio API. [GitHub](https://github.com/da-okazaki/mcp-fish-audio-server) |

**Use in DARK-METHOD:** After Voice Director outputs the voice-directed script, the user (or the agent, if you allow tool use) can call the TTS MCP with the script text and chosen voice to generate `projects/{currentProject}/audio/narration.mp3` (or similar). Reduces “Manual Step: record or run TTS” to a single MCP call.

**Example (ElevenLabs):** Configure with your API key; then in chat: “Generate TTS for the narration in `projects/my-video/script.md` using voice X and save to `projects/my-video/audio/narration.mp3`.”

---

### 3. Image generation — Visual Director & Packaging

**Purpose:** Generate concept art, scene art, or thumbnail images from the visual blueprint or packaging concepts.

**Options:**

| MCP | Best for | Notes |
|-----|----------|--------|
| **DALL·E / Azure OpenAI DALL·E 3** | Thumbnails, concept art | Text-to-image; configurable size, quality. Requires OpenAI or Azure API key. |
| **Fal MCP** | FLUX, Stable Diffusion, MusicGen | Images, video, music. [GitHub](https://github.com/raveenb/fal-mcp-server) |
| **Replicate MCP** | FLUX, SD, many models | Run Replicate models (images, video). [GitHub](https://github.com/deepfates/mcp-replicate) |
| **Image Generation MCP (Replicate Flux)** | Quick image gen | Replicate Flux model. |
| **Creatify MCP** | Video + TTS + avatars | Full video generation, not only images; see below. |

**Use in DARK-METHOD:**  
- **Visual Director:** “Generate concept art for scene 1 from the description in `visuals.md`.”  
- **Packaging:** “Generate a thumbnail image from the chosen concept in `packaging.md` (1280x720).”

---

### 4. Video generation — Visual Director (optional)

**Purpose:** Generate or assemble video from script/visual blueprint to reduce manual editing.

**Options:**

| MCP | Best for | Notes |
|-----|----------|--------|
| **Creatify MCP** | Avatar videos, TTS, URL-to-video, AI editing | 12 tools; avatar + TTS; 140+ voices. [GitHub](https://github.com/TSavo/creatify-mcp) |
| **JSON2Video MCP** | Programmatic video from API | Generate video via json2video API. [GitHub](https://github.com/omergocmen/json2video-mcp-server) |
| **DaVinci Resolve MCP** | Timeline editing in Resolve | If you edit in Resolve: project control, media, grading. [GitHub](https://github.com/samuelgursky/davinci-resolve-mcp) |
| **Fal MCP** | AI video models | Fal.ai video models. |

**Use in DARK-METHOD:**  
- **Creatify:** Good for “faceless” avatar + TTS videos; Voice Director + Visual Director outputs can drive script and style; Packaging/Publishing stay separate.  
- **DaVinci Resolve:** Editor agent can describe cuts; user or MCP can drive Resolve for timeline assembly (if you use Resolve).

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

Agents that can use MCPs (Voice Director, Visual Director, Editor, Packaging, Publishing) **prompt for authorization** at the start of their Process:

- They ask whether you have the relevant MCP configured (e.g. in `.cursor/mcp.json` or Settings > MCP).
- They ask whether you authorize **automatic execution** of the MCP's activities (e.g. generate TTS, generate thumbnails, upload to YouTube).
- If you say **yes** and the MCP is available, the agent may call the MCP to automate the manual step; if **no** (or MCP not configured), the agent provides **Manual Step** instructions as usual.

This keeps control with you and avoids running external tools without consent.

---

## Consistency with DARK-METHOD rules

- **One agent at a time:** MCPs are tools the agent (or user) can call during that agent’s step; they do not replace the agent or skip the approval gate.  
- **Approval gate:** Automating a manual step (e.g. TTS, upload) still requires the user to approve the agent’s output before moving to the next agent.  
- **Single artifact:** Each agent still produces one primary artifact; MCPs help create or fill it (e.g. write files, generate audio).  
- **State awareness:** When using the filesystem MCP, the AI must respect `dark-method/.current-project` and write only under `projects/{currentProjectFolder}/`.

---

## References

- [Cursor MCP docs](https://cursor.com/docs/context/mcp)  
- [Official MCP registry](https://registry.modelcontextprotocol.io/)  
- [MCPList – Media](https://www.mcplist.ai/function/media)  
- [Model Context Protocol – connect local servers](https://modelcontextprotocol.io/docs/develop/connect-local-servers)
