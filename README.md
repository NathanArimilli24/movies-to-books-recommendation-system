# Semantic Bridges: LLM-Based Cross-Domain Recommendation from Movies to Books

> **Goal:** given only a user's *movie* taste, recommend *books* they are likely to
> enjoy, with no shared users, no joint ratings, and no cross-domain IDs.

Most recommenders live inside one domain: Netflix suggests movies, Goodreads
suggests books. But a fan of dark, twisty sci-fi films might also love cerebral
science-fiction novels with the same tone. This project asks whether modern
LLM sentence embeddings can bridge that gap and turn "movies I like" into
"books I should read," even when a traditional collaborative filter would see
the user as a completely empty row.

The full pipeline lives in one notebook:
[`code/semantic_bridges_pipeline.ipynb`](code/semantic_bridges_pipeline.ipynb).

There is also a write-up of the project on Medium:
[Semantic Bridges: LLM-Based Cross-Domain Recommendation from Movies to Books](https://medium.com/@nathan.arimilli/semantic-bridges-llm-based-cross-domain-recommendation-from-movies-to-books-437a187c4ded).

**Team:** Bhagya Puppala, Cassandre Korvink, Janani Vakkanti, Muskan Khepar, Nathan Arimilli.

---

## The idea in one picture

```
movies a user likes  ──embed──►  shared semantic space  ◄──embed──  book catalog
   (Rotten Tomatoes)              (all-mpnet-base-v2)                (Amazon reviews)
            │                                                              │
            └──────── average liked movies → user vector ─────────────────┘
                                   │
                       cosine similarity → top-K books
```

We build a single 768-dimensional vector per movie and per book by fusing plot,
reviews, metadata, and rating text, then compare two ways of crossing the gap:

1. **Approach A, Zero-shot retrieval.** Embed both domains with the same model
   and retrieve nearest-neighbor books directly. No training.
2. **Approach B, Learned mapping.** Train a small MLP with triplet loss to map
   movie vectors into book space, using synthetic movie-book pairs built from
   the zero-shot neighbors.

A book-only Neural Collaborative Filtering (NCF) model is included as an
in-domain reference point.

---

## Results

Cross-domain retrieval on about 48k synthetic movie-book pairs, ranking against
the full ~78k-book catalog:

| Metric | Approach A: Zero-shot | Approach B: Learned mapping |
|--------|----------------------:|----------------------------:|
| HitRate@10 | **0.4625** | 0.3055 |
| NDCG@10 | **0.2361** | 0.1666 |
| GenreMatch@1 | 0.3500 | **0.4060** |

In-domain book NCF baseline: HitRate@10 ≈ **0.24** (1-of-99), dropping to ~0 on
full-catalog ranking, and it cannot do the cross-domain task at all.

**How to read this honestly.** Zero-shot wins the ranking metrics, but that is
partly by construction: the evaluation pairs were generated from the zero-shot
neighbor space, so the task partly rewards a model for re-finding its own
neighbors. The learned mapping gives up some of that raw recall in exchange for
better category alignment (higher GenreMatch@1) and a much cleaner separation
between real and random pairs: the mean cosine similarity of random movie-book
pairs falls from 0.25 in the zero-shot space to 0.07 after mapping, while true
pairs stay around 0.56.

**User-level case study (Nolan-style profile, 26 movies).** Top-20 book
recommendations land mostly in crime/thriller fiction, sci-fi, and graphic
novels, with about 70% genre overlap and an average book rating of 4.18/5 versus
a global average of 4.27/5.

Full numbers and every figure are in [`output/`](output/).

---

## Repository structure

```
.
├── code/
│   ├── semantic_bridges_pipeline.ipynb   # the full pipeline, with saved outputs
│   └── requirements.txt
├── data/
│   ├── README.md                         # Kaggle links + how to get full data
│   └── sample/                            # small, browsable samples of all 3 CSVs
├── output/
│   ├── results_summary.md                # results write-up
│   ├── metrics.csv                        # machine-readable metrics
│   └── figures/                          # 15 figures from the pipeline
├── report/
│   ├── Semantic_Bridges_Report.pdf       # full written report
│   └── Semantic_Bridges_Slides.pptx      # presentation deck
└── README.md
```

---

## How it works

**1. Data and preprocessing.** Three public Kaggle CSVs (see
[`data/README.md`](data/README.md)). Movie metadata from Rotten Tomatoes; book
metadata and ~3M user reviews from Amazon. Reviews are the primary signal for
books, so we keep only books with at least 5 reviews and cap each at 20. Text is
cleaned for LLM embeddings (lowercase, strip HTML/URLs/emojis, collapse
whitespace) without stemming or stopword removal, and numeric ratings are turned
into short phrases like "very highly rated."

**2. Component embeddings and fusion.** Each item is embedded as several
components with `all-mpnet-base-v2`, then fused with a weighted average and
L2-normalized:

- Movies: plot 0.50, consensus 0.30, metadata 0.10, ratings 0.10.
- Books: reviews 0.45, description 0.25, metadata 0.10, ratings 0.20.

**3. Models.** A book-only NCF baseline; zero-shot retrieval; and the MLP mapping
network (Linear 768→512, ReLU, Linear 512→768, L2-normalize) trained with
`TripletMarginLoss(margin=0.2)` for 9 epochs.

**4. Evaluation and visualization.** System-level retrieval metrics
(HitRate@10, NDCG@10, Recall@10, Precision@10, GenreMatch@1), a Nolan-style
user case study, and t-SNE, heatmap, radar, and network diagnostics of the
shared space.

---

## Reproducing this

> **Heads up:** end-to-end reproduction needs the full Kaggle datasets (about
> 2.9 GB) **and a GPU**. Embedding hundreds of thousands of reviews with MPNet
> is slow on CPU. The notebook ships with all of its outputs saved, so you can
> read the full story without running anything.

1. Get the data: download the three CSVs into `data/` per
   [`data/README.md`](data/README.md).
2. Install dependencies:
   ```bash
   pip install -r code/requirements.txt
   ```
3. Launch and run the notebook in order:
   ```bash
   jupyter lab code/semantic_bridges_pipeline.ipynb
   ```

The notebook caches intermediate embeddings to an `embeddings_cache/` folder, so
later phases do not re-embed. Each phase also has a bootstrap cell that reloads
saved artifacts, which lets you re-run individual phases without redoing the
whole pipeline.

---

## Honest limitations

- **The evaluation pairs are synthetic and self-referential.** They are built
  from zero-shot nearest neighbors, so the ranking benchmark structurally favors
  zero-shot. There is no real movie-to-book interaction data to validate against.
- **The learned mapping does not beat zero-shot on recall.** Its value is in
  category alignment and signal/noise contrast, not raw retrieval accuracy.
- **The movie-side collaborative model is a toy.** With only critics and
  audience as two "users," it is exploratory; the book NCF is the real baseline.
- **Reproducibility is GPU- and data-bound.** The committed outputs come from a
  full GPU run; the samples in `data/sample/` are illustrative, not enough to
  reproduce the headline metrics.

## Reuse and extension

The same recipe transfers to other text-heavy cross-domain settings (games to
books, podcasts to books). Natural next steps: real cross-domain interaction
data to train the mapping, an end-to-end contrastive dual-encoder, FAISS for
large-scale retrieval, and richer re-ranking on sentiment, pacing, or style.
