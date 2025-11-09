# Customer Sentiment Analysis for Market Expansion

An end-to-end analytics workflow that distills 87K Amazon furniture reviews into actionable insights for product, marketing, and operations teams. The project combines LLM-driven extraction, semantic clustering, and an interactive Streamlit storytelling dashboard to highlight what delights customers and where friction persists.

---

## Table of Contents

1. [Quick Start](#quick-start)
2. [Project Overview](#project-overview)
3. [Architecture & Techniques](#architecture--techniques)
4. [Key Findings](#key-findings)
5. [Challenges & Lessons Learned](#challenges--lessons-learned)
6. [Streamlit Storytelling Dashboard](#streamlit-storytelling-dashboard)
7. [Repository Layout](#repository-layout)
8. [Future Improvements](#future-improvements)

---

## Quick Start

### 1. Clone & set up

```bash
git clone https://github.com/devikamsba2024/Amazon_furniture_reviews.git
cd Amazon_furniture_reviews
python -m pip install --user streamlit pandas plotly numpy
```

### 2. Launch the interactive dashboard

```bash
python -m streamlit run streamlit_dashboard.py
```

Open `http://localhost:8501`, adjust the sidebar filters (ratings, verified purchase, Vine participation, date range, keywords) and walk through the tabs to explore the story.

### 3. Explore the notebook

- `Student_Project_Starter.ipynb` orchestrates the extraction pipeline, clustering, and chart generation.
- Large artifacts (`amazon_reviews_us_Furniture_v1_00_filtered.tsv`, gzipped JSON checkpoints) are already included; no external downloads required.

---

## Project Overview

- **Objective**: Identify the biggest purchase blockers, the differentiating delights, and segment-specific narratives that guide market expansion decisions.
- **Scope**: 87,260 furniture reviews (2008–2015), processed into structured pain points, positive aspects, themes, and purchase decision factors.
- **Outcome**: Executive-ready summaries, clustered personas, and an interactive dashboard to support follow-up action plans.

---

## Architecture & Techniques

- **Async LLM Extraction Pipeline** – `asyncio`/`aiohttp` fan out up to 20 OpenRouter calls with checkpointing for resiliency.
- **Structured Prompt Engineering** – JSON-constrained prompts enforce tight, 2–3 word canonical phrases.
- **Semantic Canonicalization** – SentenceTransformer embeddings plus two-pass clustering merge duplicate phrases into single concepts.
- **Frequency & Cohort Analysis** – Star-rating cohorts, Counter aggregations, and cohort comparisons highlight differentiated drivers.
- **Embedding & Clustering Workflow** – BGE-M3 embeddings, PCA + t-SNE projection, and K-Means (k=10) reveal persona clusters.
- **Visualization Suite** – Matplotlib/Seaborn plots and exported PNGs for distributions, temporal trends, and top terms.
- **Executive Reporting** – Automated JSON/Markdown summaries capture opportunity, key findings, and recommended actions.

---

## Key Findings

- **Universal friction** – “Difficult assembly” dominates across every star-rating band.
- **Quality control watch-outs** – Color inaccuracy, damaged arrivals, and wobbly builds permeate 4–5 star reviews.
- **Stable sentiment baseline** – Average sentiment and rating trends stayed steady from 2008–2015, providing a control for new signals.
- **Distinct personas** – Clustering surfaces buyer archetypes: assembly-sensitive DIYers, value hunters, style-driven decorators, and comfort seekers.

---

## Challenges & Lessons Learned

### API reliability under concurrency

- **Symptoms**: `Server disconnected`, `Connection reset by peer`, `502` errors at 20 concurrent requests.
- **Mitigation**: Periodic checkpointing, retry logic, graceful handling of failed batches.
- **Recommendation**: Drop to 10–15 concurrent calls or introduce adaptive rate limiting.

### Canonicalizing near-duplicate phrases

- **Problem**: Raw LLM output contained variants (e.g., “easy assembly” vs “easy assemble”) that fragmented metrics.
- **Solution**: Two-pass embedding clustering—tight similarity (≥0.75) followed by broader merges (≥0.60)—with canonical representatives per cluster.
- **Result**: Consolidated metrics, clearer narratives, and consistent terminology across pain points, positives, themes, and decision factors.

---

## Streamlit Storytelling Dashboard

`streamlit_dashboard.py` turns the analysis into a guided experience for stakeholders:

- **Executive Overview** – Surface the most recent executive summary alongside filters that update time-series trends and rating distributions instantly.
- **Reduce Friction / Delight Customers** – Live bar charts expose the top pain points and positives for any cohort, paired with full-market PNG references.
- **Segments & Stories** – Dig into cluster personas using t-SNE projections, top themes/decision drivers, and expandable sample reviews.
- **Compare Segments** – Side-by-side visuals contrast what top (⭐ 4–5) versus bottom (⭐ 1–2) reviewers praise or criticize, complete with key takeaways.
- **Operations & Follow-up** – Track shipping/packaging sentiment and decision-factor signals that inform cross-functional actions.

Run locally with the command above and navigate the tabs to extract narratives, copy quotes, and capture action items for executive briefings.

---

## Repository Layout

```
Amazon_furniture_reviews/
├── streamlit_dashboard.py                # Interactive storytelling dashboard
├── Student_Project_Starter.ipynb         # Primary notebook for extraction + clustering
├── amazon_reviews_us_Furniture_v1_00_filtered.tsv
├── review_insights_checkpoint.json
├── review_insights_checkpoint_cleaned.json.gz
├── furniture_insight_charts/             # Exported PNG visuals
├── executive_summary_*.json / *.md       # Generated business summaries
├── pain_points_*.json                    # Aggregated frequency metrics
├── positive_aspects_*.json
├── main_themes_*.json
├── purchase_factors_*.json
└── top_vs_bottom_comparison.md
```

---

## Future Improvements

1. **Adaptive rate limiting** – Dynamically modulate concurrency based on API error rates.
2. **Enhanced retry/backoff** – Expand resilience tactics for long-running extraction batches.
3. **Real-time monitoring** – Add streaming progress dashboards during large-scale processing.
4. **Interactive cluster explorer** – Extend the Streamlit app with a Plotly t-SNE scatter plot and persona annotator.

---
