# Vectorless RAG with PageIndex and Groq

This project demonstrates a vectorless retrieval-augmented generation (RAG) workflow over a PDF document using:

- PageIndex for tree-based document indexing and retrieval
- LangChain + ChatGroq for reasoning and answer generation
- A Jupyter notebook implementation in [vectorless_rag.ipynb](vectorless_rag.ipynb)

Unlike embedding-based RAG, this approach retrieves relevant sections by asking an LLM to reason over the document tree and select the best node IDs.

## Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [Requirements](#requirements)
- [Setup](#setup)
- [How to Run](#how-to-run)
- [Core Functions](#core-functions)
- [Troubleshooting](#troubleshooting)
- [Security Notes](#security-notes)

## Overview

The notebook builds an end-to-end vectorless pipeline:

1. Upload a PDF to PageIndex and wait for processing.
2. Load and inspect the generated hierarchical tree.
3. Ask an LLM to choose relevant tree nodes for a user query.
4. Retrieve node content and generate a grounded answer with citations.

## Architecture

1. **Indexing**: `PageIndexClient.submit_document(...)` uploads the PDF and starts async processing.
2. **Tree Retrieval**: `PageIndexClient.get_tree(...)` returns structured sections and nested nodes.
3. **LLM Tree Search**: `llm_tree_search(...)` sends a compressed tree to the LLM and gets `node_list`.
4. **Node Resolution**: `find_nodes_by_ids(...)` maps selected IDs back to full node content.
5. **Answer Generation**: `generate_answer(...)` produces JSON output (`answer`, `citations`).
6. **Pipeline Orchestration**: `vectorless_rag(...)` runs all steps end-to-end.

## Project Structure

- [vectorless_rag.ipynb](vectorless_rag.ipynb): Main notebook with full workflow.
- [rpa textbook-19-35.pdf](rpa textbook-19-35.pdf): Sample document used in the notebook.
- [README.md](README.md): Project documentation.
- `.env`: Local environment variables (not for committing).

## Requirements

- Python 3.10+
- A Groq API key
- A PageIndex API key
- Jupyter Notebook or VS Code notebook support

## Setup

Install dependencies:

```bash
pip install -U pageindex langchain-groq python-dotenv
```

Create a `.env` file in the project root:

```env
PAGE_Index_KEY=your_pageindex_api_key
GROQ_API_KEY=your_groq_api_key
```

Important: Keep the variable names aligned with the notebook code. If you rename variables, update the notebook accordingly.

## How to Run

Open [vectorless_rag.ipynb](vectorless_rag.ipynb) and run cells top-to-bottom:

1. Install dependencies.
2. Load environment variables and initialize `ChatGroq` and `PageIndexClient`.
3. Submit the PDF and wait until status is `completed`.
4. Fetch and inspect the tree (`get_tree`).
5. Run `llm_tree_search` test.
6. Run the full pipeline:

```python
answer = vectorless_rag(
		query="What are the components of rpa",
		tree=pageindex_tree,
		verbose=True,
)

print(answer.get("answer", ""))
print(answer.get("citations", []))
```

Expected return format:

```json
{
	"answer": "...",
	"citations": [
		{"section": "...", "page": "..."}
	]
}
```

## Core Functions

- `llm_tree_search(query, tree) -> dict`
	- Compresses the tree, prompts the LLM, and returns JSON with `thinking` and `node_list`.

- `find_nodes_by_ids(tree, target_ids) -> list`
	- Recursively traverses the tree to collect matched nodes.

- `generate_answer(query, nodes) -> dict`
	- Builds context from retrieved nodes and asks the LLM for a structured JSON answer.

- `vectorless_rag(query, tree, verbose=True) -> dict`
	- Orchestrates tree search, node retrieval, and answer generation.

## Troubleshooting

- `NameError: GROQ_API_KEY is not defined`
	- Ensure `.env` is loaded and `GROQ_API_KEY` is set.

- `OSError: Invalid argument` when submitting PDF
	- Use a valid absolute path (for example, `D:\\pageindex\\rpa textbook-19-35.pdf`).

- `TypeError: invoke() missing 1 required positional argument: input`
	- Pass the prompt as the first argument to `llm.invoke(prompt, ...)`.

- `AttributeError: 'AIMessage' object has no attribute 'choices'`
	- With LangChain ChatGroq, parse `response.content` instead of `response.choices`.

- `BadRequestError: 'messages' must contain the word 'json' ...`
	- When using `response_format={"type": "json_object"}`, include explicit JSON instructions in the prompt.

## Security Notes

- Never commit `.env` or API keys.
- Rotate exposed keys immediately if they are accidentally leaked.
- Add `.env` to `.gitignore` if not already present.
