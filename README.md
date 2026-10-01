# 🎬 Netflix Movies and TV Shows Analysis – SQL Project

## 📌 Project Overview

This project is a **Netflix Movies and TV Shows Analysis** project developed using SQL.

The goal of this project is to analyze Netflix content and answer practical business and content-related questions using SQL. The analysis covers movies vs TV shows, ratings, release years, countries, genres, directors, actors, seasons, recent additions, and content classification.

The project uses a single main table:

- `netflix_shows`

The database and table are created in SQL, followed by data import, data exploration, and analytical queries.

The database is created as `netflix`.

> **Note:** The uploaded SQL file uses PostgreSQL-style functions such as `UNNEST()`, `STRING_TO_ARRAY()`, `SPLIT_PART()`, `TO_DATE()`, `ILIKE`, and PostgreSQL casting syntax (`::`). The README documents the project as written in the source SQL without silently changing those queries.

---

## 🎯 Project Objectives

The main objectives of this project are to:

- Analyze the distribution of Movies and TV Shows
- Identify the most common ratings for Movies and TV Shows
- Find movies released in a specific year
- Identify countries producing the most Netflix content
- Find the longest movie
- Identify content added in the last five years
- Find movies and TV shows by a specific director
- Identify TV shows with more than five seasons
- Analyze content by genre
- Analyze India's content release distribution by year
- Identify documentary movies
- Find content without a director
- Analyze appearances of a specific actor
- Identify the top actors in Indian-produced content
- Categorize content based on keywords in descriptions

---

## 🛠️ Technologies Used

| Technology | Purpose |
|---|---|
| **SQL** | Data exploration, transformation and analysis |
| **PostgreSQL-style SQL syntax** | Analytical queries and string/date operations |
| **SQL Database** | Database and table management |
| **CSV Dataset** | Source data imported into `netflix_shows` |

---

## 🗄️ Database Schema

The project contains one main table: `netflix_shows`.

### `netflix_shows`

| Column | Description |
|---|---|
| `show_id` | Unique content ID and primary key |
| `type` | Content type: Movie or TV Show |
| `title` | Movie or TV show title |
| `director` | Director name(s) |
| `cast` | Cast/actor information |
| `country` | Country or countries associated with the content |
| `date_added` | Date the content was added to Netflix |
| `release_year` | Original release year |
| `rating` | Content rating |
| `duration` | Movie duration or number of TV show seasons |
| `listed_in` | Genre/category information |
| `description` | Content description |

---

## 🔗 Table Structure

```text
netflix
   │
   └── netflix_shows
          ├── show_id
          ├── type
          ├── title
          ├── director
          ├── cast
          ├── country
          ├── date_added
          ├── release_year
          ├── rating
          ├── duration
          ├── listed_in
          └── description
```

The `show_id` column is defined as the primary key.

---

## 📥 Database and Table Creation

```sql
CREATE DATABASE netflix;

USE netflix;

DROP TABLE IF EXISTS netflix_shows;

CREATE TABLE netflix_shows(
    show_id VARCHAR(20) PRIMARY KEY,
    type VARCHAR(20),
    title VARCHAR(105),
    director VARCHAR(205),
    cast VARCHAR(800),
    country VARCHAR(150),
    date_added VARCHAR(30),
    release_year INT,
    rating VARCHAR(15),
    duration VARCHAR(20),
    listed_in VARCHAR(100),
    description VARCHAR(300)
);
```

---

# 📊 Business Analysis & SQL Solutions

## 1. Count Movies vs TV Shows

**Business Question:**  
How many Movies and TV Shows are available in the dataset?

**SQL concepts used:**

- `SELECT`
- `COUNT()`
- `GROUP BY`

**Business Use:**  
Helps understand the overall distribution of Netflix content types.

```sql
SELECT 
    type,
    COUNT(*)
FROM netflix
GROUP BY 1;
```

---

## 2. Most Common Rating for Movies and TV Shows

**Business Question:**  
What is the most frequently occurring rating for each content type?

**SQL concepts used:**

- CTE
- `COUNT()`
- `RANK()`
- `PARTITION BY`
- `GROUP BY`

**Business Use:**  
Helps identify the dominant rating category separately for Movies and TV Shows.

```sql
WITH RatingCounts AS (
    SELECT 
        type,
        rating,
        COUNT(*) AS rating_count
    FROM netflix
    GROUP BY type, rating
),
RankedRatings AS (
    SELECT 
        type,
        rating,
        rating_count,
        RANK() OVER (
            PARTITION BY type 
            ORDER BY rating_count DESC
        ) AS rank
    FROM RatingCounts
)
SELECT 
    type,
    rating AS most_frequent_rating
FROM RankedRatings
WHERE rank = 1;
```

---

## 3. Movies Released in a Specific Year

**Business Question:**  
List all movies released in 2020.

**SQL concepts used:**

- `SELECT`
- `WHERE`
- Filtering

**Business Use:**  
Allows analysis of content released during a particular year.

```sql
SELECT * 
FROM netflix
WHERE release_year = 2020;
```

---

## 4. Top 5 Countries with the Most Content

**Business Question:**  
Which five countries have the highest amount of Netflix content?

**SQL concepts used:**

- `UNNEST()`
- `STRING_TO_ARRAY()`
- `COUNT()`
- `GROUP BY`
- `ORDER BY`
- `LIMIT`
- Subquery

**Business Use:**  
Helps identify the countries contributing the largest amount of content to the dataset.

```sql
SELECT * 
FROM
(
    SELECT 
        UNNEST(STRING_TO_ARRAY(country, ',')) AS country,
        COUNT(*) AS total_content
    FROM netflix
    GROUP BY 1
) AS t1
WHERE country IS NOT NULL
ORDER BY total_content DESC
LIMIT 5;
```

---

## 5. Identify the Longest Movie

**Business Question:**  
Which movie has the longest duration?

**SQL concepts used:**

- `WHERE`
- `SPLIT_PART()`
- Type casting
- `ORDER BY`

**Business Use:**  
Helps identify the longest movie available in the dataset.

```sql
SELECT *
FROM netflix
WHERE type = 'Movie'
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC;
```

---

## 6. Content Added in the Last 5 Years

**Business Question:**  
Which content was added to Netflix during the last five years?

**SQL concepts used:**

- `TO_DATE()`
- Date filtering
- `CURRENT_DATE`
- `INTERVAL`

**Business Use:**  
Helps focus analysis on recently added Netflix content.

```sql
SELECT *
FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY')
      >= CURRENT_DATE - INTERVAL '5 years';
```

---

## 7. Movies and TV Shows by Director

**Business Question:**  
Find all movies and TV shows associated with director **Rajiv Chilaka**.

**SQL concepts used:**

- Subquery
- `UNNEST()`
- `STRING_TO_ARRAY()`
- Filtering

**Business Use:**  
Helps analyze the content associated with a particular director.

```sql
SELECT *
FROM
(
    SELECT 
        *,
        UNNEST(STRING_TO_ARRAY(director, ',')) AS director_name
    FROM netflix
)
WHERE director_name = 'Rajiv Chilaka';
```

---

## 8. TV Shows with More Than 5 Seasons

**Business Question:**  
Which TV shows have more than five seasons?

**SQL concepts used:**

- `WHERE`
- `SPLIT_PART()`
- Type casting
- Conditional filtering

**Business Use:**  
Helps identify long-running TV shows.

```sql
SELECT *
FROM netflix
WHERE type = 'TV Show'
  AND SPLIT_PART(duration, ' ', 1)::INT > 5;
```

---

## 9. Content Count by Genre

**Business Question:**  
How many content items belong to each genre?

**SQL concepts used:**

- `UNNEST()`
- `STRING_TO_ARRAY()`
- `COUNT()`
- `GROUP BY`

**Business Use:**  
Helps understand the genre distribution of the Netflix catalog.

```sql
SELECT 
    UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS genre,
    COUNT(*) AS total_content
FROM netflix
GROUP BY 1;
```

---

## 10. India's Content Release Distribution

**Business Question:**  
For India, identify the years with the highest percentage of content releases.

**SQL concepts used:**

- `COUNT()`
- Subquery
- `ROUND()`
- Type casting
- `GROUP BY`
- `ORDER BY`
- `LIMIT`

**Business Use:**  
Helps analyze the distribution of Indian Netflix content across release years.

```sql
SELECT 
    country,
    release_year,
    COUNT(show_id) AS total_release,
    ROUND(
        COUNT(show_id)::numeric /
        (SELECT COUNT(show_id)
         FROM netflix
         WHERE country = 'India')::numeric * 100,
        2
    ) AS avg_release
FROM netflix
WHERE country = 'India'
GROUP BY country, 2
ORDER BY avg_release DESC
LIMIT 5;
```

---

## 11. Movies That Are Documentaries

**Business Question:**  
List all movies categorized as documentaries.

**SQL concepts used:**

- `SELECT`
- `WHERE`
- `LIKE`

**Business Use:**  
Helps identify documentary content in the catalog.

```sql
SELECT *
FROM netflix
WHERE listed_in LIKE '%Documentaries';
```

---

## 12. Content Without a Director

**Business Question:**  
Which content items do not have a director listed?

**SQL concepts used:**

- `IS NULL`
- Filtering

**Business Use:**  
Helps identify missing director information in the dataset.

```sql
SELECT *
FROM netflix
WHERE director IS NULL;
```

---

## 13. Salman Khan Movies in the Last 10 Years

**Business Question:**  
Find content featuring actor **Salman Khan** released within the last ten years.

**SQL concepts used:**

- `LIKE`
- `EXTRACT()`
- Date/year filtering

**Business Use:**  
Helps analyze the recent presence of a particular actor in the dataset.

```sql
SELECT *
FROM netflix
WHERE casts LIKE '%Salman Khan%'
  AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;
```

---

## 14. Top 10 Actors in Indian Content

**Business Question:**  
Find the top 10 actors who appear in the highest number of content items produced in India.

**SQL concepts used:**

- `UNNEST()`
- `STRING_TO_ARRAY()`
- `COUNT()`
- `GROUP BY`
- `ORDER BY`
- `LIMIT`

**Business Use:**  
Helps identify frequently appearing actors within Indian-produced Netflix content.

```sql
SELECT 
    UNNEST(STRING_TO_ARRAY(casts, ',')) AS actor,
    COUNT(*)
FROM netflix
WHERE country = 'India'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;
```

---

## 15. Content Categorization Using Description Keywords

**Business Question:**  
Categorize content based on whether the description contains the keywords `kill` or `violence`.

The source SQL labels:

- Content containing `kill` or `violence` as **Bad**
- Other content as **Good**

**SQL concepts used:**

- Subquery
- `CASE`
- `ILIKE`
- `COUNT()`
- `GROUP BY`

**Business Use:**  
Demonstrates how text-based business rules can be implemented using SQL.

```sql
SELECT 
    category,
    type,
    COUNT(*) AS content_count
FROM (
    SELECT 
        *,
        CASE 
            WHEN description ILIKE '%kill%'
              OR description ILIKE '%violence%'
            THEN 'Bad'
            ELSE 'Good'
        END AS category
    FROM netflix
) AS categorized_content
GROUP BY 1, 2
ORDER BY 2;
```

> **Note:** The `Good`/`Bad` labels above are the classification rule defined in the project SQL. They should be understood as a keyword-based classification, not as a quality judgment about the content.

---

# 🧠 SQL Concepts Demonstrated

This project demonstrates practical SQL skills including:

```text
CREATE DATABASE
CREATE TABLE
DROP TABLE
PRIMARY KEY
SELECT
WHERE
IS NULL
LIKE
ILIKE
COUNT()
ROUND()
GROUP BY
ORDER BY
LIMIT
CASE
CTE
Subqueries
RANK()
PARTITION BY
UNNEST()
STRING_TO_ARRAY()
SPLIT_PART()
TO_DATE()
CURRENT_DATE
INTERVAL
EXTRACT()
Type Casting
```

---

# 📈 Key Analytical Areas

### 🎬 Content Type Analysis
- Movies vs TV Shows
- TV shows with multiple seasons
- Longest movies

### ⭐ Rating Analysis
- Most common rating by content type

### 🌍 Country Analysis
- Top content-producing countries
- Indian content analysis

### 🎭 Genre Analysis
- Content count by genre
- Documentary identification

### 👤 People Analysis
- Director-based analysis
- Actor-based analysis
- Salman Khan content analysis
- Top actors in Indian content

### 📅 Time-Based Analysis
- Content released by year
- Content added in the last five years
- Recent actor appearances

### 📝 Text Analysis
- Description keyword classification
- Missing director information

---

# 📚 Complete SQL Query Reference

The project contains **15 analytical questions** covering content, ratings, countries, movies, TV shows, genres, directors, actors, release years, and text classification.

| # | Analysis | Main SQL Concepts |
|---|---|---|
| 1 | Movies vs TV Shows | `COUNT`, `GROUP BY` |
| 2 | Most Common Rating | CTE, `RANK`, `PARTITION BY` |
| 3 | Movies by Year | `WHERE` |
| 4 | Top 5 Countries | `UNNEST`, `STRING_TO_ARRAY`, `COUNT` |
| 5 | Longest Movie | `SPLIT_PART`, sorting |
| 6 | Recent Content | `TO_DATE`, `INTERVAL` |
| 7 | Content by Director | `UNNEST`, subquery |
| 8 | TV Shows > 5 Seasons | `SPLIT_PART`, filtering |
| 9 | Genre Analysis | `UNNEST`, `COUNT` |
| 10 | India Release Analysis | `COUNT`, subquery, `ROUND` |
| 11 | Documentaries | `LIKE` |
| 12 | Missing Directors | `IS NULL` |
| 13 | Salman Khan Analysis | `LIKE`, `EXTRACT` |
| 14 | Top Indian Actors | `UNNEST`, `COUNT`, `LIMIT` |
| 15 | Keyword Classification | `CASE`, `ILIKE`, subquery |

---

# 💼 Skills Demonstrated

This project demonstrates my ability to:

- Design a relational database table
- Create and manage SQL databases
- Import and explore datasets
- Write analytical SQL queries
- Filter and aggregate data
- Use `GROUP BY` and `ORDER BY`
- Use aggregate functions
- Work with dates and time periods
- Work with strings and comma-separated fields
- Use CTEs and subqueries
- Apply window functions
- Perform ranking analysis
- Analyze content by country and genre
- Analyze actors and directors
- Identify missing data
- Create rule-based classifications
- Solve practical data-analysis problems using SQL

---

# 🔍 SQL Query Techniques Used

The project demonstrates practical use of:

- `SELECT` for data retrieval
- `WHERE` for filtering
- `GROUP BY` for aggregation
- `ORDER BY` for sorting
- `LIMIT` for top-N analysis
- `COUNT()` for record counting
- `ROUND()` for numerical formatting
- `CASE` for conditional logic
- CTEs using `WITH`
- Subqueries for nested analysis
- `RANK()` for ranking
- `PARTITION BY` for grouped ranking
- `UNNEST()` for expanding array-like data
- `STRING_TO_ARRAY()` for splitting comma-separated values
- `SPLIT_PART()` for extracting values from text
- `TO_DATE()` for date conversion
- `EXTRACT()` for extracting year information
- `ILIKE` for case-insensitive text matching
- `IS NULL` for missing-value analysis

---

# 🚀 Project Outcome

This project converts raw Netflix movies and TV shows data into meaningful analytical insights using SQL.

It demonstrates practical experience in:

**SQL querying, data exploration, aggregation, filtering, text analysis, date analysis, ranking, CTEs, subqueries, and business-oriented problem solving.**

The project also provides a strong SQL portfolio example for demonstrating data-analysis skills in interviews and on GitHub.

---

# 📁 Project Files

```text
Netflix-Movies-and-TV-Shows-Analysis/
│
├── README.md
├── netflix_shows.sql
└── netflix_titles.csv
```

> The SQL file used for this README is `netflix_shows.sql`. The CSV filename above represents the expected dataset file; add the actual dataset filename you used to your repository if it differs.

---

# 👨‍💻 Author

**Solomon Isaac**

Aspiring Data Analyst | SQL | Excel | Power BI | Python
