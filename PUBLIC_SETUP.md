# YouTube Documentary Automation — Setup Guide

Automated AI pipeline for creating long-form YouTube documentary scripts and image prompts from a single video title.

This is **not** a fully standalone, one-click workflow. It requires an n8n installation, your own AI API credentials, and (for the research step) a local search service. Read this whole guide before importing.

## What the workflow does

You send it a video topic through a chat trigger, and it runs through a pipeline of AI agents and code nodes to produce two output files per project:

1. `final_script.txt` — a ~15,000-word narrated documentary script, broken into chapters
2. `image_prompt.csv` — a scene-by-scene list of AI image-generation prompts covering the whole script

Pipeline order:

```
When chat message received
→ Prompt Library
→ Research
→ Outline Agent
→ Split Chapters
→ Chapter Writer (one pass per chapter)
→ Total / Collect Chapters
→ Prepare Final Script Input
→ Final Script Writer
→ Image Prompt Generator
→ Merge Final Script + Image Prompts
→ Prepare Project Files
→ Create Project Folder
→ Save Project Files
```

It does not generate images, video, or audio itself — it only produces the script and the CSV of image prompts you'd feed into a separate image/video pipeline.

## Before you import

You will need:

- **Docker and Docker Compose**, to run n8n and SearXNG together (see below).
- **The `@n8n/n8n-nodes-langchain` community/LangChain nodes installed.** The chat trigger, the AI agent nodes, and the Google Gemini chat model nodes all come from this package, not from core n8n. (The standard `n8nio/n8n` Docker image used in `docker-compose.yml` includes these built in.)
- **Your own Google Gemini API credentials** (the workflow calls `models/gemini-3.5-flash-lite` through the Google Gemini(PaLM) node). You'll need your own API key/credits — none are included in the JSON.
- **A SearXNG instance for the Research step.** The "Research" node calls `http://searxng:8080` with query parameters matching SearXNG's JSON search API (`q`, `format=json`, `language`, `safesearch`). That hostname only resolves if SearXNG is running as a container named `searxng` on the same Docker network as n8n — which is exactly what the included `docker-compose.yml` sets up for you. If you're not using that Compose file, you must run your own SearXNG (or compatible JSON search API) and update the URL in the "Research" node to match.

## Quick start with Docker Compose (recommended)

This repo includes a `docker-compose.yml` that starts n8n and SearXNG together on one Docker network, so the Research node's `http://searxng:8080` URL resolves out of the box.

1. Clone the repo and `cd` into it.
2. Run `docker compose up -d`.
3. **Enable JSON output in SearXNG** — it's disabled by default for security, and the Research node needs it:
   - Find your SearXNG settings file (Compose mounts it to a `searxng_data` volume at `/etc/searxng/settings.yml` inside the container — on first run SearXNG generates a default one there).
   - Edit `settings.yml` and make sure the `search.formats` list includes `json`:
     ```yaml
     search:
       formats:
         - html
         - json
     ```
   - Restart the SearXNG container: `docker compose restart searxng`.
4. Open n8n at `http://localhost:5678` and continue with the import steps below.

Generated files also show up on your host machine under `./youtube-projects` (mapped from the container's `/home/node/.n8n/YouTubeProjects`).

If you'd rather not use Compose, you can run n8n and SearXNG however you like — just make sure the "Research" node's URL points at wherever your SearXNG (or compatible) instance is actually reachable, and that its JSON format is enabled.

## How to import

1. Open your n8n instance.
2. Go to **Workflows → Import from File** (or **Import from URL** if you're pulling straight from GitHub).
3. Select `youtube-documentary-automation.json`.
4. n8n will list every node and ask you to assign credentials for the ones that need them.

## Configuring credentials

The workflow has four Google Gemini chat model nodes (`Google Gemini Chat Model`, `Google Gemini Chat Model1/2/3`), each feeding a different AI agent (Outline Agent, Chapter Writer, Final Script Writer, Image Prompt Generator). After import:

1. Click each Gemini model node.
2. Under **Credential to connect with**, select **Create New** (or an existing credential) and enter your own Google Gemini/PaLM API key.
3. Repeat for all four nodes — they are separate credential slots even though they use the same model, so you can point them at the same key or different keys/quotas.
4. Update the "Research" node's URL to point at a search API you actually have running (see above).

You need your own API access and will be billed/rate-limited according to your own Google Gemini account — nothing here is pre-funded.

## Using the trigger

The workflow starts from a LangChain **"When chat message received"** node — this gives you a chat-style webhook/test interface in n8n where you type or send the topic (e.g. "The Salem Witch Trials") as the chat input. Run the workflow and provide the topic through that chat panel (or via the trigger's webhook if you wire it up externally).

## Where files are saved

Everything is written **inside the n8n Docker container's filesystem**, not your host machine directly (unless you've mounted a volume there yourself):

```
/home/node/.n8n/YouTubeProjects/<PROJECT_NAME>/final_script.txt
/home/node/.n8n/YouTubeProjects/<PROJECT_NAME>/image_prompt.csv
```

`<PROJECT_NAME>` is auto-generated from the topic text (sanitized, whitespace-collapsed, truncated to 120 characters).

If you want the files on your host machine, mount a Docker volume to `/home/node/.n8n/YouTubeProjects` when you start your n8n container, or adjust the path in the "Prepare Project Files" code node to a path you've made accessible.

## Environment / runtime requirements for the fs Code node

- The **"Create Project Folder"** and **"Save Project Files"** nodes use Node's built-in `fs` module directly (`mkdirSync`, `writeFileSync`, `renameSync`, `unlinkSync`, `existsSync`). No extra npm packages are required — `fs` ships with Node.
- Your n8n instance must allow Code nodes to use Node.js built-in modules. This is the n8n default; if you've locked this down via `NODE_FUNCTION_ALLOW_BUILTIN`, make sure `fs` is on the allow-list.
- The container user running n8n needs write permission to create `/home/node/.n8n/YouTubeProjects/...`. This is normally true by default in the standard n8n Docker image.
- The save step writes to a temporary file first and atomically renames it onto the final filename. If that rename fails with a file-lock-style error (`EACCES`, `EBUSY`, `EPERM` — e.g. if `final_script.txt` is open in another program via a mounted volume), it automatically falls back to a timestamped filename in the same folder instead of failing the run.

## Local services that must be running

- **A SearXNG-compatible JSON search API**, reachable at `http://searxng:8080` (provided automatically by the included `docker-compose.yml`) or whatever URL you put in the "Research" node if you're not using Compose. This is a hard dependency — the Outline Agent step uses its results, and the workflow will fail without a reachable search endpoint. SearXNG's JSON output format must be explicitly enabled in its `settings.yml` — it's off by default.

## Limitations and assumptions

- This workflow assumes a **Docker-hosted** n8n instance running alongside a **SearXNG** container reachable at the hostname `searxng` (as set up by `docker-compose.yml`). Running n8n directly on a bare-metal install will still work for the script-writing part, but the save path will land wherever `/home/node/.n8n/...` resolves to on that machine (create the folder yourself if it doesn't exist automatically), and you'll need to point the "Research" node at a real, reachable SearXNG URL instead of `http://searxng:8080`.
- SearXNG must have its `json` output format enabled (`search.formats` in `settings.yml`) — many default SearXNG configs ship with it disabled, which will make the "Research" node fail or return unparseable data.
- No retry, quality-control, video-editing, voice-generation, or image-generation steps are included — this pipeline stops at script + image-prompt CSV.
- The Google Gemini model name (`models/gemini-3.5-flash-lite`) is hard-coded in each model node's parameters. If that model isn't available to your account/region, update the `modelName` field in each of the four Gemini nodes.
- The chapter-detection regex in "Split Chapters" expects the Outline Agent's output to use recognizable chapter headings (e.g. `Chapter 1: Title`, `1. Title`, `## Chapter 1: Title`). Unusual formatting from a different model may not split correctly.
