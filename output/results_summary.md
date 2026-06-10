# Results Summary

All numbers below were produced by the pipeline notebook on the full Kaggle
datasets and are reproduced from its saved outputs.

## Cross-domain movie to book retrieval (about 48k synthetic pairs, full ~78k-book catalog)

| Metric | Approach A: Zero-shot | Approach B: Learned mapping |
|--------|----------------------:|----------------------------:|
| HitRate@10 | **0.4625** | 0.3055 |
| NDCG@10 | **0.2361** | 0.1666 |
| Recall@10 | **0.4625** | 0.3055 |
| Precision@10 | 0.0462 | 0.0305 |
| GenreMatch@1 | 0.3500 | **0.4060** |

Read this honestly: zero-shot wins the ranking metrics, but the evaluation pairs
were *built from* the zero-shot neighbor space, so part of that task is "can you
re-find your own nearest neighbors." The learned mapping trades some of that raw
recall for better category alignment (higher GenreMatch@1).

## Signal vs noise (mean cosine similarity)

| Pair type | Zero-shot space | Mapped space |
|-----------|----------------:|-------------:|
| Aligned (true) movie-book pairs | ~0.56 | ~0.56 |
| Random movie-book pairs | 0.254 | 0.073 |

The mapper keeps true pairs close while pushing random pairs toward "no relation,"
which sharpens the contrast between good and bad candidates.

## In-domain book NCF baseline (reference point only)

| Setup | HitRate@10 |
|-------|-----------:|
| 1-of-99 negative sampling | 0.2400 |
| Full ~78k-catalog ranking | ~0.00 |

NCF is a warm-start, in-domain recommender. It cannot operate in the
movie-only cold-start setting at all, which is exactly the gap the semantic
approaches fill. It is included as a familiar anchor, not as a head-to-head
competitor to the cross-domain numbers.

## User-level Nolan-style case study (26 movies, top-20 books)

- Genre overlap with the user's movie genres: about **0.70** (refined mapped
  retrieval); ~0.90 zero-shot vs ~0.80 mapped on raw lists.
- Average rating of recommended books: **4.18 / 5** vs global average **4.27 / 5**.

Figures for every stage are in [`figures/`](figures/).
