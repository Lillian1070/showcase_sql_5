# [SQL] Counting Word Occurrences in Drafts

Practice from [StrataScratch](https://platform.stratascratch.com/coding/9817-find-the-number-of-times-each-word-appears-in-drafts?code_type=3). 

- **Objective**: This query counts how many times each word appears in the `contents` field of the `google_file_store` table.
- **Practice Purpose**: Self-learning and reinforcement of SQL techniques such as data cleaning, aggregation, joins, subqueries, and window functions.
- **Outline**: 
    - [**Practice**](#section-1) (practice problem and query output)
    - [**Solution**](#section-2) (step-by-step explanation)
    - [**Future Enhancements**](#section-3) (if any)


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

- Extract the contents field from the `google_file_store` table.
- Clean and process the text data (remove punctuation, replace spaces, convert to lowercase).
- Split the `contents` into individual words.


### Step 2: Clean the Text and Extract Words

1. **Clean the Text** (Remove Punctuation): We use [`REGEXP_REPLACE()`](https://www.datacamp.com/doc/mysql/mysql-regexp-replace) to clean up the text data in the contents column by removing punctuation marks, as these can interfere with word counting.

2. **Format into a JSON Array**: Since SQL does not have a direct function to split a string into words, we can use `REPLACE()` to replace spaces with commas to format the text into a JSON array and use `CONCAT()` to wrap the result into a valid JSON array

3. **Split Using `JSON_TABLE`**: After creating the JSON array, we use the `JSON_TABLE()` function to extract each word into separate rows (aka. create a table-like structure from a JSON array of words).

```sql
JSON_TABLE(
    CONCAT(
        '["',
        REPLACE(REGEXP_REPLACE(g.contents, '[[:punct:]]', ''), ' ', '","'),
        '"]'
    ),
    '$[*]' COLUMNS (word VARCHAR(200) PATH '$')
)
```


### Step 3: Aggregate and Sort Word Frequencies

1. **Normalize Words into Lowercase**: To avoid case-sensitive mismatches (e.g., "Market" and "market" should be counted as the same word), we use the `LOWER()` function to convert all words to lowercase.

2. **Group and Count Occurrences**: We group the words by their lowercase versions (using `GROUP BY LOWER(t.word)`) and count how many times each word appears using `COUNT(*)`.

3. **Sort the Result**: Finally, we order the results by the frequency of occurrences in descending order, so the most common words appear first.

```sql
SELECT 
    LOWER(t.word) AS word,
    COUNT(*) AS occurrences
FROM [table]
GROUP BY LOWER(t.word)
ORDER BY occurrences DESC;
```


### Final Syntax and Output using MySQL

***Syntax**

```sql
SELECT 
    LOWER(t.word) AS word,
    COUNT(*) AS occurrences
FROM google_file_store g
JOIN JSON_TABLE(
    CONCAT(
        '["',
        REPLACE(REGEXP_REPLACE(g.contents, '[[:punct:]]', ''), ' ', '","'),
        '"]'
    ),
    '$[*]' COLUMNS (word VARCHAR(200) PATH '$')
) t
GROUP BY LOWER(t.word)
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


## <a name="section-3"></a>🚀 Future Enhancements

- **Handling Stop Words**: We can extend this query by excluding common "stop words" (like "a", "and", "the", etc.) to focus on more meaningful words.

- **Additional Data Processing**: Consider handling word stemming or lemmatization to treat similar words (e.g., "running" and "run") as the same.

- **Performance Optimization**: For larger datasets, optimize the query by using alternative methods for splitting text, such as preprocessing the text outside of SQL before querying.


_💬 I’d love to hear your thoughts! If you have any suggestions or questions, feel free to connect with me._
