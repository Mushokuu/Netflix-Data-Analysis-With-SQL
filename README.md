# 📊 Netflix Dataset Analysis

## 📌 Introduction

This project explores the Netflix dataset to uncover insights about its content library. Using SQL queries, we analyze ratings, genres, release decades, and age certifications to understand how Netflix curates its catalog. The study highlights top-performing titles, audience preferences, and overall content distribution across different categories.

The findings can help in understanding:
- What types of movies/shows resonate with audiences
- How Netflix's content library has evolved over time
- The influence of age certifications on ratings and availability
- Which genres dominate the platform

---

## 🔍 Analysis

# Netflix Dataset Analysis

## 1. Top and Bottom Ranked Movies and Shows (IMDB Scores)

### SQL Queries:

**Top 10 Movies:**
```sql
SELECT title, 
       type, 
       imdb_score
FROM shows_movies.titles
WHERE imdb_score >= 8.0
  AND type = 'MOVIE'
ORDER BY imdb_score DESC
LIMIT 10;
```

**Top 10 Shows:**
```sql
SELECT title, 
       type, 
       imdb_score
FROM shows_movies.titles
WHERE imdb_score >= 8.0
  AND type = 'SHOW'
ORDER BY imdb_score DESC
LIMIT 10;
```

**Bottom 10 Movies:**
```sql
SELECT title, 
       type, 
       imdb_score
FROM shows_movies.titles
WHERE type = 'MOVIE'
ORDER BY imdb_score ASC
LIMIT 10;
```

**Bottom 10 Shows:**
```sql
SELECT title, 
       type, 
       imdb_score
FROM shows_movies.titles
WHERE type = 'SHOW'
ORDER BY imdb_score ASC
LIMIT 10;
```

### Findings:
- **High-performers:** Top-rated titles consistently show exceptional IMDB scores (8.0+), indicating strong audience approval and critical acclaim from global viewers
- **Low-performers:** Bottom-rated titles exhibit significantly lower scores, often reflecting weaker plots, production shortcomings, or limited mainstream audience appeal
- **Strategic value:** This comparison provides valuable benchmarks for identifying high-performing content and areas requiring strategic improvement in content curation

---

## 2. Distribution of Movies and Shows by Decade

### SQL Query:
```sql
SELECT CONCAT(FLOOR(release_year / 10) * 10, 's') AS decade,
       COUNT(*) AS movies_shows_count
FROM shows_movies.titles
WHERE release_year >= 1940
GROUP BY CONCAT(FLOOR(release_year / 10) * 10, 's')
ORDER BY decade;
```

### Findings:
- **Historical content:** Limited representation exists from 1940s-1980s, indicating selective curation of older titles
- **Growth phases:** Significant expansion beginning in the 1990s (**121 titles**) marked the start of modern content acquisition
- **Modern dominance:** The **2010s** dominate with **3,304 titles**, while the **2020s** already contribute **1,972 titles** despite being incomplete
- **Strategic focus:** This demonstrates Netflix's strategic focus on prioritizing contemporary content that aligns with current viewing preferences while maintaining curated classics

---

## 3. Impact of Age Certifications

### SQL Queries:

**Average scores by certification:**
```sql
SELECT DISTINCT age_certification, 
       ROUND(AVG(imdb_score),2) AS avg_imdb_score,
       ROUND(AVG(tmdb_score),2) AS avg_tmdb_score
FROM shows_movies.titles
GROUP BY age_certification
ORDER BY avg_imdb_score DESC;
```

**Most common certifications:**
```sql
SELECT age_certification, 
       COUNT(*) AS certification_count
FROM shows_movies.titles
WHERE type = 'Movie' 
  AND age_certification != 'N/A'
GROUP BY age_certification
ORDER BY certification_count DESC
LIMIT 5;
```

### Findings:
- **Top performer:** **TV-14** titles achieved the highest average IMDB score (**6.71**), demonstrating strong audience engagement with content aimed at teenage and adult demographics
- **Volume leaders:** **R-rated movies** dominate the catalog with **556 titles**, followed by **PG-13** (451) and **PG** (233), reflecting diverse audience preferences
- **Strategic positioning:** The prevalence of mature content categories highlights Netflix's strategic focus on adult audiences while maintaining substantial family-friendly viewing options for broader demographic appeal

---

## 4. Most Common Genres

### SQL Queries:

**Top movie genres:**
```sql
SELECT genres, 
       COUNT(*) AS title_count
FROM shows_movies.titles 
WHERE type = 'Movie'
GROUP BY genres
ORDER BY title_count DESC
LIMIT 10;
```

**Top show genres:**
```sql
SELECT genres, 
       COUNT(*) AS title_count
FROM shows_movies.titles 
WHERE type = 'Show'
GROUP BY genres
ORDER BY title_count DESC
LIMIT 10;
```

**Overall top genres:**
```sql
SELECT t.genres, 
       COUNT(*) AS genre_count
FROM shows_movies.titles AS t
WHERE t.type = 'Movie' OR t.type = 'Show'
GROUP BY t.genres
ORDER BY genre_count DESC
LIMIT 3;
```

### Findings:
- **Movies:** Comedy leads with 384 titles, followed by Documentary (230) and Drama (224), showing diverse storytelling preferences across audiences
- **Shows:** Reality programming tops with 113 entries, then Drama (104) and Comedy (100), reflecting growing demand for unscripted content
- **Overall rankings:** Comedy (484), Documentary (329), Drama (328) demonstrate consistent cross-format appeal
- **Strategic insight:** Comedy's universal appeal across both movies and shows confirms its fundamental role in Netflix's global content strategy and audience retention

---

## 5. Average Runtime of Movies vs Shows

### SQL Query:
```sql
SELECT 'Movies' AS content_type,
       ROUND(AVG(runtime),2) AS avg_runtime_min
FROM shows_movies.titles
WHERE type = 'Movie'
UNION ALL
SELECT 'Show' AS content_type,
       ROUND(AVG(runtime),2) AS avg_runtime_min
FROM shows_movies.titles
WHERE type = 'Show';
```

### Optimization:
**Before index:** `EXPLAIN` shows full scan on `titles` for each branch of the `UNION`.

**Fix:**
```sql
CREATE INDEX idx_type_runtime ON shows_movies.titles(type, runtime);
```

**After index:** Rows filtered by `type` directly from index, avoiding double full scans.

### Findings:
- **Movies:** Average runtime demonstrates Netflix's focus on feature-length content
- **Shows:** Per-episode runtime reflects episodic format optimization for binge-watching
- **Content strategy:** Runtime differences confirm Netflix's differentiated approach between movies and serialized content
- **User experience:** Runtime patterns align with viewer consumption behaviors and platform engagement strategies

---

## 6. Top Actors in Highly Rated Titles

### SQL Query:
```sql
SELECT c.name AS actor, 
       COUNT(*) AS num_highly_rated_titles
FROM shows_movies.credits AS c
JOIN shows_movies.titles AS t 
ON c.id = t.id
WHERE c.role = 'actor'
  AND t.imdb_score > 8.0
  AND t.tmdb_score > 8.0
GROUP BY c.name
ORDER BY num_highly_rated_titles DESC;
```

### Optimization:
**Before index:** `EXPLAIN` shows join on `id` across `titles` and `credits` with no filtering support for scores.

**Fix:**
```sql
CREATE INDEX idx_scores ON shows_movies.titles(imdb_score, tmdb_score);
CREATE INDEX idx_role ON shows_movies.credits(role);
```

**After index:** Only actors + high-rated titles are scanned, join reduces to filtered subset.

### Findings:
- **Quality indicators:** Identifies actors who consistently feature in top-rated Netflix content
- **Talent patterns:** Reveals recurring collaborations between Netflix and high-performing actors
- **Strategic value:** Useful for talent acquisition and casting strategy decisions
- **Content quality:** Demonstrates correlation between specific actors and audience satisfaction ratings

---

## 7. Titles & Directors of Movies Released After 2010

### SQL Query:
```sql
SELECT DISTINCT t.title, c.name AS director, 
       release_year
FROM shows_movies.titles AS t
JOIN shows_movies.credits AS c 
ON t.id = c.id
WHERE t.type = 'Movie' 
  AND t.release_year >= 2010 
  AND c.role = 'director'
ORDER BY release_year DESC;
```

### Optimization:
**Before index:** Scans all `titles` and `credits` before filtering.

**Fix:**
```sql
CREATE INDEX idx_movie_release ON shows_movies.titles(type, release_year);
CREATE INDEX idx_director ON shows_movies.credits(role, id);
```

**After index:** Filters movies released >=2010 first, then joins only directors.

### Findings:
- **Modern focus:** Surfaces recent Netflix movie directors, sorted by recency
- **Creative partnerships:** Identifies Netflix's preferred directorial collaborations in the modern era
- **Content evolution:** Useful for spotting new creative relationships and Netflix's contemporary directorial strategy
- **Industry trends:** Reflects Netflix's shift toward working with established and emerging filmmakers post-2010

---

## Conclusion

Key insights from Netflix's library analysis:

- **Quality benchmarks:** Clear performance distinctions between high and low-rated content
- **Temporal focus:** Strong emphasis on modern content with dramatic 2000s+ expansion  
- **Age targeting:** TV-14 content performs best, while R and PG-13 dominate volume
- **Genre strategy:** Comedy, Documentary, and Drama lead across all categories

Netflix balances mainstream appeal with niche content, prioritizing recent titles that align with audience demand.

---

## Business Impact & Use Cases

### Content Strategy
- **Acquisition focus:** Prioritize Comedy, Documentary, and Drama content
- **Quality standards:** Use IMDB thresholds for content selection
- **Age optimization:** Emphasize TV-14 and PG-13 content for engagement

### Technical Applications  
- **Recommendations:** Leverage genre popularity and rating data for personalization
- **Content discovery:** Highlight high-scoring titles and popular genres
- **Production planning:** Use performance metrics to guide original content investment
