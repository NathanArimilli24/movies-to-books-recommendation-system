# Data

This project uses three public Kaggle datasets. The full files are large
(about 2.9 GB combined), so they are **not** committed to the repository.
Small, ready-to-browse samples live in [`sample/`](sample/) so you can see the
exact schema without downloading anything.

## Full datasets (download from Kaggle)

| File | Rows (full) | Source |
|------|-------------|--------|
| `rotten_tomatoes_movies.csv` | 17,712 movies | [Rotten Tomatoes Movies and Critic Reviews](https://www.kaggle.com/datasets/stefanoleone992/rotten-tomatoes-movies-and-critic-reviews-dataset) |
| `books_data.csv` | 212,404 book titles | [Amazon Books Reviews](https://www.kaggle.com/datasets/mohamedbakhet/amazon-books-reviews) |
| `Books_rating.csv` | 3,000,000 reviews | [Amazon Books Reviews](https://www.kaggle.com/datasets/mohamedbakhet/amazon-books-reviews) |

To reproduce the full pipeline, download the three files from the links above
and place them in this `data/` folder (next to this README).

## Samples in `sample/`

These are small, coherent extracts meant to illustrate the data, not to
reproduce the headline results:

- `rotten_tomatoes_movies_sample.csv` — 3,000 randomly sampled movies (trimmed to
  the columns the pipeline actually uses).
- `books_data_sample.csv` — metadata for 500 book titles.
- `Books_rating_sample.csv` — 6,221 reviews covering those same 500 titles
  (each title has at least 5 reviews, capped at 20, matching the preprocessing
  rules used in the notebook).

The samples were drawn so that every sampled book has both metadata and reviews,
which keeps the `Title` joins intact if you want to dry-run the early
preprocessing steps.

## A note on contents

`Books_rating.csv` includes a `User_id` and a public Amazon `profileName` for
each review. This is part of the original public Kaggle dataset and is reproduced
here unchanged; no private or scraped data was added.
