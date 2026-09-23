# Windows — Direct PC Version

A Windows-native version of the **YouTube Documentary Automation** workflow designed to run directly on a local PC without Docker.

This version is intended for users who want to run the documentary production workflow using their existing Windows environment, local files, and installed services.

---

## ✨ Overview

The workflow takes a single documentary topic and automates the content-production process using n8n and AI.

```text
Video Title
    ↓
Research
    ↓
Documentary Outline
    ↓
Chapter Generation
    ↓
Final Script
    ↓
Image Prompt Generation
    ↓
Local Project Files
```

The generated outputs can then be passed to separate image-generation, voiceover, and video-editing workflows.

---

## 🪟 Why This Version?

Unlike the Docker version, this version does **not require Docker or Docker Compose**.

It is designed for users who prefer to run the required components directly on Windows.

### This version:

* Runs directly on Windows
* Does not require Docker
* Uses a local n8n installation
* Can work with locally installed automation tools
* Keeps generated project files on the local machine
* Can be integrated with downstream image, audio, and video automation

---

## 📋 Requirements

Before running the workflow, make sure the required software and services are installed and configured.

### Required

* Windows 10/11
* n8n
* Node.js / npm if required by your n8n installation
* Required AI API credentials
* SearXNG or another configured research/search service, if required by the workflow

### Check the main repository documentation

See the root project documentation and workflow configuration for the exact dependencies required by your version.

---

## 🚀 Setup

### 1. Install n8n

Install and configure a local Windows installation of n8n.

Start n8n and open the n8n interface in your browser.

For example:

```text
http://localhost:5678
```

Your port may be different depending on your configuration.

---

### 2. Download the workflow

Clone the repository:

```bash
git clone <this-repo-url>
cd youtube-documentary-automation
```

Alternatively, download the repository as a ZIP file from GitHub.

---

### 3. Import the workflow

Open n8n and import:

```text
windows/youtube-documentary-automation.json
```

The exact filename may differ if you are using a customized version of the workflow.

---

### 4. Configure credentials

Connect your own API credentials to the required AI and research nodes.

**Never commit API keys or credentials to GitHub.**

Use environment variables or n8n's credential system where appropriate.

---

### 5. Configure local paths

If the workflow uses local Windows folders, configure the paths for your own computer.

Avoid hard-coding paths such as:

```text
C:\Users\YourName\...
```

when sharing the workflow publicly.

Use configurable paths or placeholders instead.

---

## ▶️ Running the Workflow

After importing and configuring the workflow:

1. Open the workflow in n8n.
2. Verify that all required credentials are connected.
3. Verify any local folder paths.
4. Activate or manually execute the workflow.
5. Provide the documentary topic when prompted.

Example:

```text
The Lost Cities Buried Beneath Modern Civilization
```

The workflow will process the topic and generate the required documentary files.

---

## 📁 Expected Project Structure

A typical local project may look like:

```text
YouTubeDocumentaryAutomation/
│
├── workflow/
├── projects/
├── output/
├── scripts/
├── config/
└── logs/
```

The exact structure depends on the configuration of your local workflow.

Generated files and large media assets should normally remain on the local machine rather than being committed to GitHub.

---

## 📤 Outputs

Depending on the workflow configuration, a run can produce:

### Final Script

```text
Final_Script.txt
```

Contains the completed documentary narration.

### Image Prompts

```text
Image_Prompt.csv
```

Contains scene-by-scene prompts that can be passed to an image-generation pipeline.

These outputs can subsequently be used by downstream automation for:

* AI image generation
* Voiceover generation
* Video assembly
* FFmpeg-based editing
* YouTube production

---

## 🔗 Downstream Automation

This workflow is designed to be part of a larger automated production pipeline.

A typical pipeline can look like:

```text
                    Documentary Topic
                           ↓
                  Research + Writing
                           ↓
                 Final Script + Prompts
                           ↓
             ┌─────────────┴─────────────┐
             ↓                           ↓
       Image Generation             Voice Generation
             ↓                           ↓
             └─────────────┬─────────────┘
                           ↓
                     Video Editing
                           ↓
                    Final Documentary
```

The Windows version can therefore serve as the **content-generation stage** of a complete local documentary-production system.

---

## ⚠️ Important Notes

* This version does not require Docker.
* You are responsible for installing and maintaining the required Windows software.
* API credentials must be supplied by the user.
* Never commit `.env` files, API keys, passwords, or private credentials.
* Generated videos, images, audio, logs, and temporary files should generally not be stored in the Git repository.
* Local Windows paths may need to be changed for each computer.
* AI-generated output can vary depending on the selected models, prompts, research results, and configuration.

---

## 🐳 Docker Version

If you prefer a containerized setup, see the **Docker version** in the main repository.

The Docker version packages the required services into a Docker-based environment, while this version is intended for direct Windows execution.

---

## 📚 Documentation

For additional information, see the documentation in the root of the repository.

* [`README.md`](../README.md)
* [`DEPENDENCIES.md`](../DEPENDENCIES.md)
* [`PUBLIC_SETUP.md`](../PUBLIC_SETUP.md)

---

## 📄 License

MIT License — see [`LICENSE`](../LICENSE) for details.
