# YouTube Documentary Automation

An **n8n-based AI workflow** that transforms a single video title into a complete long-form documentary script and a scene-by-scene image prompt CSV.

The workflow combines **AI agents, SearXNG research, chapter-based writing, and automated file preparation** to streamline the pre-production process for long-form YouTube documentaries.

---

## ✨ What It Does

Provide a single documentary topic, and the workflow handles the content-production pipeline:

```text
Video Title
    ↓
Prompt Library
    ↓
SearXNG Research
    ↓
Outline Agent
    ↓
Chapter Splitting
    ↓
Chapter Writers
    ↓
Chapter Collection
    ↓
Final Script Writer
    ↓
Image Prompt Generator
    ↓
Project File Preparation
    ↓
Final Output
```

### Output

Each workflow run produces:

* **`Final_Script.txt`** — A complete long-form narration script organized into chapters.
* **`Image_Prompt.csv`** — Scene-by-scene AI image prompts covering the visual requirements of the script.

The workflow is designed as a **content pre-production pipeline**. It does not generate the images, video, or voiceover itself, allowing the outputs to be connected to separate downstream automation.

---

## 🧠 AI Pipeline

The workflow uses multiple specialized stages rather than generating the entire documentary in a single AI request.

### 1. Research

The topic is researched through a self-hosted **SearXNG** instance to provide source material for the writing agents.

### 2. Outline Generation

An AI agent converts the research into a structured documentary outline.

### 3. Chapter Writing

The outline is split into individual chapters, which are then processed by dedicated chapter-writing agents.

This allows longer documentaries to be generated without relying on a single massive AI request.

### 4. Final Script

The completed chapters are collected and passed to a final script-writing stage that prepares the finished narration.

### 5. Image Prompt Generation

The final script is analyzed to produce scene-by-scene image prompts that can be sent to an external image-generation pipeline.

---

## 🛠️ Requirements

### Required

* [n8n](https://n8n.io/) — self-hosted
* Docker
* Google Gemini API credentials
* SearXNG
* JSON output enabled in SearXNG
* n8n LangChain / AI nodes

### Optional / Downstream

The generated files can be connected to your own:

* AI image-generation workflow
* Voiceover / TTS pipeline
* FFmpeg video editor
* Automated YouTube production system

See [`DEPENDENCIES.md`](DEPENDENCIES.md) for the complete breakdown.

---

## 🚀 Quick Start

### 1. Clone the repository

```bash
git clone <this-repo-url>
cd youtube-documentary-automation
```

### 2. Start the services

```bash
docker compose up -d
```

### 3. Configure SearXNG

Enable JSON output in the SearXNG `settings.yml`:

```yaml
search:
  formats:
    - html
    - json
```

Restart the SearXNG service after making the change.

### 4. Open n8n

Open:

```text
http://localhost:5678
```

Import:

```text
youtube-documentary-automation.json
```

### 5. Configure Gemini

Attach your own Google Gemini credentials to the Gemini chat model nodes in the workflow.

### 6. Run the workflow

Open the **When Chat Message Received** interface and provide a documentary topic.

For example:

```text
The Salem Witch Trials
```

The workflow will then perform the research, generate the outline, write the chapters, assemble the final script, and create the image-prompt CSV.

---

## 📁 Repository Structure

```text
youtube-documentary-automation/
│
├── youtube-documentary-automation.json
├── docker-compose.yml
├── README.md
├── PUBLIC_SETUP.md
├── DEPENDENCIES.md
└── LICENSE
```

| File                                  | Purpose                                          |
| ------------------------------------- | ------------------------------------------------ |
| `youtube-documentary-automation.json` | Importable n8n workflow                          |
| `docker-compose.yml`                  | Docker configuration for n8n + SearXNG           |
| `PUBLIC_SETUP.md`                     | Detailed setup and configuration guide           |
| `DEPENDENCIES.md`                     | Required, optional, and unnecessary dependencies |
| `LICENSE`                             | Project license                                  |

---

## 📤 Example Output

### Final Script

```text
Final_Script.txt
```

Contains the complete documentary narration generated from the research and chapter-writing pipeline.

### Image Prompts

```text
Image_Prompt.csv
```

Contains the visual prompts required to illustrate the documentary scene by scene.

These files can then be passed to separate automation for image generation, voiceover, and video editing.

---

## 🔗 Designed for Automation

This project is intentionally modular.

The workflow focuses on the **research → writing → visual planning** stages while leaving image generation, voiceover, and video editing to downstream systems.

This makes it possible to connect the workflow to different tools without changing the core documentary-writing pipeline.

---

## ⚠️ Notes

* You must provide your own API credentials.
* API credentials should **never be committed to the repository**.
* SearXNG must have JSON output enabled for the research step.
* The workflow is designed for a self-hosted n8n environment.
* Generated scripts and prompts will vary depending on the selected AI models, prompts, research results, and topic.

---

## 📚 Documentation

* [`PUBLIC_SETUP.md`](PUBLIC_SETUP.md) — Complete installation and configuration guide
* [`DEPENDENCIES.md`](DEPENDENCIES.md) — Dependency reference

---

## 📄 License

MIT License — see [`LICENSE`](LICENSE) for details.
