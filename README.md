# Crypto Analyzer

A conversational crypto market analyst built with [LangGraph](https://github.com/langchain-ai/langgraph) and OpenAI. Ask questions in plain English ("Give me current data for BTC and explain the key metrics"), and the agent decides which tools to call, fetches live market data and news, and answers in a structured format.

## How it works

The agent is a LangGraph `StateGraph` over `MessagesState` with two nodes:

```
START ──▶ assistant ──(tools_condition)──▶ tools
              ▲                              │
              └──────────────────────────────┘
              │
              └──(no tool calls)──▶ END
```

- **assistant**: `gpt-4o` bound to the tools below, guided by a system prompt that tells it to rely on tool data only (no invented prices).
- **tools**: a `ToolNode` that runs whatever tool calls the assistant made, then hands the results back to the assistant.
- **tools_condition**: routes to `tools` when the last AI message contains tool calls, otherwise ends the run.

An optional `MemorySaver` checkpointer keeps conversation history per `thread_id`, so follow-up questions can refer to earlier answers.

### Tools

| Tool | Source | What it returns |
|---|---|---|
| `crypto_list_tool` | [FreeCryptoAPI](https://freecryptoapi.com) | List of supported cryptocurrencies |
| `crypto_data_tool(symbol)` | FreeCryptoAPI | Live data for a symbol: `last`, `lowest`, `highest`, `daily_change_percentage`, `date`, `source_exchange` |
| `news_tool(query, max_items=5)` | [NewsAPI](https://newsapi.org) | Latest English headlines (title, source, URL, published time) |

### Response format

Each answer follows the same structure: **Summary**, **Data**, **What it means**, **Next Steps**.

## Setup

Requires Python 3.12+ and [uv](https://docs.astral.sh/uv/).

```bash
git clone https://github.com/praveencali2017/crypto_analyzer.git
cd crypto_analyzer
uv sync
```

Create a `.env` file in the project root (it is git-ignored):

```env
OPENAI_API_KEY=your-openai-key
FREECRYPTO_TOKEN=your-freecryptoapi-token
NEWSAPI_KEY=your-newsapi-key
```

The notebook checks for all three keys on startup and stops with an error if any are missing.

## Usage

Open [src/crypto_analyzer/analyzer.ipynb](src/crypto_analyzer/analyzer.ipynb) with the project's `.venv` as the kernel and run the cells in order.

- `execute(use_memory=False)`: asks one question and runs it through the graph without history.
- `execute(use_memory=True)`: same, but uses the checkpointed graph so the conversation carries over between calls.
- The last cell starts an interactive loop. Answer `yes` to ask a question, or `no` to exit.

Each run prints the full message trace (human message, tool calls, tool results, final answer), which makes it easy to see which tools the agent used.

### Example queries

- `Give me current data for BTC and explain the key metrics in bullet points.`
- `Compare BTC and ETH using current data and the latest headlines.`
- `What are the latest news stories about Solana?`

## Project structure

```
crypto_analyzer/
├── src/crypto_analyzer/
│   ├── analyzer.ipynb   # tools, system prompt, graph, and interactive runner
│   └── __init__.py
├── pyproject.toml
└── uv.lock
```

## Notes

- Market data and news come from free-tier APIs, which are rate-limited and may lag.
- This project is for learning and exploration only, not financial advice.
