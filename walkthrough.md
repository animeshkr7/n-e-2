# Name Engine V2 Walkthrough

This walkthrough outlines the newly implemented **Name Engine V2**, which runs completely in parallel to the original name engine without disrupting the primary data extraction workflow. 

## Features Implemented

1. **LLM Edge Case Detector** (`utils/eu_utils/eu_prompts_v2.py`):
   - A newly created Pydantic-backed LLM prompt isolates names with extreme formatting (e.g., dual languages, mononyms, bad punctuation) and normalizes them.

2. **Permutations Generator** (`utils/eu_utils/eu_name_engine_v2_utils.py`):
   - Once a name is cleaned, it is split into parts and the system algorithmically computes all mathematical permutations and combinations.

3. **Similarity Search & Ranker** (`utils/eu_utils/eu_name_engine_v2_utils.py`):
   - Generates a fuzzy-similarity score (`difflib.SequenceMatcher`) against a reference document.
   - Automatically ranks and returns the top 10 best matches.

4. **Parallel Threading Execution**:
   - Integrated into `utils/eu_pdf_utils/eu_pdf_data_points_extractor_utils.py` and `utils/eu_html_utils/eu_data_point_extractor_utils.py`.
   - The extractors now hook into Name Engine V2 using Python's `concurrent.futures.ThreadPoolExecutor`. 
   - This fires-and-forgets the V2 pipeline, meaning it runs asynchronously in the background. If V2 throws an error or takes a long time, the main extraction workflow will continue uninterrupted.

## Code Changes

### Added Files
- [NEW] [eu_prompts_v2.py](file:///c:/Users/aks/Documents/Github/facctum-press-release/utils/eu_utils/eu_prompts_v2.py)
- [NEW] [eu_name_engine_v2_utils.py](file:///c:/Users/aks/Documents/Github/facctum-press-release/utils/eu_utils/eu_name_engine_v2_utils.py)

### Modified Files
- [MODIFY] [eu_pdf_data_points_extractor_utils.py](file:///c:/Users/aks/Documents/Github/facctum-press-release/utils/eu_pdf_utils/eu_pdf_data_points_extractor_utils.py)
- [MODIFY] [eu_data_point_extractor_utils.py](file:///c:/Users/aks/Documents/Github/facctum-press-release/utils/eu_html_utils/eu_data_point_extractor_utils.py)

> [!TIP]
> **Next Steps:**
> Since the reference document was not strictly defined, it currently expects an empty list `[]`. When you have a live list of historical names or the full consolidated EU list ready, simply pass it into `run_name_engine_v2_pipeline(raw_names, reference_doc)` inside the extractors!
