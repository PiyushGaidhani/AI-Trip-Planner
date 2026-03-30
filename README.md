# AI Trip Planner

An agentic travel planning application built with LangGraph, FastAPI, and Streamlit. The project accepts a natural-language trip request such as "Plan a 5-day trip to Goa" and generates a structured itinerary with weather details, places to visit, restaurant suggestions, transport guidance, and rough expense estimates.

## Overview

This project combines:

- `FastAPI` as the backend API layer
- `Streamlit` as the frontend UI
- `LangGraph` to orchestrate an agent-and-tools workflow
- `LangChain` tool integrations for search, weather, and currency conversion
- `Groq` or `OpenAI` as the LLM provider

The app is designed as a travel-planning agent that can:

- create day-wise itineraries
- suggest attractions, restaurants, activities, and transportation options
- estimate hotel and trip expenses
- convert currencies
- fetch weather information

## How It Works

The backend creates a LangGraph workflow in which:

1. a system prompt instructs the model to act as a travel planner
2. the model decides whether it needs tools
3. tool calls are executed through LangGraph's `ToolNode`
4. the model integrates the tool outputs into a final travel plan

The current graph is built in [agent/agentic_workflow.py](/Users/piyushgaidhani/Desktop/AI-Trip-Planner/agent/agentic_workflow.py), and the API endpoint is exposed in [main.py](/Users/piyushgaidhani/Desktop/AI-Trip-Planner/main.py).

## Features

- Natural-language trip planning
- Agentic workflow using LangGraph
- Weather lookup support
- Place search with fallback behavior
- Expense calculation helpers
- Currency conversion
- Streamlit-based interactive UI
- FastAPI backend for programmatic access

## Project Structure

```text
AI-Trip-Planner/
├── agent/
│   └── agentic_workflow.py
├── config/
│   └── config.yaml
├── prompt_library/
│   └── prompt.py
├── tools/
│   ├── currency_conversion_tool.py
│   ├── expense_calculator_tool.py
│   ├── place_search_tool.py
│   └── weather_info_tool.py
├── utils/
│   ├── config_loader.py
│   ├── currency_converter.py
│   ├── expense_calculator.py
│   ├── model_loader.py
│   ├── place_info_search.py
│   └── weather_info.py
├── main.py
├── streamlit_app.py
├── requirements.txt
├── setup.py
└── README.md
```

## Tech Stack

- Python 3.10+
- FastAPI
- Streamlit
- LangChain
- LangGraph
- Groq
- OpenAI
- Tavily
- OpenWeatherMap
- ExchangeRate API

## Installation

This project uses `uv` for dependency management and environment synchronization. The repository includes [uv.lock](/Users/piyushgaidhani/Desktop/AI-Trip-Planner/uv.lock), so the recommended setup flow is `uv` first.

### 1. Clone the repository

```bash
git clone https://github.com/PiyushGaidhani/AI-Trip-Planner.git
cd AI-Trip-Planner
```

### 2. Install dependencies with `uv`

Install and sync the environment with:

```bash
uv sync
```

You can then run project commands with `uv run`.

### Alternative: Create and activate a virtual environment

If you are not using `uv`, you can still use a regular Python virtual environment:

```bash
python -m venv env
source env/bin/activate
```

On Windows:

```bash
env\Scripts\activate
```

### 3. Install dependencies without `uv`

Using `pip`:

```bash
pip install -r requirements.txt
```

Optional editable install:

```bash
pip install -e .
```

## Environment Variables

Create a `.env` file in the project root.

Example:

```env
GROQ_API_KEY="your_groq_api_key"
OPENAI_API_KEY="your_openai_api_key"
TAVILY_API_KEY="your_tavily_api_key"
EXCHANGE_RATE_API_KEY="your_exchange_rate_api_key"
OPENWEATHERMAP_API_KEY="your_openweathermap_api_key"
GEOFY_API_KEY="your_place_search_key"
LANGCHAIN_API_KEY="your_langchain_api_key"
```

### Which keys are actually used

- `GROQ_API_KEY`: used when the backend loads the Groq model
- `OPENAI_API_KEY`: used if you switch the model provider to OpenAI
- `TAVILY_API_KEY`: used by the Tavily place-search fallback
- `EXCHANGE_RATE_API_KEY`: used for currency conversion
- `OPENWEATHERMAP_API_KEY`: used by the weather tool
- `GEOFY_API_KEY`: read by the place-search tool
- `LANGCHAIN_API_KEY`: optional, if you are tracing with LangChain tooling

### Important note about place search

The current implementation in [utils/place_info_search.py](/Users/piyushgaidhani/Desktop/AI-Trip-Planner/utils/place_info_search.py) still uses LangChain's `GooglePlacesAPIWrapper` internally. That means:

- `GEOFY_API_KEY` is read by the app
- but the underlying wrapper is still Google Places based
- if that key is incompatible with the wrapper, the app falls back to Tavily search instead of failing hard

So at the moment, `GEOFY_API_KEY` should be treated as the app-level place-search key setting, but the place-search implementation is not yet a native Geofy integration.

## Configuration

Model configuration lives in [config/config.yaml](/Users/piyushgaidhani/Desktop/AI-Trip-Planner/config/config.yaml).

Current structure:

```yaml
llm:
  openai:
    provider: "openai"
    model_name: "o4-mini"
  groq:
    provider: "groq"
    model_name: "llama-3.3-70b-versatile"
```

The backend currently instantiates the graph with:

```python
GraphBuilder(model_provider="groq")
```

So Groq is the default active provider unless you change the backend code.

## Running the Application

This project has two services:

- a FastAPI backend
- a Streamlit frontend

You need to run both.

### 1. Start the FastAPI backend

Recommended:

```bash
uv run uvicorn main:app --reload --port 8000
```

Without `uv`:

```bash
uvicorn main:app --reload --port 8000
```

Backend URL:

```text
http://localhost:8000
```

API docs:

```text
http://localhost:8000/docs
```

### 2. Start the Streamlit frontend

In a second terminal:

Recommended:

```bash
uv run streamlit run streamlit_app.py
```

Without `uv`:

```bash
streamlit run streamlit_app.py
```

The Streamlit app sends requests to:

```text
http://localhost:8000/query
```

## API Usage

### POST `/query`

Request body:

```json
{
  "question": "Plan a trip to Goa for 5 days"
}
```

Success response:

```json
{
  "answer": "Generated travel plan..."
}
```

Error response:

```json
{
  "error": "Error details"
}
```

### Example using `curl`

```bash
curl -X POST "http://localhost:8000/query" \
  -H "Content-Type: application/json" \
  -d '{"question":"Plan a budget trip to Goa for 5 days"}'
```

## Streamlit Usage

Once the frontend is running:

1. open the Streamlit app in the browser
2. enter a trip request in the input field
3. click `Send`
4. wait for the backend to generate the trip plan

Example prompts:

- `Plan a trip to Goa for 5 days`
- `Plan a budget-friendly trip to Bali for 7 days`
- `Create a family itinerary for Dubai for 4 days`
- `Suggest an offbeat trip to Himachal Pradesh for 6 days`

## Agent Workflow

The workflow is assembled in [agent/agentic_workflow.py](/Users/piyushgaidhani/Desktop/AI-Trip-Planner/agent/agentic_workflow.py).

Core components:

- `GraphBuilder`: creates the LangGraph workflow
- `SYSTEM_PROMPT`: defines planner behavior and expected output style
- `ToolNode`: runs external tools selected by the model
- `MessagesState`: stores conversation/tool state in the graph

The compiled graph follows this pattern:

```text
START -> agent -> tools -> agent -> END
```

The backend also writes a rendered graph image to `my_graph.png`.

## Available Tools

### Weather Tool

Defined in [tools/weather_info_tool.py](/Users/piyushgaidhani/Desktop/AI-Trip-Planner/tools/weather_info_tool.py).

Supports:

- current weather lookup
- weather forecast lookup

Backed by OpenWeatherMap through [utils/weather_info.py](/Users/piyushgaidhani/Desktop/AI-Trip-Planner/utils/weather_info.py).

### Place Search Tool

Defined in [tools/place_search_tool.py](/Users/piyushgaidhani/Desktop/AI-Trip-Planner/tools/place_search_tool.py).

Supports:

- attraction search
- restaurant search
- activity search
- transportation search

Current behavior:

- tries to initialize the Google Places wrapper with the configured place key
- if initialization fails or the key is invalid, it falls back to Tavily-backed search

### Expense Calculator Tool

Defined in [tools/expense_calculator_tool.py](/Users/piyushgaidhani/Desktop/AI-Trip-Planner/tools/expense_calculator_tool.py).

Supports:

- total hotel cost estimation
- total trip expense calculation
- daily budget calculation

### Currency Converter Tool

Defined in [tools/currency_conversion_tool.py](/Users/piyushgaidhani/Desktop/AI-Trip-Planner/tools/currency_conversion_tool.py).

Supports:

- currency conversion using ExchangeRate API

## Prompt Design

The system prompt in [prompt_library/prompt.py](/Users/piyushgaidhani/Desktop/AI-Trip-Planner/prompt_library/prompt.py) instructs the agent to generate:

- a complete itinerary
- hotel suggestions
- attractions
- restaurant suggestions
- activities
- transportation information
- cost breakdowns
- per-day expense estimates
- weather details

The agent is encouraged to provide both:

- a standard tourist-oriented plan
- an offbeat alternative

## Development Notes

### Packaging

The project includes [setup.py](/Users/piyushgaidhani/Desktop/AI-Trip-Planner/setup.py) and [requirements.txt](/Users/piyushgaidhani/Desktop/AI-Trip-Planner/requirements.txt).

If you are not using `uv`, install dependencies with:

```bash
pip install -r requirements.txt
```

This is more complete than relying only on the lightweight dependencies currently listed in `pyproject.toml`.

For this repository, the preferred approach is:

```bash
uv sync
```

That keeps the environment aligned with [uv.lock](/Users/piyushgaidhani/Desktop/AI-Trip-Planner/uv.lock).

### Generated Files

The backend writes `my_graph.png` when the graph is rendered. If you run the backend repeatedly, this file may be updated.

## Troubleshooting

### `Connection refused` on `localhost:8000`

Cause:

- the FastAPI backend is not running

Fix:

```bash
uvicorn main:app --reload --port 8000
```

### `TypeError: exceptions must derive from BaseException`

Cause:

- raising a string in Python instead of an exception object

This has been addressed in the current Streamlit app by displaying the error with `st.error(...)`.

### Invalid place-search key errors

Cause:

- the configured place-search key is not compatible with `GooglePlacesAPIWrapper`

Current behavior:

- the app falls back to Tavily search instead of crashing

### Weather tool returns empty data

Check:

- `OPENWEATHERMAP_API_KEY` is set correctly
- the requested place name is valid
- the external weather API is reachable

### Currency conversion fails

Check:

- `EXCHANGE_RATE_API_KEY` is valid
- the source and target currency codes are valid ISO currency codes

## Known Limitations

- The place-search implementation is not yet a native Geofy integration.
- The backend currently creates a fresh graph for each request.
- The Streamlit app does not yet maintain a rich conversational history in the rendered UI.
- Some generated travel details depend on external APIs and may be incomplete or stale.
- The app does not currently persist user trips, sessions, or generated plans.

## Suggested Improvements

- replace the current place-search layer with a true Geofy provider integration
- add a `.env.example` file
- add automated tests for tools and API routes
- containerize the frontend and backend with Docker
- add caching for repeated API calls
- improve session memory and multi-turn chat support
- add logging and observability around tool failures

## License

Add a license file if you plan to open-source or distribute the project broadly.
