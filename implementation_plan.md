# Name Engine 2 Implementation Plan

The objective is to implement "Name Engine 2" as a separate, parallel workflow. This engine will focus on advanced name resolution by identifying edge cases via LLM, generating permutations/combinations, and performing similarity searches against a reference document to rank the top 10 matches.

## User Review Required

> [!IMPORTANT]
> Since this introduces a parallel workflow, we need to decide on the execution model. Python's Global Interpreter Lock (GIL) might affect multithreading performance if the permutation/search logic is CPU-bound. We can use `concurrent.futures.ThreadPoolExecutor` for I/O bound LLM calls and `ProcessPoolExecutor` for CPU bound tasks, or implement this as an entirely separate serverless function (like an AWS Lambda).

## Open Questions

> [!WARNING]
> Please clarify the following points before execution begins:
> 1. **Reference Document:** What "doc" are we searching against for the similarity search? Is it the `eu_consolidated.xml` file, a database of historical names, or something else?
> 2. **Similarity Mechanism:** Should the similarity search use semantic vector embeddings (e.g., OpenAI Embeddings + FAISS) or lexical string matching (e.g., `rapidfuzz` / Levenshtein distance)? Vector search is better for semantic meaning, while fuzzy matching is better for slight spelling variations.
> 3. **Parallel Workflow:** Should the parallel execution happen inside the existing extractor Python process (via threading/multiprocessing), or should it be triggered asynchronously via a message queue/separate service?

## Proposed Changes

### 1. New Module: `utils/eu_utils/eu_name_engine_v2_utils.py`
This will be the central hub for Name Engine 2, completely separated from the existing name aliases logic.

#### [NEW] `utils/eu_utils/eu_name_engine_v2_utils.py`
- `run_name_engine_v2_pipeline(raw_names, reference_doc)`: Main entry point orchestrating the steps below.
- `detect_and_clean_edge_cases(raw_names)`: Makes an LLM call to identify if the name contains edge cases (e.g., dual languages, mononyms, extreme formatting anomalies) and normalizes them.
- `generate_permutations(cleaned_names)`: Generates complex permutations and combinations of the name parts.
- `search_and_rank_names(permutations, reference_doc)`: Performs the similarity search and returns the top 10 matches.

### 2. New Prompts: `utils/eu_utils/eu_prompts_v2.py`
#### [NEW] `utils/eu_utils/eu_prompts_v2.py`
- Create `edge_case_name_prompt` to instruct the LLM specifically on how to detect and handle edge case anomalies in PR name formats.

### 3. Extractor Integration (Parallel Trigger)
We will modify the extractors where the original name engine is currently used, so that Name Engine 2 is triggered concurrently.

#### [MODIFY] `utils/eu_pdf_utils/eu_pdf_data_points_extractor_utils.py`
- Import `run_name_engine_v2_pipeline`.
- Use `concurrent.futures.ThreadPoolExecutor` to trigger `run_name_engine_v2_pipeline` in the background alongside the existing `permute_names` and `adjust_for_names` logic.

#### [MODIFY] `utils/eu_html_utils/eu_data_point_extractor_utils.py`
- Apply the same parallel execution pattern here for HTML extractions.

### 4. Dependency Updates
#### [MODIFY] `requirements.txt`
- Depending on the answer to Open Question #2, we may need to add `rapidfuzz` (for fuzzy string matching) or `faiss-cpu` (for embedding-based vector similarity search).

## Verification Plan

### Automated Tests
- Create unit tests for `detect_and_clean_edge_cases` with known edge-case names.
- Verify `generate_permutations` successfully outputs the correct mathematical number of combinations based on name parts.
- Test `search_and_rank_names` against a dummy doc with known typo variations to ensure the ranking algorithm correctly places the best match in the Top 10.

### Manual Verification
- Run an entire EU PDF extraction flow locally.
- Inspect the application logs to ensure `Name Engine 2` executed in parallel without blocking the primary extraction flow.
- Review the Top 10 ranked outputs to confirm the similarity quality.
