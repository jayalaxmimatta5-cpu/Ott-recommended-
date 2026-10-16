# AURA STREAM — OTT Content Recommender

A full-stack, taste-based streaming discovery web application and recommendation engine demonstrating **DMGT** (genre equivalence classes), **ADSA** (undirected weighted co-watch graphs), **OOPJ** (strongly-typed Java domain models), and **Python** (user-based k-nearest-neighbor collaborative filtering with cosine similarity and cold-start fallback).

---

## 🌟 Architecture & Core Concepts

| Module | Concept | Implementation |
| :--- | :--- | :--- |
| **DMGT (U2)** | **Genre Equivalence Relation & Partitions** | Equivalence relation $x \sim y \iff \text{normalize}(\text{genre}(x)) = \text{normalize}(\text{genre}(y))$. Partitions the 2,500 titles into disjoint equivalence classes $[x]_R$, exported to `data/genre_equivalence_classes.csv`. |
| **ADSA (U2)** | **Undirected Weighted Co-Watch Graph** | Graph $G = (V, E, w)$ where vertices are content IDs and edge weights $w(u, v)$ represent the number of distinct viewers who co-watched both titles. Provides $O(1)$ adjacency queries for related content and network hub identification. Exported to `data/co_watch_graph.csv`. |
| **OOPJ** | **Typed Domain Architecture** | Immutable Java records & models: `Content`, `ViewingRecord`, `Recommendation`, `GenreEquivalenceClass`, `CoWatchGraph`. Handles multi-source history loading, pipeline coordination, and output generation. |
| **Python** | **k-NN Collaborative Filtering & Fallback** | User-based collaborative filtering using cosine similarity over viewer binary interaction vectors. Automatically falls back to popularity/trending scoring for cold-start viewers with clear algorithmic explanation tags. Exported to `data/recommendations.csv`. |
| **Web UI** | **Modern Streaming Discovery Site** | Responsive obsidian cinema dark design with generative poster artwork, featured hero carousel, personalized rails, multi-faceted filtering, title details modal with co-watch neighbors, watchlist, watch history, and viewer profile switcher. |

---

## 📂 Data Structure & Explicit Mismatch Handling

The project explicitly handles the separation between sample demo records and the main catalogue:

1. **Main Streaming Catalogue:** `data/ott_movies_clean_unique.csv`
   - Contains 2,500 real titles with fields: `content_id`, `title`, `type`, `genre`, `platform`, `country`, `language`, `release_year`, `duration_minutes`, `rating`, `votes`, `weighted_rating`, `popularity_score`, `trending_score`, `tags`, `description`.
   - Content IDs follow the `C100...` scheme.

2. **Synthetic Demo Watch Histories:** `data/synthetic_demo_viewing_records.csv`
   - Clearly labeled synthetic demo data containing realistic viewing clusters across 10 personas (Sci-Fi, Cyber, Drama, Action, Comedy, K-Drama, Cinephile, etc.) with intentional taste overlaps to showcase k-NN collaborative filtering.

3. **Interactive User History:** `data/user_viewing_records.csv`
   - Persists actual viewer interactions (when a viewer clicks "Mark as Watched" or "Remove from Watched").

4. **User Watchlist:** `data/user_watchlist.json`
   - Persists saved titles across sessions.

5. **Legacy Toy Sample Records:** `data/viewing_records.csv` & `data/catalog.csv`
   - Preserved for backwards compatibility with the 12-item toy demo (`M101`–`M112`). The mismatch with the 2,500 catalogue is handled explicitly: toy IDs are never silently mapped to unrelated catalogue titles.

---

## 🚀 Running the Web Application

### Option A: One-Click Launcher (Windows)
Double-click `run.bat` or run in PowerShell:
```powershell
.\run.ps1
```

### Option B: Manual Command
From the project workspace root:
```powershell
# 1. Compile Java engine
javac -d build src/main/java/ott/recommender/*.java

# 2. Start the full-stack server
python python/server.py 8000
```
Open **[http://localhost:8000/](http://localhost:8000/)** in your browser.

---

## 💻 Running the Java / Python Pipeline via CLI

You can run the recommendation pipeline directly in terminal:

### Running on the 2,500-Title Catalogue:
```powershell
# Run collaborative filtering for the Sci-Fi demo persona:
java -cp build ott.recommender.Main --catalog data/ott_movies_clean_unique.csv --user demo_scifi_fan --top 5

# Run for a brand-new viewer demonstrating cold-start fallback:
java -cp build ott.recommender.Main --catalog data/ott_movies_clean_unique.csv --user demo_coldstart_viewer --top 5

# Run for your interactive profile:
java -cp build ott.recommender.Main --catalog data/ott_movies_clean_unique.csv --user active_user --top 5
```

### Backwards-Compatible Toy Catalog Run:
```powershell
java -cp build ott.recommender.Main --catalog data/catalog.csv --history data/viewing_records.csv --user u01 --top 5
```

### Options:
- `--catalog <path>`: path to catalogue CSV (default: `data/ott_movies_clean_unique.csv`)
- `--history <paths>`: comma-separated viewing record files (default: synthetic demo + user records)
- `--user <id>`: viewer profile ID (e.g. `active_user`, `demo_scifi_fan`, `demo_coldstart_viewer`)
- `--top <number>`: number of recommendations to return (default: `5`)
- `--neighbors <number>`: maximum similar viewers used in k-NN (default: `3`)
- `--json`: emit output in structured JSON format
- `--query-graph <id>`: display co-watch graph neighbors for a specific content ID

---

## 🎨 Website Features

1. **Featured Hero Carousel:** Rotating marquee highlighting top-ranked blockbusters with platform badges, runtime, genres, and quick action buttons.
2. **Personalized "Picked For You" Rail:**
   - Powered by Python k-NN collaborative filtering with cosine similarity.
   - Shows explanation chips: e.g. `k-NN collaborative filtering: Matched with Kiran Rao (62% taste match)`.
   - Automatically switches to **Cold-Start Fallback Mode** when a viewer has insufficient history (e.g. `Jordan Lee`).
3. **ADSA Co-Watch Graph Network Rail:** Displays titles acting as network hubs with the highest degree and shared-viewer edge weights.
4. **Trending Across OTT:** Ranked by `trending_score` across Netflix, Prime Video, and Hotstar.
5. **DMGT Genre Equivalence Class Spotlights:** Interactive tabs exploring the disjoint mathematical partitions of the catalogue.
6. **Multi-Faceted Search & Explore:**
   - Real-time search across titles, synopsis, tags, and cast.
   - Filters: Genre, Platform, Type (Movie/Series), Country, Language.
   - Sorting: Trending, Popularity, Rating, Release Year.
   - Grid / Compact List view switcher.
7. **Title Details Modal Dialog:**
   - Full catalogue metadata display.
   - Live "Mark as Watched" and "Watchlist" toggle buttons (persisted to files).
   - "Viewers Also Watched" section powered by the ADSA co-watch graph.
   - "Same Genre Equivalence Class" section powered by DMGT partitions.
8. **Viewer Profile Switcher:** Switch between active interactive profile (`You`) and synthetic demo personas (`Sci-Fi Fan`, `Prestige Drama`, `Action Buff`, `Cold Start Demo`).
9. **Architecture & Diagnostics Tab:** In-depth explanation of DMGT, ADSA, OOPJ, and Python k-NN internals with a live button to recompute the pipeline on demand.
