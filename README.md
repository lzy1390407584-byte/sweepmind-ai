# SweepMind AI

**RAG-Powered Intelligent Agent for Robot Vacuum Cleaners**

SweepMind AI is a robot-vacuum intelligent assistant built with LangChain, RAG, Chroma and Streamlit. It combines a domain knowledge base with tool-calling agents so users can ask product, maintenance and troubleshooting questions in natural language, receive grounded answers, and generate personalized usage reports.

## Highlights

- RAG question answering over robot-vacuum knowledge documents.
- ReAct-style agent with multiple tools and iterative tool calls.
- Knowledge retrieval from TXT and PDF files using Chroma.
- Qwen chat model and DashScope embedding model integration.
- Dynamic prompt switching for normal Q&A and usage-report generation.
- Tool-call monitoring and model-call logging through LangChain middleware.
- Streamlit chat interface with conversational history.
- Synthetic external usage records for personalized monthly reports.

## Main capabilities

### Domain Q&A

The assistant retrieves relevant content from documents covering product selection, daily use, maintenance, troubleshooting, floor types, pets, humidity and other robot-vacuum scenarios. Answers are summarized from retrieved context instead of relying only on model memory.

### Tool calling

The agent can select tools according to the user’s intent:

- `rag_summarize`: retrieve and summarize domain knowledge.
- `get_weather`: obtain environment information for a city.
- `get_user_location`: obtain a simulated user location.
- `get_user_id`: obtain a simulated user identifier.
- `get_current_month`: obtain a report month.
- `fetch_external_data`: retrieve a user’s monthly usage record.
- `fill_context_for_report`: switch the agent into report-generation context.

### Personalized reports

When a user asks for a usage report, the agent follows a dedicated workflow: identify the user, determine the month, retrieve usage data, obtain relevant maintenance knowledge, and produce a Markdown report with practical recommendations.

## Architecture

```text
User
 │
 ▼
Streamlit chat UI
 │
 ▼
LangChain Agent / ReAct workflow
 ├── RAG tool ──► Chroma vector store ──► TXT / PDF knowledge base
 ├── Weather and user-context tools
 ├── External usage-data tool ──► records.csv
 └── Middleware ──► tool monitoring / logging / dynamic prompts
 │
 ▼
Qwen chat model + DashScope embeddings
```

## Project structure

```text
.
├── app.py                         # Streamlit application entry point
├── agent/
│   ├── react_agent.py              # Agent construction and streaming execution
│   └── tools/
│       ├── agent_tools.py          # RAG, weather, user and report tools
│       └── middleware.py           # Tool monitoring and dynamic prompt switching
├── rag/
│   ├── rag_service.py              # Retrieval + prompt + model chain
│   └── vector_store.py             # Chroma loading, splitting and MD5 deduplication
├── model/
│   └── factory.py                  # Chat and embedding model factories
├── config/
│   ├── agent.yml                   # External usage-data path
│   ├── chroma.yml                  # Vector store and chunking configuration
│   ├── prompts.yml                 # Prompt file locations
│   └── rag.yml                     # Model names
├── prompts/                        # System, RAG and report prompts
├── data/                           # Robot-vacuum knowledge base and usage records
└── utils/                          # Configuration, file, path and logging utilities
```

## Technology stack

- Python 3.10+
- Streamlit
- LangChain Core / Agents / Community
- LangGraph middleware
- Chroma
- PyPDF
- Qwen via DashScope
- YAML configuration

## Quick start

### 1. Create an environment

```bash
conda create -n sweepmind python=3.10
conda activate sweepmind
```

### 2. Install dependencies

```bash
pip install streamlit langchain langchain-core langchain-community langchain-chroma langchain-text-splitters langgraph pyyaml dashscope pypdf
```

The exact versions should be pinned in a `requirements.txt` file before production deployment.

### 3. Configure model access

The project uses DashScope-backed Qwen chat and embedding models. Configure the required DashScope credentials through the environment variables supported by the installed LangChain integration. Do not place API keys directly in source code or commit them to Git.

The model names can be changed in:

```text
config/rag.yml
```

### 4. Build or refresh the vector store

The knowledge files are stored in `data/`. The vector-store service supports TXT and PDF files, splits documents into chunks, and records file MD5 values in `md5.text` to avoid duplicate ingestion.

Run the vector-store module when the knowledge base needs to be built or refreshed:

```bash
python -m rag.vector_store
```

### 5. Start the application

```bash
streamlit run app.py
```

Then open the local URL printed by Streamlit, usually `http://localhost:8501`.

## Configuration

Important settings are located in `config/`:

- `config/rag.yml`: chat and embedding model names.
- `config/chroma.yml`: collection name, persistence directory, retrieval count and text-splitting parameters.
- `config/agent.yml`: path to external usage records.
- `config/prompts.yml`: prompt file locations.

The default external data file is:

```text
data/external/records.csv
```

## Engineering notes

### Grounded responses

The RAG chain retrieves the top-k relevant chunks from Chroma and injects them into a controlled summarization prompt. This keeps domain answers focused on the project’s reference material.

### Dynamic prompts

The middleware checks runtime context and switches between the general customer-service prompt and the report-writing prompt after the report-context tool is called.

### Duplicate-ingestion protection

Before adding documents to Chroma, the vector-store service calculates each file’s MD5 value. Processed hashes are stored locally so restarting the application does not repeatedly insert the same documents.

### Observability

Tool calls, parameters, successes and failures are recorded through the logging middleware. This makes the agent workflow easier to inspect and debug.

## Security and privacy

- Keep DashScope API keys in environment variables or a secret manager.
- Do not commit real user data, credentials, logs containing sensitive information or private vector-store artifacts.
- Replace the demo weather and user-context tools with authenticated production services before real deployment.
- Review the access policy of any external data source before connecting it to the agent.

## Roadmap

- Add a pinned `requirements.txt`.
- Add automated tests for tools, retrieval and report workflows.
- Add source citations to RAG answers.
- Replace simulated weather and user data with service integrations.
- Add Docker Compose deployment.
- Improve conversation persistence and multi-user session isolation.

## License

No open-source license has been declared yet. Add a `LICENSE` file before accepting external contributions or permitting redistribution.

