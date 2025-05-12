# Netflix Movies and TV Shows Data Analysis Using SQL

## Introduction

This project leverages SQL to analyze Netflix’s expansive library of movies and TV shows. Our goal is to extract actionable insights about content diversity, regional trends, genre popularity, and key contributions from actors and directors. With carefully constructed queries, we unearth patterns that can inform strategic decisions for streaming platforms.

## Objectives

- **Content Diversity:** Assess the distribution between movies and TV shows.
- **Ratings Analysis:** Identify the most frequent ratings to understand the audience target.
- **Time Trends:** Analyze release patterns and recent content additions.
- **Genre & Keyword Exploration:** Discover popular genres and flag content based on intense keywords.
- **Regional & People Insights:** Examine contributions by country as well as key directors and actors.

## Dataset

- **Source:** [Netflix Shows Dataset on Kaggle](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)
- **Table Name:** `netflix`

## Table Schema

```sql
CREATE TABLE netflix (
    show_id      VARCHAR(5),
    type         VARCHAR(10),
    title        VARCHAR(250),
    director     VARCHAR(550),
    casts        VARCHAR(1050),
    country      VARCHAR(550),
    date_added   VARCHAR(55),
    release_year INT,
    rating       VARCHAR(15),
    duration     VARCHAR(15),
    listed_in    VARCHAR(250),
    description  VARCHAR(550)
);
```

## 1. Content Overview

### 1.1 Movies vs TV Shows

Count the number of movies and TV shows:

```sql
SELECT type, COUNT(*) AS Count 
FROM netflix 
GROUP BY type;
```

### 1.2 Most Common Ratings

Identify the most frequent rating for each content type:

```sql
WITH RatingCounts AS (
    SELECT type, rating, COUNT(*) AS rating_count,
           ROW_NUMBER() OVER (PARTITION BY type ORDER BY COUNT(*) DESC) AS rn
    FROM netflix
    GROUP BY type, rating
)
SELECT type, rating AS most_frequent_rating
FROM RatingCounts
WHERE rn = 1;
```

### 1.3 Genre Distribution

Break down the content by genres by splitting the `listed_in` field:

```sql
SELECT TRIM(value) AS genre, COUNT(*) AS total_content
FROM netflix
CROSS APPLY STRING_SPLIT(listed_in, ',')
GROUP BY TRIM(value);
```

## 2. Time-Based Analysis

### 2.1 Movies Released in 2019

Retrieve all movies that were released in 2019:

```sql
SELECT * 
FROM netflix 
WHERE release_year = 2019 AND type = 'Movie';
```

### 2.2 Content Added in the Last 5 Years (as of May 2020)

Select titles added since May 1, 2015. We use `TRY_CONVERT` to correctly interpret the date:

```sql
SELECT * 
FROM netflix 
WHERE TRY_CONVERT(DATETIME, date_added, 6) >= '2015-05-01';
```

### 2.3 Top 5 Years by Average Indian Content Releases

Determine which years had the highest proportion of Indian content:

```sql
WITH IndiaContent AS (
    SELECT release_year, COUNT(show_id) AS total_release
    FROM netflix
    WHERE country LIKE '%India%'
    GROUP BY release_year
),
TotalIndiaContent AS (
    SELECT COUNT(show_id) AS total FROM netflix WHERE country LIKE '%India%'
)
SELECT TOP 5
    ic.release_year,
    ic.total_release,
    CAST(ic.total_release AS DECIMAL(10, 2)) / tic.total * 100 AS avg_release_percentage
FROM IndiaContent ic
CROSS JOIN TotalIndiaContent tic
ORDER BY avg_release_percentage DESC;
```

## 3. Country-Based Analysis

### 3.1 Top 5 Countries by Content Volume

Extract and count individual countries from the `country` field:

```sql
SELECT TOP 5 TRIM(value) AS country, COUNT(*) AS total_content
FROM netflix
CROSS APPLY STRING_SPLIT(country, ',')
WHERE country IS NOT NULL AND TRIM(value) <> ''
GROUP BY TRIM(value)
ORDER BY total_content DESC;
```

## 4. Content Attributes

### 4.1 Longest Movie

Identify the movie with the longest duration:

```sql
SELECT TOP 1 *
FROM netflix
WHERE type = 'Movie'
ORDER BY CAST(LEFT(duration, CHARINDEX(' ', duration) - 1) AS INT) DESC;
```

### 4.2 TV Shows with More Than 5 Seasons

Select TV shows that have more than 5 seasons:

```sql
SELECT * 
FROM netflix 
WHERE type = 'TV Show'
  AND CAST(LEFT(duration, CHARINDEX(' ', duration) - 1) AS INT) > 5;
```

### 4.3 Documentaries

Retrieve all content classified as Documentaries:

```sql
SELECT * 
FROM netflix 
WHERE listed_in LIKE '%Documentaries%';
```

### 4.4 Content Without a Director

Identify entries without any director information:

```sql
SELECT * 
FROM netflix 
WHERE director IS NULL OR director = '';
```

### 4.5 Keyword-Based Content Categorization (Kill or Violence)

Categorize content based on whether its description contains keywords like "kill" or "violence":

```sql
SELECT
    CASE 
        WHEN description LIKE '%kill%' OR description LIKE '%violence%' THEN 'Bad'
        ELSE 'Good'
    END AS category,
    COUNT(*) AS content_count
FROM netflix
GROUP BY
    CASE 
        WHEN description LIKE '%kill%' OR description LIKE '%violence%' THEN 'Bad'
        ELSE 'Good'
    END;
```

## 5. People-Based Analysis

### 5.1 Content Directed by Rajiv Chilaka

Retrieve content directed by Rajiv Chilaka:

```sql
SELECT n.*
FROM netflix n
CROSS APPLY STRING_SPLIT(director, ',') d
WHERE TRIM(d.value) = 'Rajiv Chilaka';
```

### 5.2 Salman Khan Movie Appearances in Last 10 Years

Count the movies featuring Salman Khan released in the last 10 years:

```sql
SELECT COUNT(*) AS movie_count
FROM netflix
WHERE type = 'Movie'
  AND casts LIKE '%Salman Khan%'
  AND release_year >= YEAR(GETDATE()) - 10;
```

### 5.3 Top 10 Actors in Indian Movies

Identify the top 10 actors by appearance count in Indian movies:

```sql
SELECT TOP 10
    TRIM(value) AS actor,
    COUNT(*) AS appearance_count
FROM netflix
CROSS APPLY STRING_SPLIT(casts, ',')
WHERE country LIKE '%India%' AND type = 'Movie'
GROUP BY TRIM(value)
ORDER BY appearance_count DESC;
```

## Conclusion

This analysis demonstrates how diverse SQL techniques—from common table expressions to string splitting—can transform raw data into strategic business insights. By exploring Netflix’s content distribution, release trends, genre popularity, and regional dynamics, we’re able to offer a deeper understanding of the streaming platform’s library. These insights could play a pivotal role in shaping content acquisition strategies, viewer engagement efforts, and market-specific innovations.
