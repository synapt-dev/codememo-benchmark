# CodeMemo Benchmark

Reproducible benchmark for evaluating coding memory systems. Pins all
dependencies to exact versions so results can be independently verified.

## Published Results (2026-03-14)

| System | Factual | Debug | Architecture | Temporal | Convention | Cross-Session | **Overall** |
|--------|---------|-------|-------------|----------|------------|---------------|-------------|
| **synapt v0.6.2** (local 3B) | **97.14** | **100.0** | 92.86 | **90.91** | **80.0** | **86.36** | **90.51** |
| Mem0 v1.0.5 (OpenAI cloud) | 72.73 | 77.78 | **100.0** | 87.50 | 42.86 | 71.43 | **76.0** |

158 questions across 3 coding projects, 6 categories. Same gpt-4o-mini judge for both.

## Quick Start

```bash
# Clone this benchmark workspace
gr init https://github.com/laynepenney/codememo-benchmark.git
cd codememo-benchmark

# Install dependencies
gr run setup

# Set your OpenAI API key (required for answer generation + judging)
export OPENAI_API_KEY=sk-...

# Run both evals
gr run benchmark

# Or run individually
gr run benchmark-synapt   # synapt only
gr run benchmark-mem0     # mem0 only
```

## What's Pinned

| Component | Version | Commit |
|-----------|---------|--------|
| synapt | v0.6.2 | `cc32627` |
| eval harness | kept privately | — |
| mem0 | v1.0.5 | tag `v1.0.5` |
| mem0ai (pip) | 1.0.5 | — |
| Judge model | gpt-4o-mini | — |
| Answer model | gpt-4o-mini | — |

## What Each System Uses

| | synapt | Mem0 |
|---|--------|------|
| Memory indexing | Local (FTS5 + all-MiniLM-L6-v2) | OpenAI (gpt-4o-mini extraction + text-embedding-3-small) |
| Retrieval | Local (66ms) | Qdrant local + OpenAI embed (212ms) |
| Cloud dependency | **None** for memory | OpenAI API for all memory operations |
| Hardware | Runs on M2 Air | Runs anywhere with OpenAI key |

## Adding a New System

Implement the `SystemUnderTest` protocol in your evaluation harness:

```python
class SystemUnderTest(Protocol):
    def ingest(self, session_paths: list[Path]) -> None: ...
    def query(self, question: str, max_chunks: int = 20) -> str: ...
    def stats(self) -> dict: ...
    def close(self) -> None: ...
```

See `competitor_eval.py` for the Mem0 and Memobase adapter implementations.
