# Email Agent CrewAI

A hands-on learning project exploring [CrewAI](https://www.crewai.com/), a framework for building multi-agent AI systems. This repo walks through progressively more advanced examples — from a single agent that polishes emails, to a multi-agent marketing crew that researches, plans, and drafts content autonomously.

## What's inside

The project is organized as a step-by-step progression, with each file building on the concepts of the last.

| File                            | What it demonstrates                                                                                                                                                                                                                                                         |
| ------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `1_email_agent.ipynb`           | A single CrewAI `Agent` and `Task` that rewrites a rough, casual email into a polished, professional version.                                                                                                                                                                |
| `2_email_agent_with_tool.ipynb` | The same email agent, extended with a **custom tool** that detects and explains internal jargon/abbreviations before rewriting.                                                                                                                                              |
| `3_crew.ipynb`                  | Introduces a **Crew**: two agents (a researcher and a writer) collaborating on sequential tasks, passing context between them.                                                                                                                                               |
| `4_crew_with_tools.ipynb`       | The research/writer crew, now equipped with `SerperDevTool` for live web search instead of relying only on the model's training data.                                                                                                                                        |
| `5_yaml.py`                     | Refactors the crew into a proper CrewAI **project structure**, defining agents and tasks in YAML config files (`config/agents.yaml`, `config/tasks.yaml`) rather than inline Python.                                                                                         |
| `6_memory.py`                   | Demonstrates CrewAI's memory features, allowing agents to retain context across runs.                                                                                                                                                                                        |
| `marketing_crew/`               | A larger, more realistic multi-agent crew (`crew.py`) — a "Head of Marketing," social media content creator, blog writer, and SEO specialist — that collaboratively researches a product, builds a marketing strategy, and drafts content, saving outputs as markdown files. |

## What is CrewAI?

CrewAI lets you define **agents** (each with a role, goal, and backstory), give them **tools** (web search, file read/write, website scraping, etc.), and organize them into a **crew** that executes a series of **tasks** — either one after another (sequential) or with more complex delegation. Agents can hand off work to each other, use tools to gather real information, and produce structured or free-text outputs.

This repo uses Google's **Gemini** models as the underlying LLM for every agent.

## Setup

This project uses [`uv`](https://docs.astral.sh/uv/) for Python dependency management.

### 1. Install dependencies

```bash
uv sync
```

This creates a `.venv` and installs everything listed in `pyproject.toml` / `uv.lock`.

### 2. Set up environment variables

Create a `.env` file in the project root (this file is gitignored and should **never** be committed):

```
GEMINI_API_KEY=your_gemini_api_key_here
SERPER_API_KEY=your_serper_api_key_here
```

- Get a Gemini API key from [Google AI Studio](https://aistudio.google.com/).
- Get a Serper API key from [serper.dev](https://serper.dev/) (used for the live web search tool).

### 3. Activate the environment

```bash
.venv\Scripts\Activate.ps1   # Windows PowerShell
```

### 4. Run an example

```bash
python 5_yaml.py
```

or open any of the `.ipynb` notebooks in Jupyter/VS Code and run the cells in order.

For the marketing crew:

```bash
cd marketing_crew
python crew.py
```

## Notes on the notebooks

The `.ipynb` files were originally written for Google Colab and use `google.colab.userdata` to fetch the API key. When running them locally (e.g. in VS Code), replace that with standard `.env` loading:

```python
import os
from dotenv import load_dotenv
load_dotenv()
```

## Gemini model versions

Google periodically retires older Gemini model versions. If you hit a `404` or `429` error referencing a specific model name, check [Google AI Studio](https://aistudio.google.com/) or run:

```python
from google import genai
client = genai.Client(api_key=os.environ["GEMINI_API_KEY"])
for m in client.models.list():
    for action in m.supported_actions:
        if action == "generateContent":
            print(m.name)
```

to see which models your API key currently supports, and update the `llm:` fields in the YAML configs (or `LLM(model=...)` calls) accordingly.

## A note on API quotas

The free tier of the Gemini API has daily and per-minute request limits. Crews with many agents/tasks, `reasoning=True`, or `planning=True` enabled make several LLM calls per run and can exhaust the free tier quickly. If you hit `429 RESOURCE_EXHAUSTED` errors, either wait for the quota to reset, simplify the crew for testing, or enable billing on your Google AI Studio project.
