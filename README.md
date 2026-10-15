# Multi-Agent Research and Mapping with smolagents

A Jupyter notebook exploring web research, agent planning, delegation, and multimodal validation using Hugging Face `smolagents`.

The example researches Batman filming locations and supercar factories, estimates cargo flight times from a fictional Gotham origin at New York City coordinates, and asks a manager agent to produce a world map.

## What the notebook demonstrates

- A custom `@tool` for estimating cargo flight time between two coordinates.
- A standalone `CodeAgent` for an initial research task and a more detailed report.
- A research agent (`web_agent`) with Google search and webpage browsing tools.
- A manager agent that delegates research, calculates travel times, and creates a Plotly map.
- A multimodal final-answer check that reviews the agent's reasoning and the saved map.

## Agent workflow

| Component | Role |
| --- | --- |
| `GoogleSearchTool` | Searches for filming locations and factory information. |
| `VisitWebpageTool` | Reads webpages to support the research. |
| `calculate_cargo_travel_time` | Estimates travel time from latitude/longitude pairs. |
| `web_agent` | Researches locations on behalf of the manager. |
| `manager_agent` | Coordinates research and generates the final visualization. |
| `check_reasoning_and_plot` | Sends the saved image and agent steps to an OpenAI vision model for a PASS/FAIL review. |

The final prompt requests at least six locations, a map made with `plotly.express.scatter_map`, and point colors based on estimated travel time.

## Getting started

### 1. Clone and create an environment

The uploaded notebook records Python **3.11.9** in its metadata.

Run these commands in PowerShell on Windows:

```powershell
git clone https://github.com/gurramjayakrishna1/multi_agents_smolagents.git
cd multi_agents_smolagents
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
```

### 2. Install dependencies

```powershell
python -m pip install "smolagents[toolkit,litellm]" openai truststore pillow pandas numpy geopandas shapely plotly kaleido langfuse openinference-instrumentation-smolagents jupyterlab ipykernel
```

This is a starting dependency list inferred from the notebook, rather than a pinned, tested environment. Package APIs and model availability may require adjustments.

Plotly's Kaleido image export requires a compatible Chrome/Chromium installation. See the [Plotly image export documentation](https://plotly.com/python/static-image-export/).

### 3. Configure credentials

Set environment variables in the same PowerShell session before launching Jupyter:

```powershell
$env:ANTHROPIC_API_KEY = "<your-anthropic-api-key>"
$env:OPENAI_API_KEY = "<your-openai-api-key>"
$env:SERPAPI_API_KEY = "<your-serpapi-api-key>"

# For the notebook's Langfuse tracing setup:
$env:LANGFUSE_PUBLIC_KEY = "<your-langfuse-public-key>"
$env:LANGFUSE_SECRET_KEY = "<your-langfuse-secret-key>"
$env:LANGFUSE_BASE_URL = "<your-langfuse-project-base-url>"
```

The notebook uses environment lookups; it does **not** load a `.env` file automatically. Keep real credentials out of Git.

`GoogleSearchTool()` defaults to SerpApi. To use Serper instead, set `SERPER_API_KEY` and explicitly select `GoogleSearchTool(provider="serper")` in each relevant agent.

The notebook configures its main model through LiteLLM and uses `gpt-4o` for image validation. Update model identifiers as needed for your provider account. Research, search, and validation calls can incur API charges.

### 4. Open and run

```powershell
jupyter lab
```

Open [multi_agents_systems.ipynb](multi_agents_systems.ipynb), select the virtual environment's Python kernel, and run the cells in order. In VS Code, open the notebook and select the same environment as its kernel.

## Before running the current notebook

In the manager agent's `additional_authorized_imports` list, add a comma after `"plotly.io"`:

```python
"plotly.io",
"shapely",
```

Without the comma, Python joins the adjacent strings into `"plotly.ioshapely"`, so those two imports are not authorized separately.

The final prompt requires `saved_map.png`, while its embedded example uses `saved_image.png`. Make the example use `fig.write_image("saved_map.png")` to match the validation function.

The `manager_agent.visualize()` cell may require Graphviz and its Python package, depending on your environment.

## Travel-time calculation

The custom tool uses the haversine formula with an Earth radius of 6,371 km. It adds a 10% distance allowance, assumes a default speed of 750 km/h, and adds one hour of overhead:

```text
estimated hours = (great-circle distance × 1.1 / cruising speed) + 1.0
```

The result is rounded to two decimal places. This is an illustrative estimate; it does not model airport routing, cargo handling, customs, weather, or actual flight schedules.

## Expected results and limitations

Earlier cells request a pandas DataFrame and a detailed research report. The final multi-agent task requests a Plotly world map saved as `saved_map.png` in the notebook's working directory.

The uploaded notebook contains no saved execution outputs or map image. Results depend on live sources and model behavior; the request for “all” filming locations does not establish exhaustive coverage. The model-based PASS/FAIL check is a heuristic review, not independent proof of factual accuracy.

## References

- [Hugging Face Agents Course: Multi-Agent Systems](https://huggingface.co/learn/agents-course/unit2/smolagents/multi_agent_systems)
- [smolagents built-in tools](https://huggingface.co/docs/smolagents/reference/default_tools)
- [Plotly static image export](https://plotly.com/python/static-image-export/)
