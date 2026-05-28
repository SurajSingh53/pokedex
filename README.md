# Pokedex Intelligence Platform

An end-to-end **Python + Snowflake** project built on a Pokemon dataset (stats, types, evolutions, images).

It demonstrates:
- **ETL** with Snowpark and the Snowflake Python connector
- **Dimensional modeling** (raw -> marts) via Snowpark DataFrames
- **Battle simulation** using cross joins on type effectiveness
- **Image embeddings** (ResNet18) stored as Snowflake `VECTOR(FLOAT, 512)`
- **Vector similarity search** with `VECTOR_COSINE_SIMILARITY`
- **Snowpark ML** classifiers: predict `is_legendary` and `type1`
- **Cortex LLM** natural-language Q&A (text-to-SQL)
- **Streamlit** UI covering Explorer / Compare / Visual Similarity / Team Builder / Insights / Ask AI

## Architecture

```
CSVs + Images  ->  RAW schema (stats, evolution, images + @stage)
                        |
                        v
                 Snowpark transforms
                        |
                        v
   MARTS (DIM_POKEMON, DIM_TYPES, FACT_TYPE_EFFECTIVENESS, FACT_BATTLES, AGG_WIN_RATES)
                        |
        +---------------+---------------+
        v               v               v
   Snowpark ML    ML.POKEMON_EMBEDDINGS   Cortex (mistral-large2)
   (legendary,    (VECTOR 512)            text-to-SQL
    type1)
                        |
                        v
              Streamlit UI (local or SiS)
```

## Setup

1. **Python 3.11** (required for Snowpark).
2. Copy `.env.example` to `.env` and fill in Snowflake creds.
3. Install deps:

   ```powershell
   uv sync          # or: pip install -e .
   ```

## Run the full pipeline

```powershell
python -m scripts.run_pipeline
```

Or run each step individually:

```powershell
python -m scripts.run_ddl
python -m src.etl.load_csv
python -m src.etl.load_images
python -m src.snowpark.transform
python -m src.snowpark.battles
python -m src.ml.embeddings
python -m src.ml.train_legendary
python -m src.ml.train_type
```

## Launch the UI

```powershell
streamlit run src/app/streamlit_app.py
```

## Project layout

```
src/
  config.py              # Snowpark session + paths
  etl/
    load_csv.py
    load_images.py
  snowpark/
    transform.py         # MARTS builders
    battles.py           # battle simulation
  ml/
    embeddings.py        # ResNet -> VECTOR(FLOAT, 512)
    train_legendary.py   # Snowpark ML classifier
    train_type.py
    cortex_qa.py         # NL -> SQL via Cortex
  app/
    streamlit_app.py
sql/
  ddl.sql
scripts/
  run_ddl.py
  run_pipeline.py
Data/
  pokemon (1).csv        # stats (801 rows, 41 cols)
  pokemon.csv            # evolution chain
  Images/                # 800+ PNGs
```

## Notes

- Battle simulation defaults to the top-200 Pokemon by `BASE_TOTAL` (~40k battles). Set `SAMPLE = None` in `src/snowpark/battles.py` for the full 640k-pair cross join.
- The Cortex model defaults to `mistral-large2`. Swap to `llama3.1-70b` if you prefer.
- Images are uploaded to an internal stage; the UI reads them from the local filesystem for speed.
