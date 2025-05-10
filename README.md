# Netflix Movies and TV Shows Data Analysis using SQL

## Overview

This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objectives

- **Content Type Analysis:** Determine the proportion of movies versus TV shows available on the platform.
- **Rating Distribution:** Identify the most prevalent content ratings for both movies and TV shows.
- **Temporal and Geographic Analysis:** Examine content trends based on release years and the originating countries, as well as explore content durations.
- **Content Categorization and Keyword Analysis:** Investigate and classify content based on specific genres and the presence of particular keywords in descriptions.
- **Performance Analysis (e.g., Actor Appearances):** Analyze the frequency of actor appearances in specific regional content.

## Dataset

The data for this project is sourced from the Kaggle dataset:

- **Dataset Link:** [Movies Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

## Schema

```sql
DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix
(
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


## Business Problems and Solutions

### 1. Content Type Distribution

```sql
SELECT
    type,
    COUNT(*) AS Count
FROM netflix
GROUP BY type;
```

**Objective:** To understand the balance between movies and TV shows in the Netflix catalog.

### 2. Find the Most Common Rating for Movies and TV Shows

```sql
WITH RatingCounts AS (
    SELECT
        type,
        rating,
        COUNT(*) AS rating_count,
        ROW_NUMBER() OVER (PARTITION BY type ORDER BY COUNT(*) DESC) AS rn
    FROM netflix
    GROUP BY type, rating
)
SELECT
    type,
    rating AS most_frequent_rating
FROM RatingCounts
WHERE rn = 1;
```

**Objective:** Identify the most frequently occurring rating for each type of content.

### 3. List All Movies Released in a Specific Year (e.g., 2019)

```sql
SELECT *
FROM netflix
WHERE release_year = 2019 AND type = 'Movie';
```

**Objective:** To retrieve a list of all movies that premiered in a given year.

### 4. Find the Top 5 Countries with the Most Content on Netflix

```sql
SELECT TOP 5
    TRIM(value) AS country,
    COUNT(*) AS total_content
FROM netflix
CROSS APPLY STRING_SPLIT(country, ',')
WHERE country IS NOT NULL AND TRIM(value) <> ''
GROUP BY TRIM(value)
ORDER BY total_content DESC;
```

**Objective:** Identify the top 5 countries with the highest number of content items.

### 5. Longest Movie Duration

```sql
SELECT TOP 1
    *
FROM netflix
WHERE type = 'Movie'
ORDER BY CAST(LEFT(duration, CHARINDEX(' ', duration) - 1) AS INT) DESC;
```

**Objective:** Find the movie with the longest duration.

### 6. Find Content Added in the Last 5 Years

```sql
SELECT *
FROM netflix
WHERE TRY_CONVERT(DATETIME, date_added, 6) >= DATEADD(year, -5, GETDATE());
```

**Objective:** Retrieve content added to Netflix in the last 5 years.

### 7. Find All Movies/TV Shows by Director 'Rajiv Chilaka'

```sql
SELECT
    n.*
FROM netflix n
CROSS APPLY STRING_SPLIT(director, ',') d
WHERE TRIM(d.value) = 'Rajiv Chilaka';
```

**Objective:** List all content directed by 'Rajiv Chilaka'.

### 8. List All TV Shows with More Than 5 Seasons

```sql
SELECT *
FROM netflix
WHERE type = 'TV Show'
  AND CAST(LEFT(duration, CHARINDEX(' ', duration) - 1) AS INT) > 5;
```

**Objective:** Identify TV shows with more than 5 seasons.

### 9. Count the Number of Content Items in Each Genre

```sql
SELECT
    TRIM(value) AS genre,
    COUNT(*) AS total_content
FROM netflix
CROSS APPLY STRING_SPLIT(listed_in, ',')
GROUP BY TRIM(value);
```

**Objective:** Count the number of content items in each genre.

### 10. Find each year and the average numbers of content release in India on netflix. Return top 5 year with highest avg content release!

```sql
WITH IndiaContent AS (
    SELECT
        release_year,
        COUNT(show_id) AS total_release
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

**Objective:** Calculate and rank years by the average number of content releases by India.

### 11. List All Movies that are Documentaries

```sql
SELECT *
FROM netflix
WHERE listed_in LIKE '%Documentaries%';
```

**Objective:** Retrieve all movies classified as documentaries.

### 12. Find All Content Without a Director

```sql
SELECT *
FROM netflix
WHERE director IS NULL OR director = '';
```

**Objective:** List content that does not have a director.

### 13. Find How Many Movies Actor 'Salman Khan' Appeared in the Last 10 Years

```sql
SELECT COUNT(*) AS movie_count
FROM netflix
WHERE type = 'Movie'
  AND casts LIKE '%Salman Khan%'
  AND release_year > YEAR(GETDATE()) - 10;
```

**Objective:** Count the number of movies featuring 'Salman Khan' in the last 10 years.

### 14. Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in India

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

**Objective:** Identify the top 10 actors with the most appearances in Indian-produced movies.

### 15. Categorize Content Based on the Presence of 'Kill' and 'Violence' Keywords

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

**Objective:** Categorize content as 'Bad' if it contains 'kill' or 'violence' and 'Good' otherwise. Count the number of items in each category.


### Findings and Conclusion

**Content Diversity:** The Netflix platform offers a wide array of both movies and TV shows, catering to diverse viewer preferences.

**Rating Trends:** Understanding the distribution of content ratings can provide insights into the platform's target demographics and content suitability.

**Global Content Contribution:** The analysis of content by country highlights the significant contributions from various regions, showcasing Netflix's international content strategy.

**Genre Popularity:** Examining content counts by genre reveals popular categories and potential areas for future content investment.

**Thematic Content Analysis:** Categorizing content based on descriptive keywords provides a high-level understanding of the themes prevalent on the platform.


This SQL-based analysis offers a comprehensive overview of the Netflix content dataset, providing actionable insights that can inform content strategy, marketing efforts, and overall platform development.
