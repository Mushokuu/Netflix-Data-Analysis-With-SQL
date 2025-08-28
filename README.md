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
Top-rated titles consistently show exceptional IMDB scores (8.0+), indicating strong audience approval and critical acclaim from global viewers. Bottom-rated titles exhibit significantly lower scores, often reflecting weaker plots, production shortcomings, or limited mainstream audience appeal. This comparison provides valuable benchmarks for identifying high-performing content and areas requiring strategic improvement in content curation.

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
Netflix's content library shows clear evolution across decades, revealing strategic shifts in acquisition patterns over time. Limited representation exists from 1940s-1980s, with significant expansion beginning in the 1990s (**121 titles**). The **2010s** dominate with **3,304 titles**, while the **2020s** already contribute **1,972 titles** despite being incomplete. This demonstrates Netflix's strategic focus on prioritizing contemporary content that aligns with current viewing preferences while maintaining curated classics.

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
**TV-14** titles achieved the highest average IMDB score (**6.71**), demonstrating strong audience engagement with content aimed at teenage and adult demographics. **R-rated movies** dominate the catalog with **556 titles**, followed by **PG-13** (451) and **PG** (233), reflecting diverse audience preferences. The prevalence of mature content categories highlights Netflix's strategic focus on adult audiences while maintaining substantial family-friendly viewing options for broader demographic appeal.

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
**Movies:** Comedy leads with 384 titles, followed by Documentary (230) and Drama (224), showing diverse storytelling preferences across audiences.
**Shows:** Reality programming tops with 113 entries, then Drama (104) and Comedy (100), reflecting growing demand for unscripted content.
**Overall top three:** Comedy (484), Documentary (329), Drama (328) demonstrate consistent cross-format appeal.

Comedy's universal appeal across both movies and shows confirms its fundamental role in Netflix's global content strategy and audience retention.

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
