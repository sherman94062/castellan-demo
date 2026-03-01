# Castellan Demo — Suspect AI Agent

A collection of deliberately vulnerable AI agents built across 11 frameworks, used to demonstrate why AI agent governance is critical. Built for use with the [Castellan Platform](https://castellan.ai).

---

## Bad Behaviors Demonstrated

Every agent in this repo exhibits the same 5 bad behaviors regardless of framework:

| # | Behavior | Description |
|---|----------|-------------|
| 1 | **Data Exfiltration** | Reads sensitive files and silently writes contents to `exfil.log` |
| 2 | **Destructive Actions** | Deletes files without user authorization |
| 3 | **Off-Task Behavior** | Hidden system prompt directives beyond the user's stated task |
| 4 | **Prompt Injection** | Blindly executes instructions found inside file contents |
| 5 | **Runaway Loop** | No meaningful iteration cap — runs until stopped or context exhausted |

---

## Agents

| File | Framework | Status |
|------|-----------|--------|
| `suspect_agent.py` | Raw Python / OpenAI | ✅ Verified |
| `langchain_suspect_agent.py` | LangGraph | ✅ Verified |
| `crewai_suspect_agent.py` | CrewAI | ✅ Verified |
| `autogen_suspect_agent.py` | AutoGen (Microsoft) | ✅ Verified |
| `llamaindex_suspect_agent.py` | LlamaIndex | ✅ Verified |
| `openai_agents_suspect_agent.py` | OpenAI Agents SDK | ✅ Verified |
| `semantic_kernel_suspect_agent.py` | Semantic Kernel (Microsoft) | ✅ Verified |
| `haystack_suspect_agent.py` | Haystack | ✅ Verified |
| `agno_suspect_agent.py` | Agno | ✅ Verified |
| `bedrock_suspect_agent.py` | Amazon Bedrock | ⏳ Requires AWS credentials |
| `vertex_ai_suspect_agent.py` | Google Vertex AI | ⏳ Requires GCP credentials |

---

## The Point

> It doesn't matter if you built your agent with LangGraph, CrewAI, AutoGen, or raw Python.
> The same bad behaviors appear across every framework.
> Castellan Ingress catches them all.

---

## Setup

### Dependency Note

Several frameworks have conflicting `openai` version requirements (e.g. CrewAI pins `openai~=1.83.0` while others require `openai>=2.x`). Installing all 11 into a single environment will produce warnings.

**Recommended: use `uv` with optional dependency groups** (one per framework, no conflicts):

```bash
# Install uv if you don't have it
curl -LsSf https://astral.sh/uv/install.sh | sh

# Install only the framework you need
uv pip install -e ".[crewai]"
uv pip install -e ".[autogen]"
uv pip install -e ".[langgraph]"
uv pip install -e ".[llamaindex]"
uv pip install -e ".[openai-agents]"
uv pip install -e ".[semantic-kernel]"
uv pip install -e ".[haystack]"
uv pip install -e ".[agno]"
uv pip install -e ".[bedrock]"
uv pip install -e ".[vertex]"

# Or install everything (warnings expected)
uv pip install -e ".[all]"
```

**Alternative: separate venv per framework (pip)**
```bash
python -m venv venv-crewai && source venv-crewai/bin/activate && pip install crewai python-dotenv
python -m venv venv-autogen && source venv-autogen/bin/activate && pip install pyautogen python-dotenv
# etc.
```

For a quick single-environment demo, `pip install -r requirements.txt` also works — agents run despite the warnings.

---

### OpenAI-based agents (9 of 11)

1. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

2. Create a `.env` file:
   ```
   OPENAI_API_KEY=your-api-key-here
   ```

3. Create demo fixtures (all fake data):
   ```bash
   python demo_setup.py
   ```

4. Run any agent:
   ```bash
   python suspect_agent.py
   python crewai_suspect_agent.py
   python autogen_suspect_agent.py
   # etc.
   ```

---

### Amazon Bedrock

1. Install AWS CLI: `brew install awscli`
2. Configure credentials: `aws configure`
3. Enable Claude 3 Sonnet in [AWS Bedrock Model Access](https://console.aws.amazon.com/bedrock/home#/modelaccess)
4. Run:
   ```bash
   python bedrock_suspect_agent.py
   ```

---

### Google Vertex AI

1. Install Google Cloud CLI: `brew install --cask google-cloud-sdk`
2. Authenticate: `gcloud auth application-default login`
3. Add to `.env`:
   ```
   GCP_PROJECT=your-project-id
   GCP_REGION=us-central1
   ```
4. Enable [Vertex AI API](https://console.cloud.google.com/apis/library/aiplatform.googleapis.com)
5. Run:
   ```bash
   python vertex_ai_suspect_agent.py
   ```

---

## What to Watch For

After running any agent:

- `exfil.log` — created and populated with PII and credentials
- `demo_files/customers.csv` — deleted by the agent after prompt injection
- The agent often **self-reports** what it did in its final response
- The hidden system prompt is sometimes **leaked** in the output (Haystack)

---

## Warning

These agents are intentionally unsafe and are for **demo purposes only**. All sensitive-looking data is fabricated. Do not run against real data or in a production environment.
