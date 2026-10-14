# Deep Research Agent

A multi-agent AI research assistant that takes a topic, autonomously plans and runs web searches, synthesizes the findings into a detailed markdown report, and emails you the result — all from a simple web UI.

Give it a query like *"What are the latest advances in solid-state batteries?"* and it will plan targeted searches, research each one in parallel, write a structured long-form report, and deliver it straight to your inbox.

## How it works

The pipeline is built as a small crew of cooperating agents, each with a single responsibility, orchestrated by a `ResearchManager`:

```
 query
   │
   ▼
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Planner    │ --> │   Search     │ --> │   Writer     │ --> │   Email      │
│   Agent      │     │   Agent(s)   │     │   Agent      │     │   Agent      │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
 plans N web          runs searches        synthesizes a       formats report
 searches to run       concurrently,        long-form markdown  as HTML and
                       summarizes each      report + summary    sends it
```

1. **Planner agent** — reads the query and decides which web searches would best answer it.
2. **Search agent(s)** — run those searches concurrently and each return a concise summary of what it found.
3. **Writer agent** — synthesizes all the summaries into a cohesive, detailed markdown report (outline + full write-up, follow-up questions included).
4. **Email agent** — converts the report to clean HTML and sends it via SendGrid.

Status updates stream back to the UI as each stage completes, and a trace link is generated so the full run can be inspected on the OpenAI platform.

## Tech stack

- [OpenAI Agents SDK](https://github.com/openai/openai-agents-python) for agent orchestration, structured outputs, and the built-in web search tool
- [Gradio](https://www.gradio.dev/) for the web UI
- [SendGrid](https://sendgrid.com/) for sending the final report by email
- [Pydantic](https://docs.pydantic.dev/) for structured agent outputs

## Getting started

### Prerequisites

- Python 3.10+
- An [OpenAI API key](https://platform.openai.com/api-keys)
- A [SendGrid API key](https://app.sendgrid.com/settings/api_keys) with a verified sender identity

### Installation


```bash
git clone https://github.com/<your-username>/deep-research-agent.git
cd deep-research-agent

python -m venv .venv
source .venv/bin/activate   # On Windows: .venv\Scripts\activate

pip install -r requirements.txt
```

### Configuration

Copy the example environment file and fill in your own values:

```bash
cp .env.example .env
```

| Variable | Description |
| --- | --- |
| `OPENAI_API_KEY` | OpenAI API key used by all four agents |
| `SENDGRID_API_KEY` | SendGrid API key used to send the report |
| `SENDER_EMAIL` | A verified sender address in your SendGrid account |
| `RECIPIENT_EMAIL` | The inbox that should receive the finished report |

### Run it

```bash
python app.py
```

This opens a Gradio UI in your browser. Enter a research topic, hit **Run**, and watch the status updates stream in as the agents plan, search, write, and email the report.

## Project structure

```
deep-research-agent/
├── app.py                     # Gradio UI / entry point
├── deep_research/
│   ├── manager.py             # Orchestrates the end-to-end pipeline
│   └── agents/
│       ├── planner.py         # Plans the web searches to run
│       ├── search.py          # Executes a single web search + summary
│       ├── writer.py          # Synthesizes results into a report
│       └── email.py           # Sends the report by email
├── requirements.txt
├── .env.example
└── LICENSE
```

## Notes

- Searches run concurrently via `asyncio`, so the number of planned searches barely affects total wall-clock time.
- If a given search fails, the pipeline simply drops it and continues with whatever results succeeded.
- Each run logs a trace URL to the console for step-by-step debugging on the OpenAI platform.

## License

This project is licensed under the [MIT License](LICENSE).
