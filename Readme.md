# 🎬 Movie Recommender

A full-stack movie recommendation web app powered by **TF-IDF content-based filtering** and the **TMDB API**, built with FastAPI + Streamlit.

🔗 **Live Demo**: [movie-recommendation-almprfbdtejwn3yheeycew.streamlit.app](https://movie-recommendation-almprfbdtejwn3yheeycew.streamlit.app)

---

## ✨ Features

- 🔍 **Keyword Search** — Search movies with real-time TMDB suggestions and dropdown autocomplete
- 🧠 **TF-IDF Recommendations** — Content-based similarity using a pre-trained TF-IDF matrix on a local movie dataset
- 🎭 **Genre Recommendations** — TMDB-powered "More Like This" using genre discovery
- 🏠 **Home Feed** — Browse Trending, Popular, Top Rated, Now Playing, and Upcoming movies
- 🖼️ **Movie Details Page** — Full details with poster, backdrop, genres, overview, and dual recommendations
- 📱 **Responsive Grid** — Adjustable column layout (4–8 columns)

---

## 🏗️ Architecture

```
┌─────────────────────┐         ┌──────────────────────────┐
│   Streamlit (UI)    │ ──────▶ │   FastAPI (Backend)      │
│   app.py            │         │   main.py                │
│                     │         │                          │
│  • Search bar       │         │  • /home                 │
│  • Poster grid      │         │  • /tmdb/search          │
│  • Details view     │         │  • /movie/id/{id}        │
│  • Recommendations  │         │  • /movie/search         │
└─────────────────────┘         │  • /recommend/genre      │
                                │  • /recommend/tfidf      │
                                └────────────┬─────────────┘
                                             │
                          ┌──────────────────┼──────────────────┐
                          ▼                  ▼                  ▼
                    TMDB API           df.pkl             tfidf_matrix.pkl
                  (posters,           (movie             (cosine similarity
                   details,           dataset)            vectors)
                  discover)
```

---

## 🗂️ Project Structure

```
movie-recommendation/
│
├── movies.ipynb            # EDA + ML pipeline (Jupyter Notebook)
├── movies_metadata.csv     # Raw dataset (~32MB)
│
├── app.py                  # Streamlit frontend
├── main.py                 # FastAPI backend
│
├── df.pkl                  # Processed Movie DataFrame
├── indices.pkl             # Title → row index mapping
├── tfidf.pkl               # Fitted TF-IDF vectorizer
├── tfidf_matrix.pkl        # Pre-computed TF-IDF sparse matrix
│
├── requirements.txt
├── runtime.txt             # Python 3.11.9
├── README.md
└── .gitignore
```

---

## 📦 Dataset

- **Source**: [TMDB Movies Metadata — Kaggle](https://www.kaggle.com/datasets/rounakbanik/the-movies-dataset)
- **File**: `movies_metadata.csv` (~32MB, included in this repo)
- **Key columns used**: `title`, `overview`, `genres`, `tagline`, `vote_average`, `popularity`

---

## 🚀 Getting Started

### Prerequisites

- Python 3.11.9
- A [TMDB API key](https://www.themoviedb.org/settings/api) (free)

### 1. Clone the repository

```bash
git clone https://github.com/Tushar6405/movie-recommendation.git
cd movie-recommendation
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Configure environment

Create a `.env` file in the root directory:

```env
TMDB_API_KEY=your_tmdb_api_key_here
```

### 4. Run the FastAPI backend

```bash
uvicorn main:app --reload --port 8000
```

The API will be available at `http://127.0.0.1:8000`.  
Interactive docs: `http://127.0.0.1:8000/docs`

### 5. Run the Streamlit frontend

In a new terminal:

```bash
streamlit run app.py
```

> **Note:** By default, `app.py` points to the deployed Render backend. For local development, update the `API_BASE` in `app.py`:
> ```python
> API_BASE = "http://127.0.0.1:8000"
> ```

---

## 🔌 API Reference

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/health` | Health check |
| `GET` | `/home` | Home feed (trending/popular/etc.) |
| `GET` | `/tmdb/search?query=` | TMDB keyword search |
| `GET` | `/movie/id/{tmdb_id}` | Full movie details |
| `GET` | `/movie/search?query=` | Bundle: details + TF-IDF + genre recs |
| `GET` | `/recommend/tfidf?title=` | TF-IDF recommendations only |
| `GET` | `/recommend/genre?tmdb_id=` | Genre-based recommendations |

### Example

```bash
# Search for a movie
curl "http://localhost:8000/tmdb/search?query=inception"

# Get recommendations bundle
curl "http://localhost:8000/movie/search?query=Inception&tfidf_top_n=10&genre_limit=10"
```

---

## 📊 Exploratory Data Analysis & ML Pipeline

The recommendation model was built in a Jupyter Notebook (`movies.ipynb`) using the TMDB Movies Metadata dataset.

### 1. EDA
- Loaded `movies_metadata.csv` and explored shape, columns, and data types
- Identified and handled missing values (`isnull().sum()`)
- Removed duplicate records
- Selected relevant features: `title`, `overview`, `genres`, `tagline`, `vote_average`, `popularity`

### 2. Data Preprocessing
- Parsed `genres` from raw JSON strings using `ast.literal_eval`
- Filled missing `overview` and `tagline` with empty strings
- Combined `overview + genres + tagline` into a single `tags` column for vectorization

### 3. NLP Text Processing
- Lowercased all text and removed punctuation
- Removed **stopwords** using NLTK
- Applied **Lemmatization** (WordNetLemmatizer) to normalize words

### 4. TF-IDF Model
- Vectorized `tags` using `TfidfVectorizer` with `max_features=50,000` and `ngram_range=(1,2)`
- Computed **cosine similarity** between movie vectors
- Saved model artifacts as `.pkl` files for API use

```python
tfidf = TfidfVectorizer(max_features=50000, ngram_range=(1,2), stop_words='english')
tfidf_matrix = tfidf.fit_transform(df['tags'])
```

---

## 🧠 How Recommendations Work

### TF-IDF (Content-Based Filtering)

1. Movie metadata (titles, descriptions, genres) is vectorized using **TF-IDF** at build time
2. Vectors are stored in a **sparse matrix** (`tfidf_matrix.pkl`)
3. At query time, cosine similarity is computed between the selected movie and all others
4. Top-N most similar movies are returned, then enriched with TMDB poster data

### Genre-Based Filtering

1. Fetches the primary genre of the selected movie from TMDB
2. Queries TMDB's `/discover/movie` endpoint filtered by that genre
3. Results are sorted by popularity

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | [Streamlit](https://streamlit.io/) |
| Backend | [FastAPI](https://fastapi.tiangolo.com/) |
| ML / Similarity | scikit-learn, scipy (TF-IDF + cosine similarity) |
| Data | pandas, numpy |
| Movie Data | [TMDB API](https://www.themoviedb.org/) |
| HTTP Client | httpx |
| Deployment | Streamlit Cloud + Render |

---

## ☁️ Deployment

The project is split across two cloud services:

- **Frontend** → [Streamlit Cloud](https://streamlit.io/cloud)
- **Backend** → [Render](https://render.com) (FastAPI via `uvicorn`)

### Deploy Backend on Render

1. Connect your GitHub repo to Render
2. Set **Start Command**: `uvicorn main:app --host 0.0.0.0 --port 10000`
3. Add environment variable: `TMDB_API_KEY=your_key`
4. Update `API_BASE` in `app.py` to your Render URL

### Deploy Frontend on Streamlit Cloud

1. Push code to GitHub
2. Go to [share.streamlit.io](https://share.streamlit.io) and connect your repo
3. Set **Main file path** to `app.py`

---

## 📄 License

This project is open source and available under the [MIT License](LICENSE).

---

## 🙏 Acknowledgements

- Movie data and images provided by [The Movie Database (TMDB)](https://www.themoviedb.org/)
- *This product uses the TMDB API but is not endorsed or certified by TMDB.*