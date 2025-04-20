# [SQL] Counting Word Occurrences in Drafts

_This SQL practice is based on a problem from [StrataScratch](https://platform.stratascratch.com/coding/9817-find-the-number-of-times-each-word-appears-in-drafts?code_type=3), used here for personal learning and educational purposes._

- **Objective**: This query counts how many times each word appears in the `contents` field of the `google_file_store` table.
- **Practice Purpose**: Self-learning and reinforcement of SQL techniques such as data cleaning, aggregation, joins, subqueries, and window functions.
- **Outline**: 
    - [**Practice**](#section-1) (practice problem and query output)
    - [**Solution**](#section-2) (step-by-step explanation)
    - [**Query Optimization**](#section-3) (refinement for efficiency and readability)
    - [**Future Enhancements**](#section-4) 


## <a name="section-1"></a>🧪 Practice 

Find the number of times each word appears in the `contents` column across all rows in the `drafts` dataset. Output two columns: `word` and `occurrences`.

- **Table**: `google_file_store`

|   filename   |   contents   |
|--------------|--------------|
|  draft1.txt  |	The stock exchange predicts a bull market which would make many investors happy.|
|  draft2.txt  |	The stock exchange predicts a bull market which would make many investors happy, but analysts warn of possibility of too much optimism and that in fact we are awaiting a bear market.|
|  final.txt   |	The stock exchange predicts a bull market which would make many investors happy, but analysts warn of possibility of too much optimism and that in fact we are awaiting a bear market. As always predicting the future market is an uncertain game and all investors should follow their instincts and best practices.|


### Expected Output

|  word   | occurrences |
|---------|-------------|
|  market |	     6      |
|    a    |      5      |
|investors|	     4      |
|  and    |    	 4      |
|  the    |	     4      |


## <a name="section-2"></a>🧠 Solution

_This section outlines my thought process for solving the problem._


### Step 1: Identify Required Data

- Clean contents: Extract the `contents` field and clean the text data (remove punctuation, convert to lowercase).
- A list of individual words: Split the cleaned `contents` into separate words.


### Step 2a: Clean the Text into `content_tab`

2a-1. **Clean the Text** (Remove Punctuation): Use [`REGEXP_REPLACE()`](https://www.datacamp.com/doc/mysql/mysql-regexp-replace) to clean up the text data in the contents column by removing punctuation marks, as these can interfere with word counting.

- `'[^\w\s]'` is the [regex](https://learn.microsoft.com/en-us/sql/relational-databases/regular-expressions/overview?view=azuresqldb-current) pattern:
    - `^` (inside the square brackets) means "not".
    - `\w` matches any word character → letters (a-z, A-Z), digits (0-9), and underscore (_).
    - `\s` matches whitespace → spaces, tabs, newlines.
    - `[^\w\s]` matches anything that is **NOT** a letter, digit, underscore, or space — in other words, punctuation like . , ; : ! ? ( ) " ' - etc.


2a-2. **Normalize Words into Lowercase**: To avoid case-sensitive mismatches (e.g., "Market" and "market" should be counted as the same word), use the `LOWER()` function to convert all words to lowercase.

```sql
# content_tab
SELECT 
    LOWER(REGEXP_REPLACE(contents, '[^\w\s]', '', 'g')) AS clean_content
FROM google_file_store
```

### Step 2b: Extract Words into `word_tab`

2b-1. **Split the Text and Format into Arrays**: Use [`STRING_TO_ARRAY()`](https://www.stratascratch.com/blog/string-and-array-functions-in-sql-for-data-science/) to split the `clean_content` string into an array of words using a space `' '` as the delimiter.

2b-2. **Expand Arrays into Rows**: Use [`UNNEST()`](https://count.co/sql-resources/bigquery-standard-sql/unnest) to expand arrays into individual rows.

```sql
# word_tab
SELECT 
    UNNEST(STRING_TO_ARRAY(clean_content, ' ')) AS word
FROM content_tab
```


### Step 3: Aggregate and Sort Word Frequencies

3a. **Group and Count Occurrences**: Group the words and count how many times each word appears using `COUNT()`.

3b. **Sort the Result**: Order the results by frequency in descending order so that the most common words appear first.

```sql
SELECT 
    DISTINCT word,
    COUNT(word) AS occurrences
FROM word_tab
GROUP BY word
ORDER BY occurrences DESC;
```


### Final Syntax and Output using MySQL

***Syntax**

```sql
WITH 
    content_tab AS (
        SELECT 
            LOWER(REGEXP_REPLACE(contents, '[^\w\s]', '', 'g')) AS clean_content
        FROM google_file_store
    ),
    word_tab AS (
        SELECT 
            UNNEST(STRING_TO_ARRAY(clean_content, ' ')) AS word
        FROM content_tab
    )
    
SELECT 
    DISTINCT word,
    COUNT(word) AS occurrences
FROM word_tab
GROUP BY word
ORDER BY occurrences DESC;
```

***Output**

_Showing top 10 rows here._

|  word   | occurrences |
|---------|-------------|
|  market |	     6      |
|    a    |      5      |
|investors|	     4      |
|   and   |    	 4      |
|   the   |	     4      |
|   of	  |	     4      |
|   bull  |	     3      |
|  which  |	     3      |
|  would  |	     3      |
|  make   |	     3      |


## <a name="section-3"></a>🛠️ Query Optimization using MySQL

_Note: This section is updated on 04/20/2025._

Upon review, I realized that the temp tables can be merged into inline subqueries, excluding the need for a `WITH` clause. This makes the query more concise and avoids the creation of temporary tables.

```sql
SELECT 
    word,
    COUNT(*) AS occurrences
FROM (
    SELECT 
        UNNEST(STRING_TO_ARRAY(LOWER(REGEXP_REPLACE(contents, '[^\w\s]', '', 'g')), ' ')) AS word
    FROM google_file_store
) 
GROUP BY word
ORDER BY occurrences DESC;
```


## <a name="section-4"></a>🚀 Future Enhancements

- **Handling Stop Words**: We can extend this query by excluding common "stop words" (like "a", "and", "the", etc.) to focus on more meaningful words.

- **Additional Data Processing**: Consider handling word stemming or lemmatization to treat similar words (e.g., "running" and "run") as the same.

- **Performance Optimization**: For larger datasets, optimize the query by using alternative methods for splitting text, such as preprocessing the text outside of SQL before querying.


_💬 I’d love to hear your thoughts! If you have any suggestions or questions, feel free to connect with me._
