# Comparing small LLMs on tool selection (MetaTool benchmark)

An agent that uses tools has to make two decisions before it does anything useful: does this request need a tool at all, and if so, which one? This project tests how well three small open-source models make those two decisions, using the [MetaTool benchmark](https://arxiv.org/abs/2310.03128).

I did this as coursework for IS 698 (Software Engineering with Generative AI) at UMBC.

## Setup

- **Models:** `gemma2:2b`, `llama3.2:3b` and `qwen2.5:3b`, all run locally through [Ollama](https://ollama.com)
- **Settings:** temperature 0 and seed 42 for every model, so runs are repeatable
- **Tools:** the 47 tools in the benchmark's tool description catalog
- **Queries:** 141 in total. I drew three queries at random for each of the 47 tools from the 10,615 benchmark queries that map to those tools.

Every model got the same single prompt. It lists all 47 tool names with their descriptions and asks for a JSON object with two keys and nothing else:

```json
{"tool_needed": "yes", "tool_name": "MapTool"}
```

`tool_name` has to match one of the catalog names exactly, or be `null` when no tool is needed.

## Metrics

Accuracy, precision, recall and F1 measure the first decision (is a tool needed?). Correct Selection Rate (CSR) measures the whole task: the share of queries where the model said yes *and* named the exact right tool.

One thing to know when reading the numbers: every sampled query really does need a tool. So a model can never produce a false positive here, precision is 1.0 by construction, and accuracy equals recall. CSR is the number that tells the models apart.

## Results

| Model | Accuracy | Precision | Recall | F1 | CSR |
|---|---:|---:|---:|---:|---:|
| gemma2:2b | 0.9929 | 1.0000 | 0.9929 | 0.9964 | 0.6738 |
| llama3.2:3b | 0.9362 | 1.0000 | 0.9362 | 0.9670 | **0.6809** |
| qwen2.5:3b | 0.6667 | 1.0000 | 0.6667 | 0.8000 | 0.4965 |

## What I found

- **llama3.2:3b picked the right tool most often**, on 96 of 141 queries. gemma2:2b was one query behind at 95.
- **gemma2:2b almost never refused.** It said no tool was needed on only 1 query, but still named the wrong tool about a third of the time. Knowing a tool is needed and knowing which one are separate skills.
- **qwen2.5:3b refused far too often.** It said no tool was needed on 47 of 141 queries and scored zero on 11 of the 47 tools. A refused query can never get the right tool, which is most of the reason its CSR is so much lower.
- **Similar descriptions cause most of the wrong picks.** Pairs like ResearchHelper and ResearchFinder, TripTool and TripAdviceTool, and HouseRentingTool and HousePurchasingTool were mixed up by every model. Their descriptions differ by only a few words.

## Limits

- Three queries per tool is a small sample, so the per-tool scores are rough.
- The sample has no queries where the right answer is "no tool", so this setup can't show whether a model over-uses tools.
- One prompt, no few-shot examples. Examples that separate the look-alike tools would probably raise CSR.

## Running it

1. Install [Ollama](https://ollama.com) and pull the three models:
   ```bash
   ollama pull gemma2:2b
   ollama pull llama3.2:3b
   ollama pull qwen2.5:3b
   ```
2. Install the Python packages:
   ```bash
   pip install ollama pandas scikit-learn
   ```
3. Download `all_clean_data.csv` and `big_tool_des.json` from the [MetaTool repository](https://github.com/HowieHwong/MetaTool) and place them next to the notebook.
4. Run the notebook from top to bottom. It samples the queries, runs each model, and saves the predictions and metrics as CSV files.

## Reference

Huang, Y., et al. "MetaTool Benchmark for Large Language Models: Deciding Whether to Use Tools and Which to Use." arXiv:2310.03128.

## Author

Divyasree Guttha, M.S. Information Systems, UMBC · [LinkedIn](https://www.linkedin.com/in/divyasree-guttha)
