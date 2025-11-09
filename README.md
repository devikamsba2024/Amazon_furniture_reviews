# Customer Sentiment Analysis for Market Expansion

## Project Overview

This project analyzes Amazon furniture reviews to extract customer insights, including pain points, positive aspects, temporal trends, and product quality patterns. The analysis uses Large Language Models (LLMs) for structured extraction and semantic embeddings for clustering similar insights.

---

## Techniques Used

- **Async LLM Extraction Pipeline**: Leveraged `asyncio` and `aiohttp` to fan out up to 20 concurrent OpenRouter calls with periodic checkpointing for resilience.
- **Structured Prompt Engineering**: Crafted JSON-constrained prompts to standardize pain points, positives, themes, and decision factors into 2–3 word spans.
- **Semantic Canonicalization**: Applied SentenceTransformer embeddings with two-pass clustering to merge duplicate phrases and create canonical forms.
- **Frequency & Cohort Analysis**: Used `Counter` aggregations and rating-based cohorts (low/mid/high star) to highlight differentiated drivers.
- **Visualization Suite**: Produced matplotlib/Seaborn charts for distributions, temporal trends, and top-term bar charts.
- **Embedding & Clustering Workflow**: Generated BGE-M3 embeddings, reduced dimensions via PCA + t-SNE, and grouped reviews with K-Means (k=10).
- **Temporal Insight Tracking**: Exploded yearly pain points and sentiments to monitor evolution of customer concerns and positivity.
- **Comparative Summaries**: Stratified samples from best/worst products and prompted an LLM for differentiators and action-oriented recommendations.
- **Executive Reporting**: Automated executive-summary generation (JSON + Markdown) summarizing opportunity, key findings, and next steps.

---

## Special Considerations & Challenges

### 1. API Call Errors with Concurrent Requests

**Issue Encountered:**
During the parallel processing of 87,260 reviews, several API errors occurred:
- `Server disconnected` errors
- `[Errno 54] Connection reset by peer`
- `502: Internal server error` (502 Bad Gateway)

**Root Cause:**
The concurrent request limit was set to **20 simultaneous requests** (`CONCURRENT_REQUESTS = 20`). While this improves processing speed, it may have exceeded the API provider's rate limits or overwhelmed their servers, especially during peak processing periods.

**Impact:**
- Some batches failed during processing, resulting in incomplete extractions for certain reviews
- Processing time was extended due to retry delays and connection resets
- Overall success rate: ~99.97% (87,233 successful extractions out of 87,260 reviews)

**Lessons Learned:**
- **Conservative concurrency**: Start with lower concurrent requests (10-15) and gradually increase if API stability allows
- **Retry logic**: Implement exponential backoff for failed requests to handle transient errors
- **Error handling**: The async processing framework gracefully handled failures by returning `None` for failed extractions, allowing processing to continue
- **Checkpointing**: Periodic checkpoint saves (every 1000 batches) ensured progress was preserved even when errors occurred

**Recommendation for Future Runs:**
Consider reducing `CONCURRENT_REQUESTS` to 10-15 for more stable processing, or implement a dynamic rate limiter that adjusts based on error rates.

---

### 2. Canonicalization of Similar Pain Points and Positive Aspects

**Problem Identified:**
After extracting insights from reviews, many pain points and positive aspects had similar wording or meaning but were counted separately, leading to fragmented insights:

- **Examples of duplicates:**
  - "easy assembly" vs "easy assemble"
  - "looks great" vs "looks nice" vs "looks good"
  - "very sturdy" vs "sturdy"
  - "great price" vs "good price"
  - "difficult assembly" vs "assembly difficulty" vs "assembly time"

**Solution Implemented:**
A **canonicalization function** was developed to semantically group similar phrases and map them to a single representative (canonical) form.

**How the Canonicalization Process Works:**

1. **Semantic Embedding**: 
   - Uses SentenceTransformer model (`all-MiniLM-L6-v2`) to convert phrases into dense vector embeddings
   - Captures semantic meaning rather than exact text matching
   - Processes the top 300 most frequent phrases to limit computational cost

2. **Two-Pass Clustering**:
   - **First Pass (Tight Threshold: 0.75)**: Groups very similar phrases
     - Example: "comfortable" ↔ "comfort", "easy assembly" ↔ "easy assemble"
     - Uses cosine similarity ≥ 0.75 to identify near-identical meanings
   
   - **Second Pass (Looser Threshold: 0.60)**: Merges related first-pass clusters
     - Example: "durable" ↔ "sturdy", "great price" ↔ "good price" ↔ "low price"
     - Uses cosine similarity ≥ 0.60 to capture broader semantic relationships

3. **Representative Selection**:
   - The first unassigned phrase in each cluster becomes the canonical representative
   - All similar phrases map to this representative
   - Ensures consistent terminology across the dataset

4. **Application**:
   - Applied to four dimensions: `pain_points`, `positive_aspects`, `main_themes`, and `purchase_decision_factors`
   - Creates new columns with `_clean` suffix (e.g., `pain_points_clean`)
   - Original columns preserved for traceability

**Benefits:**
- **Consolidated insights**: Similar phrases are grouped, providing clearer, more actionable insights
- **Better aggregation**: Frequency counts are more accurate when duplicates are merged
- **Preserved semantics**: Uses embedding-based similarity, so "fast shipping" and "quick delivery" are recognized as similar even with different wording
- **Maintains precision**: Two-pass approach balances precision (first pass) with broader grouping (second pass)

**Example Transformation:**
```
Before Canonicalization:
- "easy assembly": 7,796 mentions
- "easy assemble": 5,564 mentions
- Total: 13,360 mentions across 2 separate categories

After Canonicalization:
- "easy assembly": 13,360 mentions (canonical form)
- "easy assemble" → mapped to "easy assembly"
```

**Result:**
The canonicalization process significantly improved the quality of aggregated insights, reducing fragmentation and providing clearer patterns in customer feedback.

---

## Key Technical Decisions

1. **Model Selection**: Used `google/gemini-2.5-flash-lite-preview-09-2025` for LLM extraction (fast and cost-effective for large-scale processing)

2. **Embedding Model**: `BAAI/bge-m3` for review embeddings and clustering analysis

3. **Parallel Processing**: Asynchronous processing with `aiohttp` and `asyncio` to handle 87K+ reviews efficiently

4. **Checkpoint Strategy**: Saved progress every 1000 batches to enable recovery from interruptions

5. **Data Cleaning**: Canonicalization ensures consistent terminology across extracted insights

---

## Interesting Findings

- **Top Pain Point**: "difficult assembly" appears most frequently across all rating categories, indicating a universal customer concern
- **Quality Control Issues**: Even high-rated (4-5 star) products have pain points, primarily related to minor assembly challenges and color accuracy
- **Sentiment Trends**: Overall sentiment remained relatively stable over the analysis period (2008-2015)
- **Clustering Insights**: K-Means clustering on review embeddings revealed 10 distinct customer segments with varying focus (quality, price, aesthetics, etc.)

---

## File Structure

- `Student_Project_Starter.ipynb`: Main analysis notebook
- `review_insights_checkpoint_cleaned.json`: Processed insights with canonicalized phrases
- `furniture_insight_charts/`: Generated visualizations (PNG files)
- `executive_summary_*.json` & `executive_summary_*.md`: Business recommendations
- `pain_points_*.json`, `positive_aspects_*.json`: Aggregated frequency data

---

## Future Improvements

1. **Rate Limiting**: Implement adaptive rate limiting based on API error rates
2. **Retry Logic**: Add exponential backoff for failed API calls
3. **Real-time Monitoring**: Add progress dashboards to track API health during processing


