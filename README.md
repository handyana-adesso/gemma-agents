# GemmaAgents

A progressive, hands-on lab for building AI agents in **.NET** powered by
**Gemma 4** running locally via **Ollama**. The solution is organized as three
difficulty tiers that build on each other, sharing common code through a single
library.

Nothing here calls a hosted model by default — the agents talk to Gemma 4 on
your own machine through Ollama's OpenAI-compatible API.

## Structure

```
GemmaAgents/
├─ GemmaAgents.sln
├─ README.md
├─ .gitignore
├─ src/
│  ├─ GemmaAgents.Shared/              Class library: model-client setup + reusable tools.
│  │                              NuGet packages are pinned here; the apps inherit them.
│  ├─ GemmaAgents.LocalDevHelper/             Project 1 (easy) — a CLI agent over your project folder.
│  └─ GemmaAgents.GroundedSupportAgent/       Project 2 (medium) — RAG + memory + evaluation.
└─ python/                        Project 3 (hard) — fine-tuning + serving scripts (not a .csproj).
```

Project 3 has no C# project of its own: training and serving are Python work,
and the .NET side is simply Project 2 **repointed** at the fine-tuned model's
endpoint — a configuration change, not new code.

## The three projects

| Tier | Project | What it teaches |
|------|---------|-----------------|
| Easy | `LocalDevHelper` | The core agent loop and tool calling. The model decides when to read files, list folders, or check the time. |
| Medium | `GroundedSupportAgent` | Retrieval-as-a-tool (RAG), conversation memory, graceful tool-failure handling, loop limits, and a repeatable evaluation set. |
| Hard | `python/` | LoRA fine-tuning of a small Gemma 4 variant and self-hosting it with vLLM, then pointing the medium agent at it. |

## Prerequisites

1. **.NET 8 SDK or newer** — `dotnet --version`.
2. **Ollama** with a Gemma 4 model pulled:

   ```bash
   ollama pull gemma4:12b     # reliable at tool calling (~16GB VRAM)
   ollama serve               # OpenAI-compatible API on :11434
   ```

   Lighter hardware? Use `gemma4:e4b`, but expect weaker tool selection.

3. For Project 3 only: Python 3.10+, a GPU (or Colab/Vertex AI), and the
   packages listed in `python/requirements.txt`.

## Build and run

Restore and build the whole solution:

```bash
dotnet build
```

Run a specific project:

```bash
dotnet run --project src/GemmaAgents.LocalDevHelper
dotnet run --project src/GemmaAgents.GroundedSupportAgent
```

## Configuration

The agents read these environment variables (all optional):

| Variable          | Default                     | Meaning                          |
|-------------------|-----------------------------|----------------------------------|
| `OLLAMA_MODEL`    | `gemma4:12b`                | model name from `ollama list`    |
| `OLLAMA_ENDPOINT` | `http://localhost:11434/v1` | OpenAI-compatible URL            |
| `WORKSPACE_DIR`   | current directory           | folder the agent may read        |

To run Project 2 against a fine-tuned model from Project 3, point
`OLLAMA_ENDPOINT` at your vLLM server instead — no code change needed.

## A note on packages

The `Microsoft.Agents.AI*` packages ship new versions frequently. They are
pinned to a coherent, co-versioned pair in `src/Agents.Shared/Agents.Shared.csproj`
— keep the two on the **same** version. If `dotnet restore` fails or you want
the latest:

```bash
dotnet add src/GemmaAgents.Shared package Microsoft.Agents.AI
dotnet add src/GemmaAgents.Shared package Microsoft.Agents.AI.OpenAI
```

The API shape (`GetChatClient → AsIChatClient → CreateAIAgent`) has been stable
through the 1.x line; if a method stops compiling after an upgrade, check
https://learn.microsoft.com/agent-framework/.

## License

Add your own. Gemma 4 itself is Apache 2.0; the Microsoft Agent Framework is MIT.