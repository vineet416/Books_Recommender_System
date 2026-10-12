# 📚 Books Recommender System

An item-based collaborative filtering recommender built on the Book-Crossing ratings data. It finds books that the same readers rated similarly, using k-Nearest Neighbors. A Streamlit app returns five similar books with cover images. The notebook documents the full data preparation and model training.

**Live demo:** https://books-recommender-system-vineet416.streamlit.app/

## Project Files
 
| File | Description |
|---|---|
| `app.py` | Streamlit app: pick a book, get 5 similar books with covers |
| `research.ipynb` | Loading, cleaning, filtering, pivot table, model training, artifact export |
| `artifacts/model.pkl` | Fitted `NearestNeighbors` model (brute force, Euclidean) |
| `artifacts/book_pivot.pkl` | Book × user rating matrix (742 × 888) |
| `artifacts/book_names.pkl` | The 742 titles shown in the dropdown |
| `artifacts/final_rating.pkl` | Cleaned ratings joined with book metadata (59,850 rows) |
| `requirements.txt` | Python dependencies (unpinned) |
| `.devcontainer/devcontainer.json` | Python 3.11 dev container that starts the app on port 8501 |
| `LICENSE` | MIT license |

## Dataset

Source: Book-Crossing dataset, read from `Datasets/` (not in this repo): `BX-Books.csv` (271,360 rows), `BX-Users.csv` (278,858), `BX-Book-Ratings.csv` (1,149,780). Modeling table `final_rating`: 59,850 rows × 9 columns, no missing values.

| Column | Description |
|---|---|
| `user_id` | Anonymous reader ID (888 unique) |
| `ISBN` | Edition ID (2,176 unique) |
| `rating` | 0–10; 0 means no explicit rating |
| `title` / `author` | Book title (742 unique) and author (584 unique) |
| `year` / `publisher` | Publication year (text) and publisher (357 unique) |
| `image_url` | Large cover image URL |
| `number_of_ratings` | Title's rating count before de-duplication (50–363) |

## Approach

1. **Understand the data:** load the three CSVs, check shapes, rename columns, drop small and medium image columns.
2. **Filter users:** keep users with more than 200 ratings (899 of 105,283 raters) → 526,356 ratings.
3. **Join and filter books:** merge ratings with books on ISBN, keep titles with at least 50 ratings, drop duplicate (user, title) pairs.
4. **Build the matrix:** pivot to title × user, fill missing with 0, convert to a sparse CSR matrix.
5. **Train:** `NearestNeighbors(algorithm='brute')`, k = 6 (the book itself + 5 neighbors).
6. **Serve:** pickle four artifacts; `app.py` looks up neighbors and fetches cover URLs.

## Key Insights

Recomputed from the artifacts (the raw CSVs are not available).

| Finding | Value |
|---|---|
| Data kept after filters | 59,850 of 1,149,780 ratings (5.2%) |
| Explicit ratings (1–10) | 14,961 (25.0%); the other 75.0% are zeros. Matrix density is 2.27% |
| Explicit rating average / median | 7.96 / 8; 66.7% of ratings are 8–10, only 2.9% are 1–4 |
| Most explicitly rated | *The Lovely Bones* (104, avg 8.32), *Bridget Jones's Diary* (84, 7.49), *The Da Vinci Code* (82, 8.59) |
| Highest average (≥ 20 ratings) | *The Two Towers* 9.74, *The Return of the King* 9.57, *The Secret Garden* 9.46 |
| Most rows, few real ratings | *Wild Animus*: 363 rows, only 38 explicit, avg 4.18 |
| Example output | *Harry Potter and the Chamber of Secrets* → Books 4, 3, 1, then *Exclusive*, *The Cradle Will Fall* |
| "Hub" books | *Exclusive* is in 579 of 742 top-5 lists, *No Safe Place* in 515 (3 explicit ratings each); 434 books (58.5%) are never recommended |

## Tech Stack

- **Language and data:** Python 3.11, pandas, NumPy, SciPy (sparse CSR matrix)
- **Model:** scikit-learn `NearestNeighbors` (item-based collaborative filtering)
- **App and tooling:** Streamlit, `pickle`, Jupyter Notebook, VS Code Dev Container

## How to Use

1. Install dependencies: `pip install -r requirements.txt`
2. Start the app: `streamlit run app.py` (opens at `http://localhost:8501`)
3. Choose a book, then click **Show Recommendation**.
4. To retrain, place the three CSVs in `Datasets/` and run all cells in `research.ipynb`.
