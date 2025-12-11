# AI Plugin for nb

An AI-powered plugin for [`nb`](https://github.com/xwmx/nb) that adds semantic search and question-answering capabilities using OpenAI (or compatible APIs).

## Features

- **🔍 Semantic Search** - Search notes by meaning, not just keywords
- **💬 Ask Questions** - Get AI-powered answers based on your notes (RAG)
- **📝 Summarization** - Generate summaries of individual notes or collections
- **🗣️ Interactive Chat** - Have a conversation about your notes
- **⚡ Efficient Indexing** - Embeddings are stored in a persistent index for fast searches

## Requirements

- [`nb`](https://github.com/xwmx/nb) (note-taking CLI)
- `curl` (usually pre-installed)
- `jq` (JSON processor)
- OpenAI API key (or compatible API)

### Installing jq

```bash
# macOS
brew install jq

# Ubuntu/Debian
sudo apt install jq

# Fedora
sudo dnf install jq

# Arch Linux
sudo pacman -S jq
```

## Installation

### Option 1: Install from GitHub (Recommended)

```bash
nb plugins install https://github.com/xwmx/nb/blob/master/plugins/ai.nb-plugin
```

### Option 2: Install from Local File

```bash
# If you have the plugin file locally
nb plugins install /path/to/ai.nb-plugin
```

### Option 3: Manual Installation

```bash
# Copy the plugin to your nb plugins directory
cp ai.nb-plugin ~/.nb/.plugins/
```

### Verify Installation

```bash
nb plugins
# Should show: ai.nb-plugin

nb help ai
# Should display the AI plugin help
```

## Configuration

### Set Your OpenAI API Key

You must configure an API key before using the plugin.

**Option 1: Environment Variable (Recommended)**

Add to your `~/.bashrc`, `~/.zshrc`, or shell config:

```bash
export NB_OPENAI_API_KEY="sk-your-api-key-here"
```

**Option 2: Using nb config**

```bash
nb ai config set api_key "sk-your-api-key-here"
```

### Optional Settings

```bash
# Change the default chat model (default: gpt-4o-mini)
nb ai config set model "gpt-4o"

# Change the embedding model (default: text-embedding-3-small)
nb ai config set embedding_model "text-embedding-3-large"

# Use a custom API endpoint (for OpenAI-compatible APIs)
nb ai config set base_url "https://your-api-endpoint.com/v1"

# Index scope: all notebooks or only the current one (default: all)
nb ai config set scope "all"      # or "current"
```

### View Current Configuration

```bash
nb ai config show
```

Output:
```
AI Configuration:
  api_key:         sk-abcd...wxyz
  base_url:        https://api.openai.com/v1
  model:           gpt-4o-mini
  embedding_model: text-embedding-3-small
```

## Usage

### Building the Index (Required First Step)

Before using semantic search or ask features, you need to build the embeddings index:

```bash
# Build the index (only embeds new/changed notes)
nb ai index

# Check index status
nb ai index --status

# Force rebuild entire index
nb ai index --rebuild
```

**Scope:** By default, the index includes *all* of your notebooks. To limit to only the current notebook, set `nb ai config set scope current` (or pass `nb ai index --scope current`). To force all notebooks regardless of config, use `nb ai index --all`.

**The index is stored in `~/.nb/.ai_index.json` and contains:**
- Embeddings for each note (vector representation)
- File hashes for change detection
- Note titles for display

**When to rebuild:**
- After adding many new notes: `nb ai index`
- After the index command shows changes: `nb ai index`
- If search results seem off: `nb ai index --rebuild`

### Semantic Search

Search your notes by meaning rather than exact text matching:

```bash
# Basic search
nb ai search "meeting notes about project planning"

# Limit results
nb ai search "docker configuration" -n 10

# Adjust similarity threshold (0.0-1.0, higher = stricter)
nb ai search "budget analysis" --threshold 0.8
```

**Example Output:**
```
Searching notes for: meeting notes about project planning

Results:
─────────────────────────────────────────

[0.89] meetings/2024-01-15-planning.md
   Q1 Project Planning Meeting

[0.82] projects/roadmap.md
   2024 Product Roadmap

[0.76] notes/standup-notes.md
   Weekly Standup Notes
```

Search runs against all indexed notebooks by default. To limit searches to the current notebook, set `nb ai config set scope current`.

### Ask Questions (RAG)

Ask natural language questions and get AI-generated answers based on your notes:

```bash
# Ask a question
nb ai ask "What were the action items from last week's meeting?"

# Use a specific model
nb ai ask "Summarize the project timeline" --model gpt-4o

# Include more context notes
nb ai ask "What is our deployment process?" -n 10
```

By default, questions are answered using notes from *all* indexed notebooks. Set `nb ai config set scope current` if you prefer to restrict to the current notebook.

**Example Output:**
```
Question: What were the action items from last week's meeting?

Finding relevant notes...
Generating answer...

Answer:
─────────────────────────────────────────
Based on your meeting notes from January 15th, the action items were:

1. **John** - Complete the API documentation by Friday
2. **Sarah** - Review the security audit findings
3. **Team** - Update sprint board with new priorities

The deadline for all items is January 22nd.

Sources:
  • meetings/2024-01-15-planning.md
  • projects/sprint-23.md
```

### Summarize Notes

Generate AI summaries of your notes:

```bash
# Summarize a specific note
nb ai summarize 42
nb ai summarize meetings/2024-01-15.md

# Summarize recent notes (last 10)
nb ai summarize
```

### Interactive Chat

Start a conversation about your notes:

```bash
nb ai chat
```

**Example Session:**
```
AI Chat Mode
Chat with AI about your notes. Type 'exit' or 'quit' to end.
─────────────────────────────────────────

You: What projects am I currently working on?

AI: Based on your notes, you're currently working on three main projects:

1. **Website Redesign** - In the design review phase
2. **API Migration** - Currently at 60% completion
3. **Mobile App** - Just started planning

You: Tell me more about the API migration

AI: The API migration project involves moving from REST to GraphQL...

You: exit

Goodbye!
```

### Quick Ask (Auto-detect)

If your command looks like a question, it's automatically treated as an ask:

```bash
# These are equivalent:
nb ai "What is the project deadline?"
nb ai ask "What is the project deadline?"
```

## Command Reference

```
nb ai index [--rebuild] [--status]
nb ai ask <question> [--model <model>] [-n <num>]
nb ai search <query> [-n <num>] [--threshold <0.0-1.0>]
nb ai summarize [<selector>]
nb ai chat [--model <model>]
nb ai config [show | set <key> <value>]
```

### Options

| Option | Description | Default |
|--------|-------------|---------|
| `--model <model>` | AI model for chat/ask | `gpt-4o-mini` |
| `-n, --num <number>` | Number of results/context notes | `5` |
| `--threshold <float>` | Minimum similarity score (0.0-1.0) | `0.5` |
| `--rebuild` | Force rebuild entire index | - |
| `--status` | Show index status without updating | - |

### Aliases

| Command | Aliases |
|---------|---------|
| `nb ai index` | `nb ai idx`, `nb ai i` |
| `nb ai ask` | `nb ai a` |
| `nb ai search` | `nb ai s`, `nb ai find` |
| `nb ai summarize` | `nb ai sum` |
| `nb ai chat` | `nb ai c` |
| `nb ai config` | `nb ai conf`, `nb ai cfg` |

## How It Works

### Index-Based Architecture

The plugin uses an efficient index-based approach for fast searches:

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│   Your Notes    │────▶│   nb ai index    │────▶│  .ai_index.json │
│  (*.md, *.txt)  │     │  (one-time)      │     │  (embeddings)   │
└─────────────────┘     └──────────────────┘     └─────────────────┘
                                                         │
┌─────────────────┐     ┌──────────────────┐            │
│  Search Query   │────▶│  nb ai search    │◀───────────┘
│                 │     │  (instant)       │
└─────────────────┘     └──────────────────┘
```

**Index Benefits:**
- **Build once**: Embeddings computed once per note, stored persistently
- **Fast search**: No API calls needed during search (only for query embedding)
- **Incremental updates**: Only new/changed notes are re-embedded
- **Single file**: All embeddings stored in one JSON file for easy backup

### Semantic Search

1. Your query is converted to an embedding vector (single API call)
2. The pre-built index is loaded from disk (no API calls)
3. Cosine similarity is calculated between query and all note embeddings
4. Notes above the similarity threshold are returned, sorted by relevance

### Question Answering (RAG)

1. Your question is embedded and compared against the index
2. The content of top matching notes is loaded and included as context
3. The AI generates an answer based solely on the provided context
4. Sources are cited so you can verify the information

### Index Storage

- Index file: `~/.nb/.ai_index.json`
- Contains: embeddings, file hashes, titles, metadata
- Automatically detects changed files via content hash
- Run `nb ai index --status` to see what needs updating

## Using with Other AI Providers

The plugin supports any OpenAI-compatible API:

### Azure OpenAI

```bash
nb ai config set base_url "https://your-resource.openai.azure.com/openai/deployments/your-deployment"
nb ai config set api_key "your-azure-api-key"
```

### Local LLMs (Ollama, LM Studio, etc.)

```bash
# For Ollama with OpenAI compatibility
nb ai config set base_url "http://localhost:11434/v1"
nb ai config set model "llama2"
```

### Anthropic (via proxy)

Use an OpenAI-compatible proxy service that supports Anthropic models.

## Troubleshooting

### "jq is required but not installed"

Install jq using your package manager (see Requirements section).

### "OpenAI API key not configured"

Set your API key using one of the methods in the Configuration section.

### "No embeddings index found"

You need to build the index before searching:

```bash
nb ai index
```

### "API request failed (HTTP 401)"

Your API key is invalid or expired. Check your key at https://platform.openai.com/api-keys

### "API request failed (HTTP 429)"

Rate limit exceeded. Wait a moment and try again, or upgrade your OpenAI plan.

### Index building is slow

- First-time indexing processes all notes (expected to take time)
- Subsequent `nb ai index` runs only process new/changed notes
- Consider indexing in batches for very large collections

### Notes not appearing in search

- Run `nb ai index` to update the index with new notes
- Check `nb ai index --status` to see if notes need indexing
- Ensure notes have `.md`, `.txt`, `.org`, `.markdown`, or `.rst` extensions
- Binary files and images are not indexed
- Files in `.git` directories are excluded

### Search results seem wrong

Try lowering the similarity threshold:

```bash
nb ai search "your query" --threshold 0.3
```

Or rebuild the index:

```bash
nb ai index --rebuild
```

## Privacy & Security

- API keys are stored in `~/.nb/.ai_config` with restricted permissions (600)
- Note content is sent to OpenAI (or your configured provider) only during indexing
- Embeddings index (`~/.nb/.ai_index.json`) is stored locally
- During search, only the query is sent to the API (not your notes)
- Consider using a local LLM if privacy is a concern

## License

This plugin is part of nb and is licensed under the AGPL-3.0 license.

## Contributing

Issues and pull requests are welcome at https://github.com/xwmx/nb

