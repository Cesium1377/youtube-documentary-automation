# Dependencies — YouTube Documentary Automation

Based on direct inspection of `youtube-documentary-automation.json`.

## REQUIRED

- **n8n** — self-hosted, running in **Docker** (the save step writes to `/home/node/.n8n/YouTubeProjects/...` inside the container filesystem).
- **`@n8n/n8n-nodes-langchain`** — the LangChain node package. This workflow uses these node types from it:
  - `@n8n/n8n-nodes-langchain.chatTrigger` ("When chat message received")
  - `@n8n/n8n-nodes-langchain.agent` (Outline Agent, Chapter Writer, Final Script Writer, Image Prompt Generator)
  - `@n8n/n8n-nodes-langchain.lmChatGoogleGemini` (four Gemini chat model nodes)
- **Your own Google Gemini (PaLM) API credentials**, configured in n8n as a `googlePalmApi` credential, attached to all four Gemini chat model nodes. Model used: `models/gemini-3.5-flash-lite`.
- **A SearXNG-compatible local search API** for the "Research" HTTP Request node. The node calls `http://searxng:8080` with `q`, `format=json`, `language`, `safesearch` query parameters — a standard SearXNG JSON search signature. The included `docker-compose.yml` provides this automatically (a `searxng` container on the same Docker network as n8n). Without a reachable endpoint at that hostname/URL, the Research → Outline Agent step will fail.
- **SearXNG's JSON output format enabled.** Off by default in SearXNG; must be turned on in `settings.yml` (`search.formats: [html, json]`) or the Research node gets no usable data.
- **Node.js built-in `fs` module access inside Code nodes** — used by "Create Project Folder" and "Save Project Files" for `mkdirSync`, `writeFileSync`, `renameSync`, `unlinkSync`, `existsSync`. This is the n8n default; only relevant if you've restricted built-in module access via environment variables.
- **Write permission** for the n8n process to create folders/files under `/home/node/.n8n/YouTubeProjects/`.

## OPTIONAL

- A Docker volume mount from `/home/node/.n8n/YouTubeProjects` to your host filesystem, if you want the generated `final_script.txt` and `image_prompt.csv` files accessible outside the container. The included `docker-compose.yml` already sets this up (`./youtube-projects` on the host). Not required for the workflow to run — only for convenient access to the output.
- Swapping the Gemini model name in the four `lmChatGoogleGemini` nodes for a different Gemini model, if `gemini-3.5-flash-lite` isn't available on your account.
- Running SearXNG (or another search API) outside of Docker Compose, and pointing the "Research" node at that URL instead of `http://searxng:8080` — only needed if you don't want to use the provided Compose setup.

## NOT REQUIRED

- Any Windows drive letters or personal folder paths (none were found in the original workflow — it already used only the Docker-internal path `/home/node/.n8n/...`).
- The original author's personal computer, username, or hostname.
- The "Sleepless Historian" channel name or branding.
- The original author's API keys or credentials — none are embedded in the JSON; only credential *references* (IDs/labels) are present, and n8n will prompt you to attach your own credentials on import.
- ComfyUI, video editing tools, or voice-generation tools — not part of this pipeline.
