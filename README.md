# AI For BI SQL Server using LangGraph

This repository provides an advanced, agent-powered T-SQL assistant for Microsoft SQL Server 2019, leveraging LangChain, LangGraph, Azure OpenAI, and Python data visualization. The assistant can interpret natural language business intelligence (BI) questions, generate optimized SQL queries, execute them, explain results, and produce charts when possible.

## Features

- **Natural Language to SQL**: Converts business questions into correct, efficient T-SQL queries for SQL Server 2019.
- **Schema-aware**: Uses schema metadata to ensure queries only reference valid tables/columns.
- **Error Handling & Retries**: Attempts to correct failed queries using error feedback.
- **Result Explanation**: Provides clear, human-readable answers based on query results.
- **Automated Chart Generation**: Attempts to visualize results using Matplotlib and Pandas, with chart type chosen via LLM.
- **Workflow Orchestration**: Modular agent workflow powered by LangGraph.

## Code Structure Overview

The core logic resides in `app.ipynb`, which orchestrates the following workflow:

1. **Environment Setup**
   - Loads connection strings and Azure OpenAI credentials from environment variables.
   - Establishes SQL Server connection using SQLAlchemy.

2. **Schema Extraction**
   - Retrieves schema metadata from `[ExecutiveDashboard].[dbo].[schema_metadata]`.
   - Formats schema info for LLM prompting.

3. **Agent State Modeling**
   - Defines `AgentState` (using Pydantic) to track question, schema, SQL code, result, answers, chart code, and more.

4. **LangGraph Workflow Nodes**
   - **Relevance Check**: Determines if the question relates to warehouse data.
   - **Schema Fetch**: Loads schema info into state.
   - **SQL Query Generation**: Uses LLM to translate the question + schema into a T-SQL query (with NOLOCK hints).
   - **Query Execution**: Runs SQL and records result/errors.
   - **Validation**: Checks for execution errors.
   - **Retry**: If errors, prompts the LLM for corrections (up to MAX_RETRIES).
   - **Result Explanation**: Uses LLM to summarize the result.
   - **Visualization**: If possible, LLM generates Python code for a chart and the notebook executes it, embedding the image.

5. **Orchestration**
   - State transitions are managed via LangGraph's `StateGraph`, with conditional edges for retries and error handling.

6. **Usage Example**
   - Example queries show how to run the assistant and display answers, SQL, and charts.

## Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/UtkPatAI25/AI_For_BI_SQL_Server_using_Langgraph.git
   cd AI_For_BI_SQL_Server_using_Langgraph
   ```

2. **Install dependencies**
   - Recommended: Use a virtual environment.
   ```bash
   pip install -r requirements.txt
   ```
   - Required packages include:
     - `python-dotenv`
     - `sqlalchemy`
     - `pandas`
     - `matplotlib`
     - `langchain`
     - `langgraph`
     - `langchain-openai`
     - `pydantic`
     - `azure-openai` (if needed, for Azure LLM access)

3. **Set up environment variables**
   - Create a `.env` file with:
     ```
     py-connectionString=your_sqlalchemy_connection_string
     AZURE_OPENAI_DEPLOYMENT_NAME=your_azure_deployment
     AZURE_OPENAI_ENDPOINT=your_azure_endpoint
     AZURE_OPENAI_API_KEY=your_api_key
     AZURE_OPENAI_API_VERSION=2024-12-01-preview
     ```

## Usage

Open `app.ipynb` in Jupyter Notebook or VS Code, and run the cells. You can interact with the assistant using the provided examples:

```python
answer, sql_code, final_state = run_sql_assistant("Top 10 distributor by sales in May 2025")
print("LLM Answer:\n", answer)
print("SQL Query Executed:\n", sql_code)

if final_state.get("viz_image"):
    from IPython.display import Image
    import base64
    display(Image(data=base64.b64decode(final_state["viz_image"])))
elif final_state.get("viz_reason"):
    print("No chart generated:", final_state["viz_reason"])
```

## How It Works

### Flow Diagram

![image.png](attachment:image.png)

### Node Details

- **check_relevance_node**: Filters out unrelated questions.
- **get_schema_node**: Loads schema info for context.
- **build_query_node**: Prompts LLM to generate SQL (schema-constrained, NOLOCK, T-SQL only).
- **execute_query_node**: Runs SQL against database.
- **validate_node**: Looks for errors in execution.
- **retry_query_node**: Attempts to fix failed queries using error feedback.
- **explain_result_node**: LLM generates a human-readable answer from result.
- **analyze_and_generate_visualization_node**: Asks LLM to write Python code for charts, executes code, and embeds image.

### Security & Reliability

- SQL queries are strictly limited to schema metadata.
- NOLOCK hints are enforced to avoid locks.
- LLM-generated Python code for charts is executed in a controlled environment.
- Error handling, retries, and clear explanations are integrated.

## Customization

- **Schema Source**: Adjust the schema extraction query as needed.
- **LLM Prompting**: Modify system prompts for different SQL dialects or business rules.
- **Visualization Code**: Extend chart support by customizing the LLM prompt and chart execution logic.

## Limitations

- Requires up-to-date schema metadata in `[schema_metadata]` table.
- Only supports SQL Server 2019 (syntax and features).
- LLM accuracy depends on prompt quality and model configuration.
- Visualization is limited to what the LLM can infer from sample data and columns.

## License

MIT License (see LICENSE file).

## Contact

For questions, issues, or contributions, open an issue or reach out via GitHub.
