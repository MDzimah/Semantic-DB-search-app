# Semantic DB Search

Initial vibe-coded skeleton for a local semantic database search app. This repository is pending formal review and is not the final version.

The scope is generic semantic DB search over tabular datasets. Fund/instrument matching is the initial use case, not the only intended use case.

The project uses a hybrid ranking strategy:
- Semantic similarity from embeddings
- Structured deciding-column scoring (broker, issuer, ticker, ISIN, etc.)
- Lexical fallback score

This reduces pure keyword overlap errors and keeps ranking grounded in semantic + structured signals.

## Status

This is an early implementation snapshot intended for iteration and review.

Implemented:
- Excel import and row normalization
- Deciding-column selection in UI
- Conservative text cleaning (removes obvious deciding-value repetition from descriptive text)
- Embedding-based retrieval with SentenceTransformers
- Vector index with Faiss (with fallback path if Faiss is unavailable)
- Reranking with deciding-column weighting
- Query history persistence and basic settings persistence
- Streamlit UI with:
  - Import button
  - Reset button
  - Results-per-query control
  - Sort mode (recency/alphabetical)
  - Collapsible query result blocks
  - Multi-line query input
- Launcher entrypoint for future executable packaging

Planned / pending review:
- Production-grade .exe packaging workflow (spec file and release pipeline)
- Full persisted index reload/reuse flow for very large datasets
- Advanced alias/family detection layer
- Export-to-Excel results

## Tech Stack

- Python 3.10+
- Streamlit
- openpyxl
- sentence-transformers
- faiss-cpu
- rapidfuzz
- numpy
- pandas
- pyinstaller

## Project Structure

- src/app.py: Streamlit app and UI flow
- src/data_loader.py: Excel ingestion, normalization, deciding-column cleaning
- src/search_engine.py: Embeddings, Faiss indexing/search, reranking
- src/state_store.py: Local persistence for dataset snapshot, settings, history
- src/launcher.py: Launcher entrypoint for app/exe mode
- src/requirements.txt: Python dependencies
- src/saved_state/: persisted local state files
- src/data/: data files (if stored locally)
- src/cache/: cache files

## Ranking Logic (Current)

Final score is a weighted combination of:
- Semantic score (embedding similarity)
- Decision score (fuzzy/exact agreement against selected deciding columns)
- Lexical score (display-text fuzzy overlap)

Current default weights in code:
- Semantic: 0.60
- Decision: 0.30
- Lexical: 0.10

Column-based weighting is applied so critical fields (ticker/ISIN/broker-like columns) contribute more in decision scoring.

## Persistence

Current local state includes:
- Dataset snapshot and chosen deciding columns
- Query history with timestamps
- Basic UI settings

State is stored in src/saved_state.

## Notes

- First model load can be slow because SentenceTransformers may download model assets.
- You may see non-fatal transformer watcher warnings in Streamlit logs (for optional vision modules such as torchvision). They do not block core text search behavior.
- For large datasets (for example ~45K rows), expect an initial indexing step before fast query-time retrieval.

## Scope Clarification

- Primary direction: generic semantic DB search across structured datasets.
- Initial motivating scenario: fund/instrument lookup and matching.
- Current repository state: non-final scaffold pending review and refinement.

## Next Recommended Steps

1) Add robust index persistence/reload tied to dataset fingerprint.
2) Improve deciding-value decontamination and alias handling for shortened broker names.
3) Add packaged Windows release flow with PyInstaller spec and release instructions.
4) Add evaluation script with known query-result pairs to tune reranking weights.
